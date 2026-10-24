# Project Overview

---

## Business problem

A sales organisation selling six products across five segments and five countries needs
to answer four questions from one report:

1. What is the current state of the business?
2. Where is performance strong and where is it weak?
3. What is driving the difference?
4. What can management actually do about it?

The original report answered the first two. It described outcomes — totals by segment,
by country, by product — without explaining them. This project moves it from
**descriptive** to **diagnostic**, and is honest about where the data stops.

---

## Audience

| Audience | What they need | Where they get it |
|---|---|---|
| Sales leadership | State of the business in one glance; where to intervene | Home |
| Segment and country managers | Their own performance in context | Sales Analysis, bookmark views |
| Finance | Margin, discount, cost structure | Net Profit, Cost of Goods |
| Operations | Volume patterns | Units Sold |
| A reviewer or hiring manager | Evidence that the analysis is sound | Documentation and the Scope panel |

The report is designed for the first audience first. An executive opening it sees the
income statement, not a cover page.

---

## Scope

### In scope

- Sales performance: volume, revenue, discount, cost, profit.
- Breakdown by period, segment, country and product.
- Diagnosis of profitability through the margin-versus-discount relationship.
- Scenario analysis on a discount cap.
- Honest statement of what the data cannot support.

### Out of scope, with reasons

| Excluded | Why |
|---|---|
| Forecasting | Tested and rejected. Flat mean MASE 0.83 beats seasonal naïve 1.25; 8 of 12 calendar months appear once |
| Product profitability | Cost per unit tracks sale price, not product. Only 164 of 700 rows reconcile with `Manufacturing Price` |
| Customer analysis | No customer identifier in the source |
| Seasonality | The monthly pattern correlates with record count at r = 0.871 — a sampling artefact |
| Concentration analysis | Herfindahl 1,835 against 1,667 for an even split. No finding |
| Price-volume-mix decomposition | Requires a defensible unit cost, which this data does not provide |

Each exclusion is a decision taken on evidence and documented in
[`KNOWN_LIMITATIONS.md`](KNOWN_LIMITATIONS.md). None is an omission.

---

## Analytical approach

**Phase 1 — Correct.** Fix what was wrong before adding anything: year-on-year computed
on a like-for-like window, unsupported product profitability removed, measure
definitions made explicit, validation established.

**Phase 2 — Diagnose.** The margin and discount view. The finding that drives it: for
every segment, profit is positive exactly when gross margin percentage exceeds the
realised discount rate. Enterprise, the only loss-making segment, breaks even at a
5.29% discount cap.

**Phase 3 — Govern.** Metric register on the Help page, alt text, documented defects.

**Phase 4 — Publish.** Repository documentation, screenshots, release.

The discipline across all four: **make the analysis more valuable, not merely bigger.**
A seven-page report is not better than a six-page report. A complex forecast is not
better than a well-validated baseline. Every addition has to justify itself against the
alternative of leaving it out.

---

## The Home page redesign

The original landing page covered **25.8% of its canvas** and carried a single
contiguous empty block of 1005 × 354 px — **38.6% of the page**. The background image
had been doing the design work; with it removed, the page had a hole where its substance
should be.

Six defects, in order of cost:

1. The dead block.
2. Figures buried inside prose sentences.
3. Five related numbers presented as unrelated panels.
4. Four identical `Learn More` buttons, visually heavier than the figures above them.
5. Text overflowing its containers.
6. `Net Proft` — a typo on the landing page of a portfolio piece.

**The redesign's organising idea: the landing page is the income statement.** Gross
sales less discounts is net sales; net sales less cost of goods is profit. That chain is
arithmetically true, four of its five stations already have a drill page behind them,
and laying the page out along it means the structure carries meaning rather than
decoration.

Below the chain: a volume panel, and a panel stating the data's limits. Putting the
limitations where an executive sees them first is deliberate.

---

## What this project demonstrates

- Defining a business problem before choosing a method.
- Assessing data feasibility before committing to an analysis.
- Building and validating business metrics against source.
- Diagnosing drivers rather than reporting outcomes.
- **Rejecting an impressive-sounding technique on evidence.**
- Designing an executive report in Power BI.
- Communicating limitations without undermining the work.

---

## Data provenance

The dataset is a variant of Microsoft's publicly distributed **Financial Sample**
workbook, extended with a 25-row manager dimension. It is sample data. No finding in
this report describes a real business outcome.

This is stated on the landing page, in the README, and in `KNOWN_LIMITATIONS.md`.
