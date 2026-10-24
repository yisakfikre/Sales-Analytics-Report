# Implementation SOP — Sales Analytics Report

End-to-end procedure, from the current PBIX to a published GitHub repository.
Written to be executed without Claude. Where a UI path could not be verified it is
marked **[VERIFY]** with a safe way to find the right control.

**Status of this document:** instructions only. Nothing in it has been applied to your
PBIX or to GitHub.

---

## Legend

| Tag | Meaning |
|---|---|
| **[VERIFIED]** | Read directly from your PBIX, your Excel workbook, or Microsoft documentation |
| **[CALCULATED]** | Computed from your source data; method stated |
| **[ASSUMED]** | A reasonable interpretation, not confirmed |
| **[UNTESTED]** | Behaviour that must be tested in Power BI before relying on it |
| **[VERIFY]** | A UI path that may differ in your build |

---

## Phase 1 — Inventory, prerequisites, backup

### 1.1 Confirm you have these inputs

| Input | Needed for |
|---|---|
| `Sales Analysis Report.pbix` | Everything |
| `Sales_Dataset.xlsx` | Reconciliation, data dictionary |
| `Cream_Forest_Theme_v2.json` | Theme application |
| Six page screenshots | README, documentation |
| Existing GitHub repository | Publication |

### 1.2 Back up

1. Close Power BI Desktop.
2. Copy the PBIX file (`Ctrl+C`, `Ctrl+V` in File Explorer).
3. Rename the copy `Sales_Analysis_Report_PRE-REDESIGN_<YYYY-MM-DD>.pbix`.
4. Move it to a `_backups` folder.

Take the copy with the application **closed**. A copy taken while Desktop holds the
file can capture an inconsistent state.

### 1.3 Prerequisites inside Power BI

- **View** ribbon → enable **Selection** pane and **Bookmarks** pane.
- **View → Page view → Actual size.** Coordinates only match what you see at actual size.

### Checkpoint 1
- [ ] Dated backup exists in `_backups` and opens
- [ ] Selection and Bookmarks panes visible
- [ ] Page view is Actual size

---

## Phase 2 — Source-data and existing-report audit

This phase is **already done**. The findings are recorded in
`01_Project_Guidance/AUDIT_FINDINGS.md` and `06_Validation_and_Release/RECONCILIATION_RESULTS.md`.
Re-run it only if the dataset changes.

Summary of what was established **[VERIFIED]**:

- 700 rows, 16 months (Sep 2021 – Dec 2022), monthly grain, no nulls in any column.
- 5 segments, 5 countries, 6 products, 4 discount bands, 25 managers.
- Report has 11 pages (7 visible + 4 tooltip pages) and 13 bookmarks.
- Home page: 66 containers, 54 leaf visuals, covering 25.8% of the canvas.
- Navigation is entirely bookmark-driven; no page-navigation actions exist.

---

## Phase 3 — Existing measure and model validation

### 3.1 What is known about the model

**[VERIFIED]** Five tables: `FACT_SALES`, `DIM_DATE`, `DIM_MANAGERS`, `Measure`,
`LAST_REFRESHED`.

**[VERIFIED]** 24 measure names are referenced by the report. Full list in
`METRIC_DEFINITIONS.md`.

**[UNKNOWN]** No measure's DAX could be read. The `DataModel` part of the PBIX is
XPress9-compressed and contains no readable text. The `Measures` sheet in your workbook
is empty.

### 3.2 Reconcile the base measures — do this first

Open the report with no filters applied and confirm each measure against the source:

| Measure | Expected value |
|---|---|
| `M_Units Sold` | 1,125,788 |
| `M_Gross Sales` | 127,931,598.50 |
| `M_Discounts` | 9,205,248.24 |
| `M_Net Sales` | 118,726,350.26 |
| `M_Cost Of Goods` | 101,832,648.00 |
| `M_Profit` | 16,893,702.26 |

**[CALCULATED]** from `Sales_Dataset.xlsx`, sheet `Sales`, 700 rows, summed by column.

If any disagrees, stop. Every downstream figure depends on these.

### 3.3 Establish `M_Date Selection` behaviour **[UNTESTED]**

1. On Home, find the card showing `Sep/21 - Dec/22`.
2. Select one manager in the Manager slicer.
3. Observe whether the card changes.

| Result | Label the card |
|---|---|
| Unchanged | `REPORTING WINDOW` |
| Narrows | `SELECTED WINDOW` |
| Blank or error | Stop — replace it with a static text box reading `Sep 2021 – Dec 2022` |

