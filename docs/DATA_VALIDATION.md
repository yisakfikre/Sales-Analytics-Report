# Data Validation

Evidence for every figure the report publishes. Each result states its method so it can
be re-run.

**What was validated here:** the source workbook, and the arithmetic derived from it.
**What was not:** the report's own rendering. That requires Power BI Desktop and is
covered by the acceptance checklist in `IMPLEMENTATION_SOP.md` Phase 7.

---

## 1. Source profile **[VERIFIED]**

Method: read `Sales_Dataset.xlsx` with `openpyxl`, sheet `Sales`, all 700 data rows.

| Check | Result |
|---|---|
| Row count | 700 |
| Null values, any column | **0** |
| Distinct segments | 5 |
| Distinct countries | 5 |
| Distinct products | 6 |
| Distinct discount bands | 4 |
| Distinct manager IDs | 25 |
| Distinct dates | 16 |
| Date range | 2021-09-01 → 2022-12-01 |
| Date grain | First of month, no gaps |

---

## 2. Column totals **[CALCULATED]**

Method: sum each column across all 700 rows.

| Column | Total |
|---|---|
| `Units Sold` | 1,125,788 |
| `Gross Sales` | 127,931,598.50 |
| `Discounts` | 9,205,248.24 |
| `Sales` (net) | 118,726,350.26 |
| `COGS` | 101,832,648.00 |
| `Profit` | 16,893,702.26 |

**These are the reconciliation targets.** Any measure in the report that claims to
represent one of these must return exactly this value with no filter applied.

---

## 3. Internal identities **[CALCULATED]**

Both hold exactly, to the cent:

```
Gross Sales − Discounts = Net Sales
127,931,598.50 − 9,205,248.24 = 118,726,350.26   difference: 0.00  ✓

Net Sales − COGS = Profit
118,726,350.26 − 101,832,648.00 = 16,893,702.26  difference: 0.00  ✓
```

Derived ratios:

| Ratio | Value |
|---|---|
| Discount rate (Discounts ÷ Gross Sales) | 7.2% |
| COGS as share of net sales | 85.8% |
| Profit margin (Profit ÷ Net Sales) | 14.2% |

---

## 4. Cost structure **[CALCULATED]**

Method: group all 700 rows on `Sale Price`; compute `COGS ÷ Units Sold` per row.

| Sale price | COGS per unit | Rows |
|---|---|---|
| $7 | $5.00 | — |
| $12 | $3.00 | — |
| $15 | $10.00 | — |
| $20 | $10.00 | — |
| $125 | $120.00 | — |
| $300 | $250.00 | — |
| $350 | $260.00 | — |

Residual variation within each price tier is below $0.25 per unit and is
floating-point noise, not real cost variation.

**Cross-check:** rows where `COGS ÷ Units Sold = Manufacturing Price` — **164 of 700**.

**Conclusion:** cost is driven by price tier, not by product. This is the evidence that
rules out product profitability.

---

## 5. Seasonality **[CALCULATED]**

Method: aggregate revenue by month; correlate with the count of source rows per month.

| Statistic | Value |
|---|---|
| Correlation, monthly revenue vs record count | **r = 0.871** |
| Revenue per record in apparent peak months | **Lower** than outside them |

**Conclusion:** the monthly shape is sampling density, not demand. Classified as a data
artefact, not a business finding.

---

## 6. Forecastability **[CALCULATED]**

Method: rolling-origin backtest on the monthly revenue series, scored with MASE.

| Method | MASE |
|---|---|
| Flat mean | **0.83** |
| Seasonal naïve | 1.25 |

Coverage check: **8 of 12 calendar months appear only once** in the data. No month
appears three times.

**Conclusion:** forecasting is not supported. A seasonal-naïve method scoring worse than
a flat mean indicates no exploitable structure.

---

## 7. Break-even identity **[CALCULATED]**

For every segment, profit is positive exactly when gross margin percentage exceeds the
realised discount rate. Verified across all five segments; the relationship is an
identity, not a correlation.

| Segment | Result |
|---|---|
| Channel Partners | Holds |
| Enterprise | Holds — the only loss-making segment |
| Government | Holds |
| Midmarket | Holds |
| Small Business | Holds |

