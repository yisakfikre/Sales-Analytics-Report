# Methodology

How each finding was produced, so it can be checked or re-run.

---

## Tools

| Step | Tool |
|---|---|
| Source profiling and arithmetic | Python 3, `openpyxl` |
| Statistical tests | `scipy`, `numpy` |
| Report structure inspection | Python, parsing `Report/Layout` as UTF-16 JSON |
| Colour and contrast | Python, WCAG 2.1 and CIELAB formulae |
| Report build | Power BI Desktop, DAX, Power Query |

---

## 1. Reading the PBIX

A PBIX is a ZIP archive. Extracting it gives:

| Part | Readable? | Contains |
|---|---|---|
| `Report/Layout` | **Yes** | UTF-16 JSON: pages, visuals, positions, bookmarks, formatting |
| `DiagramLayout` | Yes | Model diagram table list |
| `Report/StaticResources/` | Yes | Images and icons |
| `Metadata`, `Settings`, `Version` | Yes | Small metadata |
| `DataModel` | **No** | XPress9-compressed; no readable text |

`Report/Layout` is the evidence base for every structural claim in this project —
page and visual counts, positions, bookmark configuration, hard-coded colours.

`DataModel` is why no measure DAX is documented. This limitation is stated wherever it
matters rather than filled with plausible-looking guesses.

---

## 2. Source profiling

All 700 rows read from `Sales_Dataset.xlsx`, sheet `Sales`. Distinct values counted per
column; nulls counted per column; column totals summed. Results in
[`DATA_VALIDATION.md`](DATA_VALIDATION.md) §1–2.

---

## 3. Cost structure

**Question:** can product profitability be derived?

**Method.** Group all 700 rows by `Sale Price`. Within each group compute
`COGS ÷ Units Sold` per row and examine the spread. Separately, count rows where
`COGS ÷ Units Sold` equals `Manufacturing Price`.

**Result.** Cost per unit is constant within each of the seven price tiers (residual
variation below $0.25, consistent with floating-point noise) and unrelated to product.
Only 164 of 700 rows reconcile with `Manufacturing Price`.

**Conclusion.** Cost is a function of price tier. Product-level profit would be an
artefact of which tier a product sold in. Product profitability is excluded.

**Why this matters methodologically.** The obvious move — divide profit by product — is
available in the data and produces numbers. Checking whether those numbers *mean*
anything is the step that distinguishes analysis from reporting.

---

## 4. Seasonality

**Question:** is the monthly revenue pattern seasonal demand?

**Method.** Aggregate revenue by month. Count source rows per month. Correlate the two.
Then compute revenue *per record* per month and compare peak months against the rest.

**Result.** r = 0.871 between monthly revenue and monthly record count. Revenue per
record is **lower** in the apparent peak months.

**Conclusion.** The pattern is sampling density in the source file, not demand. If it
were demand, revenue per record would be flat or higher in peaks, not lower.

---

## 5. Forecastability

**Question:** can this series be forecast usefully?

**Method.** Rolling-origin backtest on the monthly revenue series. Two candidates — flat
mean and seasonal naïve — scored with MASE, which is scale-free and handles the
near-zero values a percentage error metric would distort.

**Result.**

| Method | MASE |
|---|---|
| Flat mean | 0.83 |
| Seasonal naïve | 1.25 |

Coverage: 8 of 12 calendar months appear exactly once; no month appears three times.

**Conclusion.** No exploitable temporal structure. A seasonal method scoring worse than
a flat average is the diagnostic. With 16 observations and single-observation months, no
seasonal term can be estimated, let alone validated.

**Why no forecast was published.** Power BI will happily draw a projected line on this
series. The line would look authoritative. Publishing it would be the single least
honest thing this project could do.

---

## 6. The break-even identity

**Question:** what actually determines whether a segment is profitable?

**Method.** For each segment, compute gross margin percentage and realised discount
rate, then test whether `profit > 0` coincides with `gross margin % > discount rate`.

**Result.** The relationship holds for all five segments. It is an identity, not a
correlation — it follows from the definitions.

**Enterprise break-even cap.** Solve for the discount cap at which Enterprise profit
crosses zero, using `scipy.optimize.brentq` over the row-level capping function.
**Result: 5.29%.**

**Capping must be done at row level.** `SUMX` over fact rows with
`MIN(Discounts, cap × Gross Sales)`, never on the aggregate — capping an aggregate lets
a heavily discounted row hide behind a lightly discounted one.

**Interpretation note.** A cap is a ceiling, not a target. Setting discounts *to* 5.29%
is not what the number says.

---

## 7. Concentration

**Method.** Herfindahl index on product revenue share.

**Result.** 1,835, against 1,667 for a perfectly even six-product split.

**Conclusion.** Near-even. No concentration finding exists, so none is published. A
Pareto chart here would present an impression of concentration that the data does not
contain.

---

## 8. Year-on-year

**Constraint.** `DIM_DATE` has no column named `Date`, so standard time intelligence
(`DATEADD`, `SAMEPERIODLASTYEAR`) cannot be used. Period shifting is done on the integer
`MonthIndex` with `SELECTCOLUMNS` and `TREATAS`.

**Constraint.** Only 4 of 16 months have a prior-year equivalent.

**Method.** The measure determines its own like-for-like window — the subset of
in-context months whose `MonthIndex − 12` falls within the data — rather than depending
on a date slicer, because the report has none. Full DAX in
[`METRIC_DEFINITIONS.md`](METRIC_DEFINITIONS.md).

**Result.** +40.1% over the 4 comparable months, unfiltered.

**Why not compare all 16 months.** That would compare 16 months against the 4 that
overlap — a partial-period comparison presented as a full one.

---

## 9. Display rounding

**Question:** at what precision does the displayed chain add up?

**Method.** Enumerate 36 single-selection states: unfiltered, each of the 21 managers
present in the fact table, each of 5 segments, each of 5 countries. For each, round the
five chain figures to N decimals in millions and test both identities on the **rounded**
values.

**Result.**

| N | States passing |
|---|---|
| 0 | 19 / 36 |
| 1 | 24 / 36 |
| 2 | 23 / 36 |

**Conclusion.** One decimal place fails least often; no precision avoids the problem,
including whole dollars. The page uses 1 dp and carries the standard disclosure.

**Why this was worth testing.** The original intuition was that two decimals would be
"more accurate" and one decimal "cleaner". Both were wrong about what matters — whether
a reader doing the arithmetic in their head gets the right answer.

---

## 10. Accessibility

**Contrast.** sRGB → linear → relative luminance → WCAG 2.1 ratio, computed for every
text-on-surface and data-on-surface pair in the palette.

**Colour vision.** Viénot deuteranopia and protanopia transforms applied to the
good/bad/neutral set, then ΔE in CIELAB between the transformed colours.

**Result.** No three-colour semantic set remains distinguishable across all CVD types on
hue alone. Therefore ▲ ▼ – glyphs accompany every semantic colour. This is a design
constraint derived from measurement, not a stylistic preference.

---

## Evidence classification

Every material claim in this documentation carries one of:

| Tag | Meaning |
|---|---|
| **VERIFIED** | Read directly from the PBIX, the workbook, or vendor documentation |
| **CALCULATED** | Computed from source data; method stated |
| **INFERRED** | A reasonable interpretation, not directly confirmed |
| **UNTESTED** | Behaviour requiring testing in Power BI |
| **UNKNOWN** | Cannot be determined from available evidence |

Where uncertainty affects a recommendation, the recommendation is stated conditionally
rather than asserted.
