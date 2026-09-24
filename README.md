# Excel Analytics on the Olist Marketplace

## Purpose

This project builds real Excel fluency through tiered, deliberate practice against a real
commercial dataset. It has two goals and neither one is optional. First, broad coverage of the
Excel function library, the kind tested on a timed screening assessment. Second, the analytical
judgment to build a workbook a commercial analyst would actually hand to a category manager and
a finance lead.

The dataset was picked because it's hard. Nine related tables span three separate grains, real
money moves through it, and one column looks like a stable customer ID but isn't, which quietly
breaks any retention analysis built on it. Every finding in this project was checked against the
actual data, not assumed from a schema diagram.

The work moves from manual formula-based joins through Power Query, the Data Model and DAX,
financial modeling, and VBA automation.

---

## A note on scope

This is a portfolio project built for a specific purpose, passing Excel screening assessments
and defending a real analytical workbook in interviews. A few things it deliberately is not.

- **Not a forecast.** The data window runs 2016 to 2018 and has partial boundary months, so it
  doesn't support a defensible forecast. This project doesn't attempt one.
- **Not VBA application development.** The automation layer (Tiers 12 and 13) covers procedural
  automation, refresh orchestration, validation, export. Class modules, custom userforms, and
  packaged add-ins are out of scope.
- **Not a statement about the Brazilian e-commerce market.** The data is Brazilian. The analysis
  isn't market expertise.

---

## Status

Tiers 0 and 1 are closed. Tier 2, lookups, conditional aggregation, and the manual grain
audit, is next.

Last updated 24 September 2026.

---

## Dataset