**Enterprise break-even discount cap: 5.29%.** Method: solve for the cap at which
segment profit crosses zero, using `scipy.optimize.brentq` on the row-level capping
function. Note that a cap is a ceiling, not a target.

---

## 8. Concentration **[CALCULATED]**

| Statistic | Value |
|---|---|
| Herfindahl index, product revenue | 1,835 |
| Herfindahl index, perfectly even six-product split | 1,667 |

**Conclusion:** near-even distribution. A concentration or Pareto analysis produces no
finding, which is why none is published.

---

## 9. Display rounding **[CALCULATED]**

Method: for each of 36 single-selection states — unfiltered, each of the 21 managers
present in the fact table, each of 5 segments, each of 5 countries — round the five
chain figures to N decimal places in millions and test whether both identities still
hold on the **displayed** values.

| Decimal places | States where the chain adds up |
|---|---|
| 0 | 19 of 36 |
| **1** | **24 of 36** |
| 2 | 23 of 36 |

**Conclusion:** one decimal place fails least often, and no precision avoids the
problem. The page uses 1 dp and discloses it. This is the standard treatment in
published financial statements.

---

## 10. Report structure **[VERIFIED]**

Method: parse `Report/Layout` from the PBIX (UTF-16 JSON).

| Check | Result |
|---|---|
| Pages | 11 — 7 visible, 4 tooltip |
| Bookmarks | 13 |
| Home page containers | 66 — 12 groups, 54 leaf visuals |
| Home page canvas coverage | **25.8%** |
| Largest contiguous empty block on Home | 1005 × 354 px at (0, 310) = **38.6% of canvas** |
| Sales Analysis page visuals | 92 |
| Measures referenced | 24 |
| Static resources registered | 60 |

---

## 11. Theme reachability **[VERIFIED]**

Method: scan `Report/Layout` for hard-coded colour literals by visual type and property.

| Visual type | Count | Property | Value |
|---|---|---|---|
| slicer | 12 | `items.background` | `#07345F` |
| clusteredBarChart | 7 | `dataPoint` gradient | `#0F76D7` → `#094780` |
| clusteredBarChart | 7 | `dataPoint` gradient | `#F4A08A` → `#EE6C4D` |
| shape | 11 | `fill.fillColor` | `#07345F` |
| actionButton | 3 | `fill.fillColor` | `#DDA705` |

**Conclusion:** 33 visuals carry explicit formatting a theme cannot override. The
gradient fill rules on the 14 bar charts are the reason `dataColors` had no effect
through four theme revisions.

---

## 12. Accessibility **[CALCULATED]**

Method: sRGB → linear → relative luminance, WCAG 2.1 contrast ratio.

| Pair | Ratio | WCAG |
|---|---|---|
| Ink `#1F2A22` on panel `#E3DCCA` | > 4.5:1 | AA text ✓ |
| Secondary `#54594F` on panel `#E3DCCA` | > 4.5:1 | AA text ✓ |
| Ramp step 1 `#1F3B2C` on panel | 8.93:1 | ✓ |
| Ramp step 2 `#2C543B` on panel | 6.30:1 | ✓ |
| Ramp step 3 `#396A4C` on panel | 4.60:1 | ✓ |
| Ramp step 4 `#4A7A5A` on panel | 3.64:1 | AA non-text ✓ |
| Ramp step 5 `#52805B` on panel | 3.34:1 | AA non-text ✓ |

Colour-vision check: no three-colour semantic set survives all CVD types on hue alone,
so the good/bad/neutral set is always paired with ▲ ▼ – glyphs. Tested with Viénot
deuteranopia and protanopia transforms.

---

## What remains untested

| Item | Why | How to test |
|---|---|---|
| `M_Date Selection` behaviour under filter | DAX not readable | SOP Phase 3.3 |
| Clear-filters button | Bookmark semantics inferred from JSON | SOP Phase 3, test 1.2 |
| Drill-route filter consistency | Same | SOP Phase 3, test 1.3 |
| Transparent-button click behaviour | Undocumented | SOP Phase 6, prototype test |
| Every existing measure's DAX | Compressed in the PBIX | Open the PBIX and read the formula bar |
| Power Query steps | Not readable | Open Power Query Editor |
| Relationship cardinality | Not readable | Open Model view |
