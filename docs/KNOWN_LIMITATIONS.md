# Known Limitations

What this dataset and report cannot support, and why. Each item is evidence-backed.

---

## 1. The data is a sample, not real business data

This is a variant of Microsoft's publicly distributed **Financial Sample** workbook,
extended with a 25-row manager dimension.

**No finding in this report describes a real business outcome.** It demonstrates
analytical method on a known dataset. Any figure quoted is a property of the sample.

This is stated on the landing page as well as here. A portfolio project that conceals
its data provenance invites a reviewer to discover it, which is worse than disclosing it.

---

## 2. Product profitability cannot be derived

Cost per unit is a deterministic function of **sale price**, not of product:

| Sale price | Cost per unit | Implied gross margin |
|---|---|---|
| $7 | $5 | 28.6% |
| $12 | $3 | 75.0% |
| $15 | $10 | 33.3% |
| $20 | $10 | 50.0% |
| $125 | $120 | 4.0% |
| $300 | $250 | 16.7% |
| $350 | $260 | 25.7% |

Only **164 of 700 rows** have `COGS ÷ Units Sold = Manufacturing Price`.

**[CALCULATED]** by grouping all 700 rows on `Sale Price` and computing `COGS / Units
Sold`. Residual variation is floating-point noise below $0.25 per unit.

**Consequence.** A product's apparent profitability is an artefact of which price tier
it happened to sell in, not of what it costs to make. The report shows product
**revenue** and **volume** and stops there.

The `Manufacturing Price` column must not be used as a unit cost.

---

## 3. Customer-level analysis is impossible

The source carries no customer identifier. The finest grain available is
segment × country × product × discount band × month.

**Not possible:** customer concentration, repeat purchase, cohort analysis, customer
lifetime value, churn, basket analysis.

Segment-level data does not permit customer-level conclusions. A "segment" here is one
of five market categories, not a customer.

---

## 4. Forecasting is not supported

Tested and rejected on evidence, not omitted for lack of effort.

| Method | MASE |
|---|---|
| Flat mean | **0.83** |
| Seasonal naïve | 1.25 |

**[CALCULATED]** by rolling-origin backtest on the monthly revenue series.

A seasonal-naïve forecast performing *worse* than a flat average is a signal that there
is no exploitable temporal structure to forecast.

Compounding reasons:

- 16 monthly observations total.
- **8 of 12 calendar months appear only once**, so no seasonal term can be estimated.
- No month has three observations, so no seasonal index can be validated.

A forecast line drawn on this data would look authoritative and mean nothing.

---

## 5. The apparent seasonality is a sampling artefact

Monthly revenue correlates with the **number of records in that month** at **r = 0.871**.
**[CALCULATED]**

Revenue *per record* is lower in the apparent peak months than outside them. The peaks
are density in the source file, not demand.

**Consequence.** Nothing in this report should be read as evidence of a seasonal
pattern, and no inventory or staffing decision should be taken from the monthly shape.

---

## 6. Year-on-year comparison covers 4 months, not 16

The data runs September 2021 to December 2022. Only **September–December 2022** have a
prior-year equivalent.

Any year-on-year figure is computed on that 4-month like-for-like window and labelled as
such. Comparing the full 16 months against the overlapping 4 would be a partial-period
comparison presented as a full one.

Unfiltered year-on-year profit over the comparable window: **+40.1%**. **[CALCULATED]**

---

## 7. Revenue concentration shows nothing

Herfindahl index across the six products: **1,835**, against **1,667** for a perfectly
even split. **[CALCULATED]**

The products are near-evenly distributed. A concentration or Pareto analysis on this
dataset produces no finding, which is why the report does not include one.

---

## 8. The model's internals could not be inspected

**[UNKNOWN]** The `DataModel` part of the PBIX is XPress9-compressed. Not readable:

- DAX of any existing measure
- Calculated columns
- Relationship cardinality and cross-filter direction
- Power Query / M transformation steps
- Row-level security, if any

Measure **names** are known because the report layout references them, and six measures'
**values** are confirmed because they reconcile against the source.

Anyone extending this report should open the PBIX and document the DAX directly. The
placeholders in `METRIC_DEFINITIONS.md` mark exactly where.

---

## 9. External image dependency

The 25 manager photographs are referenced by URL on a third-party host (`i.ibb.co`).
If that host becomes unavailable the images break and there is no local fallback.
**[VERIFIED]** from the `Managers` sheet.

---

## 10. Known defects in the current report

| Defect | Evidence | Status |
|---|---|---|
| `Net Proft` typo on the Home landing page | Hard-coded in `Button_Net_Profit` | Fixed by the redesign |
| Clear-filters button does not clear filters | `Home Default` bookmark has `suppressData: true` | **[UNTESTED]** — verify, then fix |
| One drill route discards the filter, three preserve it | `COGs` bookmark has Data on; the other three have it off | **[UNTESTED]** — verify, then make consistent |
| 14 bar charts ignore the theme | Gradient fill rule on `dataPoint` overrides `dataColors` | Must be cleared per visual |
| 12 slicers ignore the theme | Explicit `items.background` = `#07345F` | Must be cleared per visual |
| Six broken image links in the current README | — | Fixed by the new README once screenshots are captured |

---

## 11. Rounding

The Home page presents an arithmetic chain. Rounded figures do not always satisfy it:

| Decimal places | States where the displayed chain adds up |
|---|---|
| 0 | 19 of 36 |
| 1 | **24 of 36** |
| 2 | 23 of 36 |

**[CALCULATED]** across all 36 single-selection states: unfiltered, each of 21 managers
present in the fact table, each of 5 segments, each of 5 countries.

No precision avoids it, including whole dollars. The page uses one decimal place and
carries the standard disclosure: *figures in millions; components may not sum exactly
due to rounding*.

---

## What this project does **not** claim

- That any figure describes a real company.
- That product profitability is known.
- That anything about customers can be inferred.
- That future sales can be predicted from this data.
- That the monthly pattern reflects seasonal demand.
