# E-commerce Sales Analysis with SQL (Olist, Brazil)

**Tools:** SQL (joins, CTEs, window functions) in SQLite, Python (pandas, matplotlib)

**Links:**
- Kaggle notebook: `https://www.kaggle.com/code/shainapereira/e-commerce-sales-analysis/edit`


## Questions
1. How fast is revenue growing, and is the growth continuing?
2. Which products, categories and regions drive sales?
3. Do customers come back?
4. How good is delivery, and does it affect customer reviews?
5. How dependent is the business on a few sellers?

## Data
Public "Brazilian E-Commerce Public Dataset by Olist": 8 linked tables covering 99,441 orders, 112,650 order items, 96,096 customers, 3,095 sellers, 32,951 products, payments and reviews, from September 2016 to October 2018.

**Definitions used everywhere:** revenue = item price excluding freight, delivered orders only ($13,221,498). Growth is measured on the 20 full months from January 2017 to August 2018. A customer is `customer_unique_id`. A delivery is late if it arrived on a later calendar day than the estimate.

## Method
1. **Cleaning (pandas)** - no rows removed except duplicate reviews (99,224 -> 98,673, one per order). Added English names for 2 missing product categories, labelled 610 missing categories "unknown", converted dates. Full before/after table: `CHANGES_AND_DECISIONS.md`.
2. **SQL (SQLite)** - 15 queries across 8 tables using multi-table joins, CTEs, `LAG`, `RANK`, `ROW_NUMBER`, `NTILE` and running totals.
3. **Charts** - revenue trend, categories, late deliveries by month, reviews by delivery result, seller concentration, time to a second order.

## Key findings
1. **Revenue grew fast, then flattened.** January to August 2018 revenue was $7.22M, up 141% from $2.99M in the same months of 2017 (orders up 140%). Monthly revenue peaked in November 2017 ($987,765, +52.4% on October), but through 2018 it stayed within $826K to $978K a month, and August 2018 ($838,577) was 9% below January 2018.
2. **Average order value is stable at about $137** ($137.04 overall, between $124 and $149 in every month).
3. **Sales are spread across categories but concentrated by region.** Health and beauty is the top category (9.3% of revenue), followed by watches and gifts (8.8%) and bed, bath and table (7.7%); the top 10 categories hold 62.4%. Sao Paulo state alone buys 38.3% of revenue, and the top three states (SP, RJ, MG) buy 63.4%.
4. **Very few customers come back.** Of 93,358 customers with a delivered order, 97.0% ordered once. The 2,801 repeat customers (3.0%) bring only 5.5% of revenue and spend a little less per order ($123 vs $138). And 30.5% of second orders were placed within 24 hours of the first, which looks like one shopping trip split into two orders, so genuine returns after at least a day are about 2.1% of customers.
5. **Delivery got worse at peak times.** 6.77% of orders were late overall, but the rate reached 12.4% in November 2017, 14.1% in February 2018 and 19.0% in March 2018, against between 1% and 8% in every other full month. Average delivery time fell from 16.6 days (February and March 2018) to 8.6 days (June to August 2018).
6. **Late delivery wrecks reviews.** The average review score is 4.29 for on-time orders (6.6% one-star), 3.29 for orders 1 to 3 days late, 2.10 for 4 to 7 days late and 1.70 for more than 7 days late (69.8% one-star).
7. **Remote states wait longest.** Alagoas has the highest late rate (21.4%, 24.5 days average delivery), followed by Maranhao (17.4%) and Sergipe (15.2%). Sao Paulo is at 4.5% and 8.8 days. Rio de Janeiro is a big market with a high late rate (12.1% on 12,350 orders).
8. **A small group of sellers carries the platform.** The top 10% of sellers (297 of 2,970 with sales) earn 67.1% of revenue, the top 20% earn 82.3%, and the bottom half earns about 3%. The #1 seller earns $226,988 (1.7%) so no single seller dominates.
9. **Payments:** credit cards make up 78.5% of payment value (3.5 installments on average), bank slips (boleto) 18.0%.

These are patterns in one public dataset. The data cannot say why delivery slowed in early 2018 or why customers do not return.

## Recommendations
- **Protect delivery capacity before peaks** (November and the first quarter) and track late rate weekly. Late deliveries are the clearest driver of bad reviews.
- **Fix the worst regions first:** set realistic delivery estimates, or add logistics partners, for Alagoas, Maranhao, Sergipe and Rio de Janeiro.
- **Create a reason to return.** With 97% one-time buyers, test follow-up offers 30 to 90 days after the first order, and measure repeat rate on orders at least a day apart so same-day split orders do not inflate it.
- **Look at growth drivers again.** Revenue plateaued in 2018 even though early 2018 was strong, so test new categories and regions beyond Sao Paulo.
- **Manage seller risk.** Hold regular reviews with the top 297 sellers and set delivery targets for sellers with high late rates.

## Limitations
- Public data from 2016 to 2018; patterns may have changed.
- Revenue excludes freight and counts delivered orders only ($370K of item value from cancelled and unfinished orders was left out).
- "Late" is measured against the estimate Olist gave the customer, not against what the customer expected.
- Reviews exist for 98,673 of 99,441 orders and customers choose whether to review, so scores may lean towards very happy or very unhappy customers.
- No cost or margin data, so profit cannot be analysed.
- Two English category names are my own translation (listed in `CHANGES_AND_DECISIONS.md`).

## Repository contents
- `Olist_Ecommerce_Sales_SQL.ipynb` - full notebook (cleaning, 15 SQL queries, charts, findings)
- `code/01_clean_olist.py`, `code/02_queries.sql`, `code/02_run_sql.py` - the same code as separate files
- `clean/` - the 8 cleaned tables
- `CHANGES_AND_DECISIONS.md` - every change to the data and why

