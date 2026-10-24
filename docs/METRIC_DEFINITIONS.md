# Metric Definitions

---

## Important: the DAX of existing measures is not documented here

**[UNKNOWN]** The `DataModel` part of the PBIX is XPress9-compressed and contains no
readable text. The `Measures` sheet in the source workbook is empty. No existing
measure's definition could be read.

What **is** verified: the measure **names**, because the report layout references them,
and the **values** six of them return, because those reconcile against the source.

**To complete this file**, open the PBIX, select each measure in the Data pane, copy its
formula from the formula bar, and paste it into the DAX column below. That takes about
fifteen minutes and turns this from a list into documentation.

---

## Existing measures — 24 referenced by the report **[VERIFIED: names]**

| Measure | Used | Business meaning | DAX |
|---|---|---|---|
| `M_Units Sold` | 126× | Total units sold | <!-- CONFIRM --> |
| `M_Gross Sales` | 50× | Revenue before discount | <!-- CONFIRM --> |
| `M_Discounts` | 45× | Total discount given | <!-- CONFIRM --> |
| `M_Net Sales` | 128× | Revenue after discount | <!-- CONFIRM --> |
| `M_Cost Of Goods` | 123× | Cost of goods sold | <!-- CONFIRM --> |
| `M_Profit` | 140× | Net sales less cost of goods | <!-- CONFIRM --> |
| `M_Profit Margin` | 33× | Profit as a share of net sales | <!-- CONFIRM --> |
| `M_Profit Margin Formatted` | 55× | Above, as display text | <!-- CONFIRM --> |
| `M_100% Minus Profit Margin` | 16× | Complement, for stacked bars | <!-- CONFIRM --> |
| `M_Profit YOY% Formatted` | 10× | Year-on-year profit change | See "Phase 1 measure" below |
| `M_Profit1` | 4× | Unclear — possibly a duplicate | <!-- CONFIRM: is this still needed? --> |
| `M_# of Distinct Segments` | 6× | Count of segments in context | <!-- CONFIRM --> |
| `M_# of Distinct Countries` | 6× | Count of countries in context | <!-- CONFIRM --> |
| `M_Date Selection` | 62× | Reporting window as text | <!-- CONFIRM --> |
| `M_Discount Band Colors` | 5× | Conditional formatting colour | <!-- CONFIRM --> |
| `M_Transparent` | 25× | Returns a transparent colour | <!-- CONFIRM --> |
| `M_User Greeting` | 10× | Greeting text | <!-- CONFIRM --> |
| `M_Welcome_All` | 2× | Greeting text | <!-- CONFIRM --> |
| `M_Units Sold_[DIM_DATE.YearMonth]` | 16× | Sparkline variant | Auto-generated |
| `M_Gross Sales_[DIM_DATE.YearMonth]` | 8× | Sparkline variant | Auto-generated |
| `M_Discounts_[DIM_DATE.YearMonth]` | 8× | Sparkline variant | Auto-generated |
| `M_Net Sales_[DIM_DATE.YearMonth]` | 16× | Sparkline variant | Auto-generated |
| `M_Cost Of Goods_[DIM_DATE.YearMonth]` | 16× | Sparkline variant | Auto-generated |
| `M_Profit_[DIM_DATE.YearMonth]` | 14× | Sparkline variant | Auto-generated |

> `M_Profit1` is referenced only four times and its name suggests an accidental
> duplicate of `M_Profit`. Check whether it is still needed; duplicate measures that
> represent the same business metric are a maintenance liability.

---

## Verified values

These six reconcile exactly against the source workbook with no filter applied.
**[CALCULATED]** by summing the matching column across all 700 rows.

| Measure | Value |
|---|---|
| `M_Units Sold` | 1,125,788 |
| `M_Gross Sales` | 127,931,598.50 |
| `M_Discounts` | 9,205,248.24 |
| `M_Net Sales` | 118,726,350.26 |
| `M_Cost Of Goods` | 101,832,648.00 |
| `M_Profit` | 16,893,702.26 |

The two identities hold exactly, to the cent:

```
Gross Sales − Discounts = Net Sales
127,931,598.50 − 9,205,248.24 = 118,726,350.26  ✓

Net Sales − Cost of Goods = Profit
118,726,350.26 − 101,832,648.00 = 16,893,702.26  ✓
```

---

## Year-on-year — the model has no Date column

`DIM_DATE` carries no column named `Date`, so `DATEADD` and the standard time
intelligence functions cannot be used. Period shifting is done on the integer
`MonthIndex` with `TREATAS`.

Only **4 of 16 months** have a prior-year equivalent (Sep–Dec 2022 against Sep–Dec
2021), so any year-on-year figure must restrict itself to a like-for-like window or it
compares four months against sixteen.

```dax
M_LFL Months =
VAR _AllDataMonths =
    CALCULATETABLE ( VALUES ( DIM_DATE[MonthIndex] ), REMOVEFILTERS ( DIM_DATE ), FACT_SALES )
VAR _FirstDataMonth = MINX ( _AllDataMonths, DIM_DATE[MonthIndex] )
VAR _SelectedMonths = CALCULATETABLE ( VALUES ( DIM_DATE[MonthIndex] ), FACT_SALES )
RETURN
    COUNTROWS ( FILTER ( _SelectedMonths, DIM_DATE[MonthIndex] - 12 >= _FirstDataMonth ) )
```