### 3.4 Test the five new label measures **before** redesigning anything

Create the measures in `METRIC_DEFINITIONS.md` §"New measures", then drop each on a
**blank page** in a card and confirm:

| Measure | Expected, unfiltered |
|---|---|
| `M_Units Label` | `1,125,788 units` |
| `M_Discount Rate Label` | `7.2% of gross` |
| `M_COGS Ratio Label` | `85.8% of net sales` |
| `M_Margin Label` | `14.2% margin` |
| `M_Coverage Label` | `Across 5 segments and 5 countries` |

Then select one manager and confirm each changes. Delete the test page afterwards.

**Do not start Phase 5 until all five pass.** They are the only reason the redesigned
page stays truthful under a filter.

### Checkpoint 3
- [ ] Six base measures reconcile
- [ ] `M_Date Selection` behaviour recorded
- [ ] Five new measures exist and pass both tests
- [ ] Test page deleted

---

## Phase 4 — Theme and design system

### 4.1 Apply the theme

**View → Themes → Browse for themes →** select `Cream_Forest_Theme_v2.json`.

### 4.2 Understand what the theme cannot do

A theme sets defaults for **native visuals only**. It never overrides explicit
per-visual formatting, and it never reaches shapes, buttons, text boxes or images.

**[VERIFIED]** from your PBIX, these are set explicitly and the theme cannot touch them:

| What | Count | Property | Value |
|---|---|---|---|
| Slicers | 12 | `items.background` | `#07345F` |
| Bar charts | 14 | `dataPoint` **gradient fill rule** | 7× `#0F76D7`→`#094780`, 7× `#F4A08A`→`#EE6C4D` |
| Shapes | 11 | `fill.fillColor` | `#07345F` |
| Buttons (Help page) | 3 | `fill.fillColor` | `#DDA705` |

The bar charts are the significant one. A **gradient fill rule** on the data point
overrides `dataColors` completely, which is why bars stayed blue and coral through
every theme version.

Clearing procedure is in `04_Design_Assets/THEME_APPLICATION_NOTES.md`.

### Checkpoint 4
- [ ] Theme applied
- [ ] You have read the list of what must be cleared by hand

---

## Phase 5 — Home page redesign

Full click-by-click procedure: `02_Implementation_SOP/HOME_PAGE_SOP.md`.
Complete object inventory: `02_Implementation_SOP/OBJECT_INVENTORY.md`.

The sequence is:

1. Duplicate `Home` → `Home NEW`. **The original stays intact until acceptance passes.**
2. Ungroup, then delete/retain per the inventory — by name, in the Selection pane.
3. Build the masthead (forest band, nav, slicer).
4. Build the title band.
5. Build the P&L strip (panel, 5 stations, 4 operators).
6. Build the Volume and Scope panels.
7. Build the footer (forest band — this is what makes the white-only social icons work).
8. Prototype **one** transparent button and test it, then copy it.
9. Set layer order; verify by clicking.
10. Update bookmarks.

### Checkpoint 5
- [ ] `Home NEW` built; `Home` untouched
- [ ] Every object in the inventory is accounted for

---

## Phase 6 — Exact Power BI actions

In `HOME_PAGE_SOP.md`. Key cross-cutting rules:

**Position and size.** Format pane → **General → Properties → Position** tab (X, Y) and
**Size** tab (Width, Height). **[VERIFIED]** against Microsoft documentation. Type the
value and press `Tab`; clicking away can discard it.

**Fonts.** Theme fonts are `Segoe UI` and `Segoe UI Semibold`. Text boxes do **not**
inherit theme fonts — set them by name each time. Choose `Segoe UI Semibold` from the
font list rather than pressing **B**.

**Buttons.** Format button pane → **Button** tab → **Action** → `On` → **Type** →
`Bookmark`. **[VERIFIED]** against Microsoft documentation.

**Testing buttons in Desktop.** Hold `Ctrl` and click. **[VERIFIED]**

**Transparent buttons.** Fill `On` at Transparency `100`, **not** Fill `Off`.
**[UNTESTED — prototype first, see HOME_PAGE_SOP §9.2]**

### Checkpoint 6
- [ ] Every object sits at its specified coordinates
- [ ] No object is outside the 1280 × 720 frame

---

## Phase 7 — Testing, reconciliation, accessibility, acceptance

