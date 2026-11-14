# What I changed in the data, and why

Raw files: 8 CSVs from "Brazilian E-Commerce Public Dataset by Olist" (the geolocation file was not used).
Cleaned tables: `clean/` folder. Code: `code/01_clean_olist.py`.

## Row counts: before and after
| Table | Raw rows | Clean rows | Note |
|---|---|---|---|
| orders | 99,441 | 99,441 | unchanged |
| order_items | 112,650 | 112,650 | unchanged |
| payments | 103,886 | 103,886 | unchanged |
| reviews | 99,224 | **98,673** | reduced to one review per order |
| customers | 99,441 | 99,441 | unchanged |
| products | 32,951 | 32,951 | unchanged |
| sellers | 3,095 | 3,095 | unchanged |
| translation | 71 | **74** | 3 rows added |

## Checks that found nothing to fix
| Check | Result |
|---|---|
| Duplicate rows in any table | 0 |
| Items pointing to a missing order, product or seller | 0 |
| Orders pointing to a missing customer | 0 |
| Delivered orders with no items | 0 |
| Deliveries recorded before the purchase date | 0 |

## Changes made
| # | Change | Before | After | Why |
|---|---|---|---|---|
| 1 | Dates converted to real dates | text | datetime (5 order dates, shipping limit, 2 review dates) | Needed for date maths and month grouping |
| 2 | Added `purchase_month` and `is_delivered` to orders | none | "2017-11" style month; 1/0 flag | Lets every query group by month and filter to completed sales |
| 3 | Reviews cut to one per order (latest kept) | 99,224 rows | 98,673 rows | 547 orders had more than one review and 814 review IDs repeated. Joining the raw table would count those orders twice and inflate review and seller figures |
| 4 | Review comment text dropped, `has_comment` flag added | text columns, mostly empty | 1/0 flag | Not used in the analysis; the text columns were 88% and 59% empty |
| 5 | Product columns renamed | `product_name_lenght`, `product_description_lenght` | `..._length` | Original spelling mistake |
| 6 | Missing product category set to "unknown" | 610 blank | "unknown" | Keeps those products in category totals instead of silently dropping them |
| 7 | Added English names for 2 missing categories | `pc_gamer`, `portateis_cozinha_e_preparadores_de_alimentos` had no translation | `pc_gaming`, `portable_kitchen_and_food_preparers` | The translation file only covered 71 of 73 categories. These 2 English names are my own translation |
| 8 | Added an "unknown" row to the translation table | none | `unknown` | So the join from change 6 works |

## Left alone on purpose (and how it is handled)
- **8 delivered orders have no delivery date.** They are kept in revenue but left out of delivery-time and late-delivery figures.
- **6 orders that are not marked delivered have a delivery date.** Ignored, because their status says they were not delivered.
- **14 delivered orders have no approval time.** Not used anywhere, so not changed.
- **9 payments of $0 and 3 payments of type "not_defined"** are left as they are. Revenue comes from item prices, not payments, so they do not affect revenue.
- **No outliers were removed.** Prices from $0.85 to $6,735 and delivery times up to 209 days are real data points.

## Definitions and decisions
| Decision | Reason |
|---|---|
| **Revenue = item price, excluding freight, for delivered orders only** | $13,221,498. Freight is shipping cost, not product sales. Counting only delivered orders avoids counting cancelled or unfinished sales: $370,146 of item value was excluded (cancelled, unavailable, shipped, invoiced, processing, approved) |
| **Revenue is never taken from the payments table** | An order can have several payment rows (2,961 do) and several items (9,803 do). Joining payments to items would multiply money. Payments total $16.0M against $15.8M of item price plus freight, so they differ for reasons like vouchers and installment fees |
| **Growth uses full months only: 2017-01 to 2018-08** | The data has 4 orders in 2016-09, 324 in 2016-10, 1 in 2016-12, none in 2016-11, and 16 plus 4 in 2018-09 and 2018-10. Including them would show fake growth and fake collapses |
| **A customer is `customer_unique_id`** | `customer_id` is new for every order, so counting it makes it look like nobody ever reorders |
| **Late = delivered on a later calendar day than the estimate** | The estimate is a date with no time of day. Comparing raw timestamps would mark an order delivered on the promised afternoon as late. This gives 6.77% late, against 8.11% the other way |
| **Repeat purchases are measured on delivered orders** | A cancelled order is not a real purchase |

## The 4 files built for the dashboard (`code/03_build_dashboard_files.py`)
| File | One row per | Used for |
|---|---|---|
| `olist_items_flat.csv` | order item (112,650) | revenue, categories, states, sellers |
| `olist_orders_flat.csv` | order (99,441) | delivery, reviews, repeat customers |
| `olist_monthly_summary.csv` | month (23) | revenue trend and month-over-month growth |
| `olist_sellers_summary.csv` | seller with sales (2,970) | seller ranking and concentration |

The script checks that total revenue is $13,221,498.11 in all four files and stops with an error if they ever disagree.
