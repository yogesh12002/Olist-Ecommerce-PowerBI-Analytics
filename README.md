# Olist E-Commerce Analytics Platform — Power BI Solution

An end-to-end Business Intelligence and Data Engineering solution built on the **Olist Brazilian E-Commerce dataset**. This project transforms raw transactional CSV extracts into a normalized **Multi-Fact Star Schema** in Power Query and delivers a four-page interactive Power BI dashboard analyzing revenue growth, customer retention, fulfillment logistics, and payment/review behavior.

---

## Key Highlights & Business Insights

* **Gross Revenue & Order Scale:** **$15.42M** in total gross revenue across **96K** distinct orders and **110K** line items sold, with an Average Order Value (AOV) of **$159.83**.
* **Customer Retention:** **96K** unique buyers with a **2.91%** repeat customer rate (**3K** repeat buyers) across all **27** Brazilian states, led by São Paulo (`SP`), Rio de Janeiro (`RJ`), and Minas Gerais (`MG`).
* **Logistics & SLA Performance:** **91.89%** on-time delivery rate with an average transit time of **12.41 days** and freight accounting for **14.26%** of gross revenue.
* **Payments & Satisfaction:** **78.34%** of payment volume (`$16.01M` total) processed via credit card (averaging **2.85** installments), achieving an overall **4.09 / 5.00** customer review score (**57.78%** 5-star reviews).

---

## Data Warehouse & Star Schema Architecture

Raw CSV files are isolated in a disabled-load staging layer in Power Query (M) and materialized into **3 Fact Tables** and **6 Conformed Dimension / Bridge Tables** using strict single-direction `*:1` relationships:

![Data Model Star Schema](images/data_model.png)

### Fact Tables
| Table Name | Defined Grain | Key Metrics & Attributes |
| :--- | :--- | :--- |
| **`fact_order_items`** | 1 row per line item sold | `price`, `freight_value`, `order_status`, purchase & delivery timestamps |
| **`fact_payments`** | 1 row per payment installment/method | `payment_value`, `payment_type`, `payment_installments`, `payment_sequential` |
| **`fact_reviews`** | 1 row per customer satisfaction survey | `review_score` (1–5), survey creation & response timestamps |

### Dimension & Bridge Tables
* **`dim_orders` (Bridge Table):** Conformed order dimension (`order_id`) connecting `fact_order_items`, `fact_payments`, and `fact_reviews` to prevent many-to-many fan-out anomalies.
* **`dim_date`:** Official Date Table spanning `2016-09-01` to `2018-10-31`, linked to a pure date column (`order_purchase_date`) to eliminate midnight timestamp mismatches.
* **`dim_customers`:** Customer master with surrogate `customer_key` and `customer_unique_id` for accurate repeat-buyer cohort tracking.
* **`dim_products`:** Product catalog enriched with English category translations (`product_category_name_english`).
* **`dim_sellers`:** Merchant directory keyed via surrogate `sellers_key`.
* **`dim_geolocation`:** Deduplicated postal code reference table (`geolocation_zip_code_prefix`) providing normalized latitude/longitude coordinates.

---

## Semantic Layer & DAX Highlights

All KPIs are organized inside a dedicated `AllMeasures` table. Cross-fact filtering between `fact_order_items`, `fact_payments`, and `fact_reviews` is handled cleanly using `TREATAS` to preserve single-direction relationships:

```dax
// Core Revenue & AOV
Total Gross Revenue = SUM ( fact_order_items[price] ) + SUM ( fact_order_items[freight_value] )
Average Order Value (AOV) = DIVIDE ( [Total Gross Revenue], [Total Orders], 0 )

// Time Intelligence
MoM Revenue Growth % = 
VAR CurrentMonthRev = [Total Gross Revenue]
VAR PrevMonthRev = CALCULATE ( [Total Gross Revenue], DATEADD ( dim_date[Date], -1, MONTH ) )
RETURN
IF ( NOT ISBLANK ( PrevMonthRev ) && PrevMonthRev <> 0, DIVIDE ( CurrentMonthRev - PrevMonthRev, PrevMonthRev ) )

// Cross-Fact Lineage via TREATAS
Filtered Payment Value = 
CALCULATE (
    SUM ( fact_payments[payment_value] ),
    TREATAS ( VALUES ( fact_order_items[order_id] ), fact_payments[order_id] )
)
```

---

## Dashboard Walkthrough

All four report pages share a unified header featuring global slicers (`Year`, `Quarter`, `Order Status`), a one-click **Clear all slicers** button, and directional navigation controls.

### 1. Executive Overview
Tracks top-line revenue, order volume, AOV, items sold, Month-over-Month revenue growth, Top 10 Product Categories, monthly sales trends, order status distribution, and state-level revenue rankings.

![Executive Overview](images/page1_executive_overview.png)

### 2. Customer & Geography Insights
Analyzes unique vs. repeat customer acquisition, repeat customer rate (`2.91%`), revenue per customer (`$160.43`), geographic revenue distribution, and top-performing cities.

![Customer & Geography Insights](images/page2_customer_geography.png)

### 3. Delivery & Logistics Performance
Monitors average transit days (`12.41`), on-time delivery SLA (`91.89%`), late order volume (`8K`), freight cost ratio (`14.26%`), state-level delivery delays, and seller fulfillment performance.

![Delivery & Logistics Performance](images/page3_delivery_logistics.png)

### 4. Payments & Customer Reviews
Evaluates total payment value (`$16.01M`), payment method split (Credit Card vs. Boleto/Voucher/Debit), installment distributions, review score breakdown, and the correlation between delivery delays and customer star ratings.

![Payments & Customer Reviews](images/page4_payments_reviews.png)

---

## Repository Structure & Setup

```text
├── data/                        # Raw Olist CSV source files (or link to Kaggle dataset)
├── docs/                        # Technical Architecture & BI Documentation (PDF)
├── images/                      # Dashboard & Star Schema screenshots referenced in README
├── Olist_Ecommerce_BI.pbix      # Main Power BI Desktop report & data model
└── README.md                    # Project documentation
```

### How to Run Locally
1. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/olist-ecommerce-powerbi.git
   ```
2. Save your screenshots inside an `images/` folder matching the filenames above (`data_model.png`, `page1_executive_overview.png`, etc.).
3. Open `Olist_Ecommerce_BI.pbix` in **Microsoft Power BI Desktop**.
4. If refreshing raw data locally, update the source folder path parameter in **Transform data → Data source settings** and click **Refresh**.
