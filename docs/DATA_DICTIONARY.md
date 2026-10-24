# Data Dictionary

Source: `Sales_Dataset.xlsx`. All contents below read directly from the workbook.
**[VERIFIED]**

---

## Workbook sheets

| Sheet | Rows | Purpose |
|---|---|---|
| `Sales` | 700 data rows + header | The fact table |
| `Managers` | 25 data rows + header | Manager dimension |
| `Refreshed` | empty | Refresh timestamp placeholder |
| `Measures` | empty | Placeholder; contains no DAX |

---

## Sheet: `Sales` — 700 rows, no nulls in any column

| Column | Type | Distinct | Values / range | Notes |
|---|---|---|---|---|
| `Segment` | Text | 5 | Channel Partners, Enterprise, Government, Midmarket, Small Business | |
| `Country` | Text | 5 | Canada, France, Germany, Mexico, United States of America | |
| `Product` | Text | 6 | Amarilla, Carretera, Montana, Paseo, VTT, Velo | |
| `Discount Band` | Text | 4 | None, Low, Medium, High | |
| `Units Sold` | Decimal | — | Total 1,125,788 | |
| `Manufacturing Price` | Currency | — | — | **Does not reconcile with `COGS`** — see warning below |
| `Sale Price` | Currency | 7 | 7, 12, 15, 20, 125, 300, 350 | Determines cost per unit |
| `Gross Sales` | Currency | — | Total $127,931,598.50 | `Units Sold × Sale Price` |
| `Discounts` | Currency | — | Total $9,205,248.24 | |
| `Sales` | Currency | — | Total $118,726,350.26 | Net sales. `Gross Sales − Discounts` |
| `COGS` | Currency | — | Total $101,832,648.00 | Cost of goods sold |
| `Profit` | Currency | — | Total $16,893,702.26 | `Sales − COGS` |
| `Date` | Date | 16 | 2021-09-01 → 2022-12-01 | First of month; monthly grain |
| `Month Number` | Whole | 12 | 1–12 | |
| `Month Name` | Text | 12 | January … December | |
| `Year` | Whole | 2 | 2021, 2022 | |
| `Manager ID` | Text | 25 | NL01 … NL25 | Joins to `Managers` |

> **Naming note.** The column called `Sales` in the workbook is **net sales**, after
> discount. In the model it surfaces as `M_Net Sales`. The ambiguity is in the source,
> not the model.

### Grain

One row per `Segment` × `Country` × `Product` × `Discount Band` × `Date`. Monthly.
There is **no transaction-level or customer-level record.**

### Warning: `Manufacturing Price` does not reconcile with `COGS`

Cost per unit (`COGS ÷ Units Sold`) is a deterministic function of `Sale Price`:

| Sale Price | COGS per unit |
|---|---|
| $7 | $5 |
| $12 | $3 |
| $15 | $10 |
| $20 | $10 |
| $125 | $120 |
| $300 | $250 |
| $350 | $260 |

Only **164 of 700 rows** have `COGS ÷ Units Sold = Manufacturing Price`.

**[CALCULATED]** by grouping all 700 rows on `Sale Price` and computing
`COGS / Units Sold` per row. Residual variation is floating-point noise below $0.25.

**Consequence:** `Manufacturing Price` cannot be used as a unit cost, and no
product-level profitability can be derived. See
[`KNOWN_LIMITATIONS.md`](KNOWN_LIMITATIONS.md).

---

## Sheet: `Managers` — 25 rows

| Column | Type | Notes |
|---|---|---|
| `Manager ID` | Text | NL01 … NL25. Primary key |
| `Manager Name` | Text | Full name |
| `First Name` | Text | |
| `Last Name` | Text | |
| `Territory` | Text | `<Segment>-<Country>`, e.g. `Channel Partners-Canada` |
| `Image URL` | Text | External, hosted on `i.ibb.co` |

`Territory` is a concatenation of the segment and country a manager covers. The 25
managers are exactly the 5 × 5 segment-country combinations.

> **External dependency.** The 25 manager photographs are hosted on a third-party image
> host. If that host becomes unavailable the images break in the report and there is no
> local fallback. Consider embedding them or removing the dependency before publishing.

---

## Power BI model tables **[VERIFIED]** from `DiagramLayout` and layout references

| Table | Role |
|---|---|
| `FACT_SALES` | The 700-row fact table |
| `DIM_DATE` | Date dimension |
| `DIM_MANAGERS` | Manager dimension |
| `Measure` | Measures-only table; all measures prefixed `M_` |
| `LAST_REFRESHED` | Refresh timestamp |

### Columns referenced by the report

| Table | Columns in use |
|---|---|
| `FACT_SALES` | `Segment`, `Segment Name Short`, `Country Name Short`, `Product`, `Product Name`, `Discount Band` |
| `DIM_DATE` | `Year`, `Quarter`, `Month`, `YearMonth`, `Reporting Period` |
| `DIM_MANAGERS` | `Manager Name`, `Image URL` |

`FACT_SALES` carries both long and short name columns for Segment, Country and Product.
The short columns are load-bearing: they are used where axis labels must fit a narrow
chart. Do not delete them as duplicates.

**[UNKNOWN]** Column data types, calculated columns, relationship cardinality and
Power Query steps could not be inspected. The `DataModel` part of the PBIX is
XPress9-compressed and contains no readable text.
