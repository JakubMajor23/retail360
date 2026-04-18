# 🏪 Retail360 — CRM & Customer Intelligence Analytics

[![Wersja Polska](https://img.shields.io/badge/Wersja-Polska-red?style=for-the-badge)](README_PL.md)

Retail360 is a comprehensive end-to-end analytical project that transforms raw e-commerce transactional data into a production-ready, 3-page operational dashboard. The project demonstrates the full data lifecycle: from advanced cleaning and extraction (Python/Pandas) to building a dedicated Star Schema (Data Engineering) and professional visualization focused on driving specific business decisions (Power BI).

---

## 📊 Business Case & Project Rationale

### 🎯 Who is this for?
The primary users are **Heads of CRM**, **Customer Strategy Managers**, and **Account Managers**. This report is 100% **Customer-Centric**, prioritizing relationship health and long-term value over simple logistics or financial reporting.

### ❓ The Problem: "Analytical Myopia"
Many e-commerce businesses operate reactively, leading to:
*   **"Spray and Pray" Marketing:** Generic campaigns sent to the entire database (e.g., 5,000+ customers) without personalization.
*   **Budget Leakage:** Discounts being sent to "Champions" who would have purchased anyway.
*   **Silent Churn:** Ignoring "At Risk" customers until they have already transitioned to "Lost."
*   **Poor Resource Allocation:** Marketing teams guessing when to communicate and what products to offer.

### 💡 The Solution & Business Value
This dashboard shifts the organization from **reactive** to **proactive**. Instead of just reporting what happened, it tells managers **what to do next**. It enables precise RFM segmentation, automated Churn Risk scoring, and actionable recommendations that directly impact **Customer Lifetime Value (CLV)** and marketing ROI.

---

## 🖥️ Dashboard Architecture: From Insights to Action

The dashboard follows a logical flow: **STATUS → ALARM → ACTION**.

### 1. Health Check (Customer Base Vitality)
**Key Question:** *"What is the state of our customer base RIGHT NOW?"*
**Objective:** A 30-second situational overview to identify where the money is and find reasons for concern.

![Health Check Dashboard](photos/d1.png)

*   **Visuals:** KPI tiles showing active customer ratio and "Top 20% Revenue Share". Bar charts comparing volume vs. revenue structure.
*   **Business Decisions:** Assessing financial security and monitoring macro trends (growing vs. stagnating base).

### 2. Churn Risk (Revenue Salvation)
**Key Question:** *"Who are we losing RIGHT NOW and what is the cost?"*
**Objective:** An operational view to calculate the value of "money at risk" and trigger rescue actions.

![Churn Risk Dashboard](photos/d2.png)

*   **Visuals:** "Total CLV at Risk" tiles, "Risk Tier Distribution" (Healthy, Watchlist, Critical), and a drill-through table of specific customers.
*   **Business Decisions:** Prioritizing Account Manager workflows for "Win-back" campaigns.

### 3. Behavior & Patterns (Campaign Optimization)
**Key Question:** *"How should we target campaigns to maximize conversion?"*
**Objective:** Provide hard data for the Marketing Department.

![Behavior & Patterns Dashboard](photos/d3.png)

*   **Visuals:** AOV and Return Rate analysis by segment, purchase heatmap (days/hours), and "Top Products" list.
*   **Business Decisions:** Scheduling targeted newsletters (e.g., Sunday 6:00 PM) based on segment behavior.

---

## ⚙️ ETL & Data Engineering

The data preparation process (found in `ETL.ipynb`) transforms raw transaction logs (UCI Online Retail II dataset, 2009-2011) into a clean, analytical Star Schema.

### 🧹 Cleaning & Extraction:
*   **Guest Handling:** Assigned `customer_id = 0` to unregistered users instead of deleting them (~23% of total sales).
*   **Noise Removal:** Filtered out operational non-sales entries (POSTAGE, test logs).
*   **Financial Standardization:** Flagged returns and unified product metadata.

### 🧪 Advanced Feature Engineering:
*   **RFM Segmentation:** Programmatic assignment to groups: *Champions, Loyal, Recent Buyers, Promising, At Risk, Lost*.
*   **Risk Score (0-100):** A custom scoring algorithm calculating churn probability based on segment, recency, and frequency.
*   **Automated Recommendations:** Automatically assigns recommended actions (e.g., *Upsell, Win-back, Personal Outreach*).

---

## 🗄️ Data Model (Star Schema)

High-performance schema optimized for the **VertiPaq** engine in Power BI. 

```mermaid
erDiagram
    dim_date {
        int date_key PK
        date full_date
        int year
        int quarter
        int month
        string month_name
        int day
        int day_of_week
        string day_name
        string year_month
        string year_quarter
        boolean is_month_end
    }

    dim_segment {
        int segment_key PK
        string segment_name
        string segment_group
        int sort_order
        int lifecycle_rank
    }

    dim_customer {
        int customer_key PK
        int customer_id
        boolean is_registered
        date first_purchase_date
        date last_purchase_date
        string acquisition_month
        int total_transactions
        int total_items
        float total_revenue
        float avg_order_value
        int days_since_last_purchase
        boolean active_90d_flag
        string value_tier
        int rfm_r_score
        int rfm_f_score
        int segment_key FK
        float clv_proxy
        int risk_score
        string risk_tier
        string recommended_action
    }

    dim_product {
        int product_key PK
        string stock_code
        string product_name
        float unit_price_median
    }

    fct_orders {
        int order_key PK
        string invoice
        int customer_key FK
        int date_key FK
        int hour
        int total_quantity
        int total_items
        float total_revenue
        float gross_revenue
        int num_lines
        int num_unique_products
        boolean is_return
    }

    fct_order_lines {
        int order_line_key PK
        int order_key FK
        int customer_key FK
        int product_key FK
        int date_key FK
        int quantity
        int quantity_abs
        float price
        float line_total
        float line_total_abs
        boolean is_return
    }

    fct_customer_migration {
        int migration_key PK
        int customer_key FK
        int from_date_key FK
        int to_date_key FK
        int segment_from_key FK
        int segment_to_key FK
        string migration_direction
        float revenue_from
        float revenue_to
    }

    dim_segment ||--o{ dim_customer : current_segment
    dim_customer ||--o{ fct_orders : places
    dim_date ||--o{ fct_orders : order_date
    fct_orders ||--|{ fct_order_lines : contains
    dim_customer ||--o{ fct_order_lines : buys
    dim_product ||--o{ fct_order_lines : product
    dim_date ||--o{ fct_order_lines : line_date
    dim_customer ||--o{ fct_customer_migration : customer
    dim_date ||--o{ fct_customer_migration : month_from_to
    dim_segment ||--o{ fct_customer_migration : segment_from_to
```

---

## 🛠️ Technology Stack

*   **Python 3.13:** Logic & ETL.
*   **Pandas & NumPy:** Data transformations.
*   **Jupyter Notebook:** Pipeline development.
*   **Power BI:** Visualization & DAX.
*   **Mermaid:** Architecture documentation.

---

## 🚀 Getting Started

1.  **Download Source Data:** Place `online_retail_II.xlsx` in the `data/raw/` folder.
2.  **Run ETL:** Execute the `ETL.ipynb` notebook to generate CSV tables in `star_schema/`.
3.  **Open Dashboard:** Load the Power BI (`.pbix`) file.
4.  **Refresh:** Point the data sources to your local CSV files and refresh.