This project uses the Brazilian E-Commerce Public Dataset by Olist, available at
[kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).
It's real, anonymized commercial transaction data from a Brazilian marketplace, covering roughly
100,000 orders placed between September 2016 and October 2018, across nine related files. A
secondary dataset, the
[Marketing Funnel by Olist](https://www.kaggle.com/datasets/olistbr/marketing-funnel-olist),
adds two more files and gets used as an optional extension in Tiers 9 and 10.

Row counts and primary keys below were checked directly against the source files, not assumed.

| Table | Rows | Primary key | Role |
|---|---|---|---|
| olist_orders_dataset | 99,441 | order_id | Core |
| olist_order_items_dataset | 112,650 | order_id + order_item_id | Core |
| olist_order_payments_dataset | 103,886 | order_id + payment_sequential | Core |
| olist_order_reviews_dataset | 99,224 | review_id + order_id | Core |
| olist_customers_dataset | 99,441 | customer_id | Dimension |
| olist_products_dataset | 32,951 | product_id | Dimension |
| olist_sellers_dataset | 3,095 | seller_id | Dimension |
| product_category_name_translation | 71 | product_category_name | Dimension |
| olist_geolocation_dataset | 1,000,163 | none, needs aggregation | Reference |
| olist_closed_deals_dataset | 842 | mql_id | Funnel (optional) |
| olist_marketing_qualified_leads_dataset | 8,000 | mql_id | Funnel (optional) |

Full column-by-column detail, data types, and key roles live in `DATA_DICTIONARY.xlsx`.

---

## Tech stack

- Excel (Tables, structured references, dynamic arrays, PivotTables)
- Power Query and M for extract, transform, and load
- Excel Data Model (Power Pivot) and DAX for the star schema and measures
- VBA for refresh orchestration, validation, and export automation
- Git and GitHub for version control and portfolio publication

---

## Reproducing this environment

1. Download the Brazilian E-Commerce Public Dataset from Kaggle (link above). A Kaggle account
   is required, and the download is a single zip containing all nine CSVs.
2. Extract the CSVs into a local `/data` folder. This folder isn't committed to the repo since
   the files are publicly available from Kaggle and aren't original work.
3. Open the primary workbook (`.xlsx`, no macros required). Power Query's source queries point
   at a folder-path parameter, so set it to your local `/data` folder on first open.
4. Refresh all queries and check the row counts against the Dataset table above.
5. The automation build (`.xlsm`) is optional and only needed from Tier 13 on. It requires
   macros to be enabled.

---

## Skill progression, fifteen tiers

| Tier | Focus |
|---|---|
| 0 | Scoping and data inventory, covering schema orientation, primary and foreign keys, and table relationships |
| 1 | Structured tables and formula foundations, covering Excel Tables, named ranges, and structured references |
| 2 | Lookups, conditional aggregation, and a manual grain audit, walking into the fan-out and customer-identity traps by hand |
| 3 | Logical, text, date, and statistical functions, covering the IF family, text parsing, date arithmetic, and descriptive statistics |
| 4 | Dynamic arrays, LET, and LAMBDA, covering FILTER, UNIQUE, SORT, and reusable calculations |
| 5 | PivotTables, covering grouping, slicers, calculated fields, and the wall a flat table eventually hits |
| 6 | Power Query fundamentals, covering Get and Transform, the Applied Steps pane, and scaling from a working subset to the full dataset |
| 7 | Power Query joins and the grain audit done properly, covering merges, anti-joins, and reconciling against Tier 2 |
| 8 | M language literacy, covering the Advanced Editor, custom columns, parameters, and custom functions |
| 9 | The Data Model and DAX, covering the star schema, relationships, measures, and time intelligence |
| 10 | Financial and consumer analysis, covering assumptions, contribution margin, cohort retention, and Goal Seek |
| 11 | Dashboard and validation, covering conditional formatting, precedent tracing, and a full pipeline refresh proof |
| 12 | VBA foundations and the Excel object model, covering loops, error handling, and debugging |
| 13 | Automation build, covering refresh orchestration, a validation routine, and PDF export |
| 14 | Publication and portfolio packaging, finalizing both builds and the documentation |

---

## Findings so far

Checking the source files directly, rather than assuming, turned up three data issues that would
quietly produce wrong answers if handled naively.

1. **The fan-out problem.** `order_items` and `order_payments` both carry multiple rows per
   order. A naive join on `order_id` alone, without the full composite key, multiplies rows and
   inflates any summed dollar figure. Confirmed structurally in Tier 0. Gets quantified with an
   actual reconciliation in Tiers 2 and 7.

2. **customer_id isn't a person.** `customers.customer_id` gets generated fresh for every order.
   `customer_unique_id` is the stable identifier for an actual person. Any retention analysis
   built on the former returns a near-zero repeat-purchase rate that looks plausible and is
   wrong. Confirmed structurally in Tier 0. Gets proven with real numbers in Tier 2.

3. **Duplicate reviews.** 547 of 99,224 review rows share an order_id with another review row.
   Of those, 345 orders have identical scores across their duplicates, which looks like a
   resurvey artifact, and 202 have genuinely different scores, which looks like a changed
   customer experience over time, like a delayed delivery. The resolution rule adopted is to
   keep the most recent review per order, by `review_answer_timestamp`. The tradeoff, documented
   rather than hidden, is that this can understate an early-stage complaint that got resolved by
   the time of the final review.

The remaining two traps, partial boundary months and status/date column selection for revenue
recognition, aren't investigated yet. They surface in Tier 2.

---

## Known limitations and caveats

- **Data window.** The dataset ends in October 2018. Nothing here claims to reflect the current
  e-commerce market.
- **Currency.** All monetary values are in Brazilian reais, not USD.
- **Boundary months.** The series opens and closes with partial, low-volume months. Trend
  analysis trims these out. The exact rule lands in `METHODOLOGY.md` once Tier 2 closes.

---

## Future work

- **Geolocation mapping.** The geolocation table needs aggregation to zip-prefix grain before
  it's joinable. Mapping visuals are an optional Tier 9 extension, not scheduled yet.
- **Marketing funnel join.** The closed_deals and marketing_qualified_leads pair adds a
  cross-source funnel analysis. Optional Tier 10 extension, not scheduled yet.
- **VBA beyond automation.** Class modules, custom userforms, and packaged add-ins got
  deliberately scoped out (see A note on scope). Worth revisiting only if a specific role
  requires it.

---

## Documentation

- `DATA_DICTIONARY.xlsx` is the table and column inventory, built after Tier 0.
- `METHODOLOGY.md` will hold the reasoning behind each judgment call, the duplicate-review
  resolution, the revenue recognition date, the boundary-month trimming. Gets created once
  Tier 2 produces the first real entries.
- `DECISIONS.md` is a running log of scope changes and major calls, with the date and reason.
- `NOTES.md` holds the tier-by-tier closing-question answers, in Christopher's own words.