```dax
M_Profit YoY % =
VAR _AllDataMonths =
    CALCULATETABLE ( VALUES ( DIM_DATE[MonthIndex] ), REMOVEFILTERS ( DIM_DATE ), FACT_SALES )
VAR _FirstDataMonth = MINX ( _AllDataMonths, DIM_DATE[MonthIndex] )
VAR _SelectedMonths = CALCULATETABLE ( VALUES ( DIM_DATE[MonthIndex] ), FACT_SALES )
VAR _LFLMonths = FILTER ( _SelectedMonths, DIM_DATE[MonthIndex] - 12 >= _FirstDataMonth )
VAR _CurSet = SELECTCOLUMNS ( _LFLMonths, "MonthIndex", DIM_DATE[MonthIndex] )
VAR _PYSet  = SELECTCOLUMNS ( _LFLMonths, "MonthIndex", DIM_DATE[MonthIndex] - 12 )
VAR _Cur = CALCULATE ( [M_Profit], REMOVEFILTERS ( DIM_DATE ), TREATAS ( _CurSet, DIM_DATE[MonthIndex] ) )
VAR _Pri = CALCULATE ( [M_Profit], REMOVEFILTERS ( DIM_DATE ), TREATAS ( _PYSet, DIM_DATE[MonthIndex] ) )
RETURN
    IF ( NOT ISEMPTY ( _LFLMonths ) && _Pri > 0, DIVIDE ( _Cur - _Pri, _Pri ) )
```

**Expected unfiltered result:** `+40.1%` over 4 comparable months. **[CALCULATED]**

The measure selects its own like-for-like window rather than depending on a date
slicer, because the report has none.

---

## New measures for the Home page redesign

Five text-returning measures. Each bakes its formatting in, so the sub-line under each
figure moves correctly when the Manager slicer changes. Without them the ratios would
be typed text that states a false percentage under every selection except "all".

Create them on the `Measure` table, display folder `Home Page Labels`.

```dax
M_Units Label =
VAR _u = [M_Units Sold]
RETURN
    IF ( NOT ISBLANK ( _u ), FORMAT ( _u, "#,##0" ) & " units" )
```

```dax
M_Discount Rate Label =
VAR _r = DIVIDE ( [M_Discounts], [M_Gross Sales] )
RETURN
    IF ( NOT ISBLANK ( _r ), FORMAT ( _r, "0.0%" ) & " of gross" )
```

```dax
M_COGS Ratio Label =
VAR _r = DIVIDE ( [M_Cost Of Goods], [M_Net Sales] )
RETURN
    IF ( NOT ISBLANK ( _r ), FORMAT ( _r, "0.0%" ) & " of net sales" )
```

```dax
M_Margin Label =
VAR _r = DIVIDE ( [M_Profit], [M_Net Sales] )
RETURN
    IF ( NOT ISBLANK ( _r ), FORMAT ( _r, "0.0%" ) & " margin" )
```

```dax
M_Coverage Label =
VAR _s = [M_# of Distinct Segments]
VAR _c = [M_# of Distinct Countries]
RETURN
    IF (
        NOT ISBLANK ( _s ),
        "Across " & _s & " segments and " & _c & " countries"
    )
```

**Spelling matters:** the existing measure is `M_Cost Of Goods` — capital O in "Of".

**Why `DIVIDE` and the `ISBLANK` guard.** `DIVIDE` returns blank rather than erroring on
a zero denominator. Without the guard, `FORMAT ( BLANK(), "0.0%" )` returns an empty
string and the card would read a bare ` of gross` with no number in front of it.

### Expected values, unfiltered **[CALCULATED]**

| Measure | Expected |
|---|---|
| `M_Units Label` | `1,125,788 units` |
| `M_Discount Rate Label` | `7.2% of gross` |
| `M_COGS Ratio Label` | `85.8% of net sales` |
| `M_Margin Label` | `14.2% margin` |
| `M_Coverage Label` | `Across 5 segments and 5 countries` |

Test all five on a blank page before touching the Home page.

---

## Margin and discount measures (Phase 2 work, not yet built)

For the margin and discount diagnostic view. `M_Discount Cap` is a what-if parameter.

```dax
M_Discount Recovered =
VAR _Cap = [M_Discount Cap]
RETURN
    SUMX ( FACT_SALES, MAX ( 0, FACT_SALES[Discounts] - _Cap * FACT_SALES[Gross Sales] ) )
```

```dax
M_Realised Discount Rate at Cap =
VAR _Cap = [M_Discount Cap]
RETURN
    DIVIDE (
        SUMX ( FACT_SALES, MIN ( FACT_SALES[Discounts], _Cap * FACT_SALES[Gross Sales] ) ),
        [M_Gross Sales]
    )
```

Both cap at **row level** inside `SUMX`, not on the aggregate. Capping an aggregate
would let a heavily discounted row hide behind a lightly discounted one.

**Enterprise breaks even at a 5.29% cap.** **[CALCULATED]** by solving for the cap at
which segment profit crosses zero. Note that a cap is a ceiling, not a target.
