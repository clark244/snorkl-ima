# Generating a Client Report (AI-assisted)

`index.html` is the **canonical template** for an Impact Measurement Assessment. There is
no build step. The markup, JS, and SVG live in `index.html`; all CSS lives alongside it in
`styles.css` (linked from the `<head>`). Keep these two files together — `index.html`
references `styles.css` by relative path, so they must travel as a pair. The fonts load from
Typekit/Google CDNs; everything else is local.

The currently-filled version (Snorkl) doubles as a **gold-standard example**: every
content region is filled with real, well-formed copy. To make a new client report, you
clone the file and **rewrite the content regions** while leaving the **chrome** untouched.

---

## How to generate a new report

1. Copy `index.html` → `dist/<client>.html` **and** copy `styles.css` alongside it (keep the
   originals as the template). The copied HTML still links `styles.css` by relative path.
2. Feed an LLM: (a) the source material for the new client — discovery notes,
   questionnaire, public info; (b) this guide; (c) the filled Snorkl file as the example.
3. Instruct it to rewrite **only** the content regions listed below, preserving the chrome
   and all the coupling rules.
4. Run the **post-generation checklist** at the bottom before sending.

### Exporting a portable single file (for email / PDF)

The working copy links `styles.css`, so the two files must stay together. To produce **one
self-contained `.html`** (CSS inlined, no local dependencies) for emailing or PDF export:

```
python3 bundle.py                      # -> dist/snorkl-bundled.html
python3 bundle.py dist/<client>.html   # custom output path
```

`bundle.py` needs only system Python 3 — no install, no `package.json`. It reads
`index.html` + `styles.css`, inlines the CSS back into a `<style>` block, and writes the
bundle to `dist/` (gitignored). It never modifies the source files. Open the bundled file
and print-to-PDF from there. (CDN fonts still load over the network as before.)

The model is *clone + rewrite-in-place*, not *regenerate from scratch*. The chrome (layout,
CSS, JS logic, SVG geometry) is hard-won and identical for every client — never rebuild it.

---

## Content regions to rewrite (per client)

These carry client-specific content. Locations are approximate — search by the anchor text.

| # | Region | Where | Notes |
|---|--------|-------|-------|
| 1 | Cover + sidebar meta | `cover-client`, `cover-meta-value`, `sidebar-meta` | Client name, "prepared for" contact, date. **"Cobalt Collective" is the report author — constant, do not change.** |
| 2 | Key Findings (4 cards) | `class="kf-card"` | The executive summary. |
| 3 | Recommendation at a Glance | `id="recommendation"` | Intro paragraph **+ the strategy matrix** (`matrix-table`). The matrix dots (`●`/`○`/`–`) and horizons encode judgments — set them deliberately per client. |
| 4 | §01 About This Report | `id="about"` | Mostly boilerplate; swap client name, source dates, and the discovery-conversation reference. |
| 5 | §02 Process Model — prose | `id="model"` intro `<p>` | The narrative framing of the diagram. |
| 6 | §02 Process Model — **the diagram** | the two `<svg>` blocks + JS data | **The hardest region. See "The diagram trap" below.** |
| 7 | §03 Evidence Landscape | `id="evidence-table"` + closing `<p>` | Table rows = the client's existing evidence assets. |
| 8 | §04 Measurement Opportunities | `id="opportunities"` intro `<p>`s | Includes the buyer-reality paragraph. |
| 9 | §04 The Four Priorities (cards) | `class="priority-card"` | The detailed payload. |
| 10 | §05 Recommended Next Steps | `next-steps-list` | |
| 11 | Sidebar nav labels | `<nav>` in `id="sidebar"` | Must **mirror** the section titles and the four priority titles. |

---

## The diagram trap (region 6) — read before touching the model figure

The interactive process model carries the same content in **multiple coupled places that
must stay in sync**. Change one without the others and the figure breaks silently.

The model has **7 nodes**: `product, impl, behavior, outcome, mech1, mech2, user`.
For each node, content lives in:

1. **SVG `<text>` labels** in `#view-model` — the visible box text. **Positioned by
   hand-tuned `x`/`y` coordinates.** Rewrite the *words*, keep them short enough to fit the
   box, and **do not change coordinates** unless you re-check the layout visually.
2. **Clickable `<rect ... onclick="show('<id>')">`** — the hit region. The `id` string must
   match a key in the `nodes` object. Don't rename ids.
3. **The `nodes` JS object** (`const nodes = {...}`, ~line 1384) — the detail-panel content
   (`cat`, `title`, `tip`, `badge`, `badgeText`, `know`, `gaps`) shown when a box is clicked.
   This is real per-client prose. The `badge` field (`ok`/`par`/`gap`) drives the colored
   confidence pill — set it to reflect how well-understood that part of the causal chain is.
4. **Measurement view** (`#view-meas`) — a second `<svg>` plus the `priorityData` JS object
   (`const priorityData = {...}`, ~line 1608) that fills the priority popups. Its four
   entries mirror the four priorities; keep them consistent with the priority cards (region 9).

**Rule:** if you change a node's meaning, update all of 1–4 for that node. If you change the
*number or identity* of nodes, you are redrawing the figure — that's a layout job, not a
content swap; do it deliberately and eyeball the result.

---

## Do NOT touch (chrome — identical for every client)

- The entire `styles.css` file (all CSS, including `--measure` and the layout variables).
- All JS **functions/logic**: `show()`, `switchView()`, `toggleCard()`, the scroll-spy
  `IntersectionObserver` + `navMap`, the detail-panel editing logic, `openPriorityCard()`.
  (You edit the JS **data** — `nodes`, `priorityData` — never the functions.)
- SVG **geometry**: `<rect>`/`<line>`/`<marker>`/`<defs>` coordinates, arrowheads, filters,
  the dim-overlays and connector lines. Only the `<text>` *words* are content.
- The footer brand and "Cobalt Collective" / `cobaltcollective.org` author identity.
- `navMap` keys — if you add or remove a top-level section, update `navMap` (~line 1352) to
  match, or scroll-spy highlighting breaks. (Adding sections is rare; most reports reuse the
  five-section skeleton.)

---

## Other coupling to keep in sync

- **Nav ↔ sections:** every `navMap` key has a section `id`; every nav `<a href="#x">` points
  at one. The four priority sub-links (`#card-p1`…`#card-p4`) must match the four card ids.
- **Matrix ↔ priorities:** the four matrix rows (region 3) and the four priority cards
  (region 9) are the same four priorities — titles, order, and horizons must agree.
- **Recommendation summary ↔ §04:** the top summary intentionally restates §04's opening.
  Keep them consistent but not verbatim.

---

## Post-generation checklist

- [ ] No "Snorkl", "Jon Laven", "Houston ISD", "Des Moines", "Twin Rivers", or other
      previous-client specifics left anywhere (grep the file).
- [ ] "Cobalt Collective" author identity intact (cover, footer, sidebar brand).
- [ ] Open in a browser: every model-diagram box click opens a panel with matching content.
- [ ] Toggle to Measurement View: hovering each priority badge highlights connectors; popups
      open with the right priority content.
- [ ] All four priority cards expand/collapse; nav sub-links scroll to the right card.
- [ ] Scroll top→bottom: the sidebar active state tracks the visible section.
- [ ] Matrix rows match the four priority cards (titles + horizons).
- [ ] Print/PDF preview looks right (the print-only model-description section renders).
- [ ] Prose still respects the `--measure` width — no paragraph runs the full page width.