### 7.1 Reconciliation

Figures must match Phase 3.2 with no selection. Rounding note: at **1 decimal place**
the displayed chain adds up in 24 of 36 single-selection states; at 2 dp, 23; at 0 dp,
19 **[CALCULATED]**. No precision avoids it, which is why the strip carries the standard
disclosure "figures in millions; components may not sum exactly due to rounding".

### 7.2 Filter behaviour

- [ ] One manager selected → all five station figures, all four sub-lines, the Volume
      figure and the coverage line change together
- [ ] The four Scope lines and the `Source window` line do **not** change
- [ ] Clearing returns every figure to Phase 3.2 values

### 7.3 Accessibility

- [ ] Alt text set on every button and every card
- [ ] Selection pane → **Tab order** tab: decorative shapes removed from the sequence,
      the rest in reading order
- [ ] No meaning carried by colour alone — the semantic set uses ▲ ▼ – glyphs as well
- [ ] Text contrast: body `#54594F` and ink `#1F2A22` on panel `#E3DCCA` both exceed
      4.5:1 **[CALCULATED]**

### 7.4 Acceptance

- [ ] `Net Proft` appears nowhere
- [ ] All six nav routes land correctly and return
- [ ] All four drill routes land correctly
- [ ] Clear-filters button resets the slicer
- [ ] No product-level profit figure anywhere
- [ ] Checked at Actual size **and** Fit to page

**Only when every box is ticked:** rename `Home` → `Home OLD`, hide it, rename
`Home NEW` → `Home`, move it to first position, re-update both Home bookmarks.
Keep `Home OLD` hidden — not deleted — for one full review cycle.

---

## Phase 8 — Documentation and screenshots

### 8.1 Capture screenshots

With the report at Actual size, capture each page at 1280 × 720 and save as PNG:

```
docs/images/01_home.png
docs/images/02_sales_analysis_period.png
docs/images/03_sales_analysis_segment.png
docs/images/04_sales_analysis_country.png
docs/images/05_sales_analysis_product.png
docs/images/06_help.png
docs/images/07_data_model.png
```

Lower-case, zero-padded, underscore-separated. The README references exactly these paths.

### 8.2 Fill the placeholders

Every file in `03_GitHub_Documentation/` is complete except where marked
`<!-- CONFIRM: ... -->`. Search for that string and resolve each one. They are listed in
`MANIFEST.md`.

### Checkpoint 8
- [ ] Seven screenshots exist at the exact paths above
- [ ] No `CONFIRM` markers remain

---

## Phase 9 — GitHub preparation and publication

Full procedure: `GITHUB_RELEASE_CHECKLIST.md`.

Summary:

1. Clone your existing repository locally.
2. Create a branch: `git checkout -b redesign/home-page`.
3. Copy the documentation files to the repo root, screenshots to `docs/images/`.
4. Add `.gitignore` **before** the first `git add`.
5. Verify the PBIX size. If over 100 MB, GitHub will reject it — use Git LFS or publish
   a link instead.
6. `git add -A && git commit -m "Redesign Home page; add project documentation"`
7. `git push -u origin redesign/home-page`
8. Open a pull request, review the rendered Markdown on GitHub, then merge.
9. Tag the release: `git tag -a v2.0 -m "Home page redesign" && git push --tags`

**Check every relative image path renders on GitHub before merging.** Broken image
links are the most common defect in a BI portfolio repository, and your current README
already has six.

---

## Phase 10 — Review, release, rollback, maintenance

### 10.1 Release

- [ ] README renders correctly on GitHub, all images visible
- [ ] Repository description and topics set
- [ ] Release tagged

### 10.2 Rollback

See `06_Validation_and_Release/ROLLBACK_PROCEDURE.md`. Three independent routes:
the dated backup PBIX, the hidden `Home OLD` page, and the Git history.

### 10.3 Maintenance rules

- **Any object added to Home means re-updating both Home bookmarks.** Every bookmark in
  this report captures all visuals on its page, so a later addition falls outside the
  captured state and will disappear when the clear button is used. **[VERIFIED]**
- Changing the theme does **not** restyle the shapes. The masthead, footer, three
  panels, three dividers and the gold rule hold their colours by hand.
- If the dataset changes, revisit the four statements in the Scope panel first.
- The 25 manager images are hosted on an external host (`i.ibb.co`). If that host goes
  away the images break. **[VERIFIED]** from the `Managers` sheet.
