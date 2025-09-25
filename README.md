# 📈 Retail Sales & Inventory Analytics using SQL Window Functions

This project focuses on leveraging **Advanced SQL** and **Window Functions** to solve critical business challenges for a national **FMCG retail company** in Rwanda. By analyzing daily sales data, the goal is to transform raw information into actionable insights that prevent risks like stockouts, overstocking, and ineffective promotions.

The analysis framework is designed to deliver three primary expected outcomes:

1.  Highlight **top performing products** by quarter and region.
2.  Compute **sales growth and momentum** to detect deteriorating performance.
3.  Set customers by **revenue quartiles** to guide targeted promotions.
   
So here’s the source : Griva, A., Bardaki, C., Pramatari, K., & Papakyriakopoulos, D. (2018). Retail Business Analytics: Customer Visit Segmentation Using Market Basket Data. Expert Systems with Applications, 100, 1–15. ResearchGate.
   
### Names

  * **NGOGA Desire Arnaud** (ID: 27627)

-----

## 🗄️ Database Structure and Schema

The project utilizes a simple, normalized schema based on three tables: `customers`, `products`, and `transactions`.

### 1\. Table Creation (DDL)

<img width="430" height="141" alt="customer table" src="https://github.com/user-attachments/assets/4aa28b62-0a7d-466d-b827-05f3753a5d2d" />
<img width="458" height="153" alt="products table" src="https://github.com/user-attachments/assets/b7f9ff52-8957-417b-934c-b92c6a98a85d" />
<img width="750" height="186" alt="transcations table" src="https://github.com/user-attachments/assets/61ad90a4-045d-4f9d-ad27-0dc56b5b814f" />

### 2\. Entity-Relationship Model

The `transactions` table links customers and products, representing a many-to-many relationship via Foreign Keys.
<img width="1115" height="328" alt="Relation customer" src="https://github.com/user-attachments/assets/35bba329-c0ad-4fb4-b072-7a4bd487fc69" />
<img width="1111" height="348" alt="relation products" src="https://github.com/user-attachments/assets/6d98b090-6f32-476f-af68-570d3228e5d4" />
<img width="1099" height="284" alt="Relation Transcation" src="https://github.com/user-attachments/assets/6e339b90-f611-422a-b2ea-61da2303028e" />


### 3\. Data Insertion (DML)

Sample data was inserted to populate the tables and test the analytical queries.

<img width="491" height="573" alt="inserting data" src="https://github.com/user-attachments/assets/c5c9399b-b983-48f0-ade1-a477c998abd1" />
<img width="487" height="392" alt="inserting products data" src="https://github.com/user-attachments/assets/f9eef184-aa32-48ca-b3b9-9a5e0307bcf6" />
<img width="790" height="396" alt="inserting transcations data" src="https://github.com/user-attachments/assets/46f3c473-9c2a-4c17-ab0c-8398b6ebb181" />

<img width="591" height="300" alt="transcations data" src="https://github.com/user-attachments/assets/279b2903-e6ef-4843-af3f-830a953103c3" />
<img width="393" height="256" alt="customer data" src="https://github.com/user-attachments/assets/ad729ad6-7e91-4047-84c2-bad7c49390eb" />
<img width="332" height="300" alt="products data" src="https://github.com/user-attachments/assets/72e402ba-4d11-4b7e-9ae0-58f934a65619" />


-----

## 📊 Analytical SQL Queries

The core of this project is the use of powerful SQL **Window Functions** to derive business metrics.

### 1\. Top Product by Region (`ROW_NUMBER()`)

Identifies the highest-selling product in terms of total amount for each geographical region.

```sql
SELECT
    sales_data.region,
    sales_data.product_name,
    sales_data.total_amount
FROM
    (
        SELECT
            c.region,
            p.name AS product_name,
            SUM(t.amount) AS total_amount,
            ROW_NUMBER() OVER (PARTITION BY c.region ORDER BY SUM(t.amount) DESC) AS rn
        FROM
            transactions t
        INNER JOIN customers c ON t.customer_id = c.customer_id
        LEFT JOIN products p ON t.product_id = p.product_id
        GROUP BY
            c.region, p.name
    ) sales_data
WHERE
    sales_data.rn = 1 -- Selects the top product (rn=1) per region
ORDER BY
    sales_data.region;
```

**Output:**

| REGION | PRODUCT\_NAME | TOTAL\_AMOUNT |
| :--- | :--- | :--- |
| Gakenke | Milk | 55000 |
| HUYE | salt | 75000 |
| Kigali | Coffee Beans | 25000 |
| MUSANZE | Ipad | 65000 |

-----

### 2\. Sales Momentum: Month-over-Month Growth (`LAG()`)

Calculates the Year-over-Year (YoY) sales growth percentage using the `LAG()` function to look back 12 months for comparison.

```sql
WITH monthly_sales AS (
    SELECT
        TO_CHAR(sale_date, 'YYYY-MM') as current_month,
        SUM(amount) as current_sales
    FROM transactions
    GROUP BY TO_CHAR(sale_date, 'YYYY-MM')
)
SELECT
    current_month,
    current_sales,
    LAG(current_sales, 12) OVER (ORDER BY current_month) as previous_year_sales,
    ROUND(((current_sales - LAG(current_sales, 12) OVER (ORDER BY current_month)) /
    LAG(current_sales, 12) OVER (ORDER BY current_month)) * 100, 2) as yoy_growth_percentage
FROM monthly_sales
ORDER BY current_month;
```

**Output (Sample):**

| CURRENT\_MONTH | CURRENT\_SALES | PREVIOUS\_YEAR\_SALES | YOY\_GROWTH\_PERCENTAGE |
| :--- | :--- | :--- | :--- |
| 2024-01 | 25000 | (null) | (null) |
| 2024-02 | 55000 | (null) | (null) |
| 2024-03 | 65000 | (null) | (null) |
| 2024-04 | 75000 | (null) | (null) |

-----

### 3\. Sales Momentum: Difference from Annual Average (`AVG() OVER()`)

Compares monthly sales performance against the average monthly sales for the entire year, partitioned by `sale_year`.

```sql
WITH monthly_data AS (
    SELECT
        TO_CHAR(sale_date, 'YYYY-MM') as month,
        TO_CHAR(sale_date, 'YYYY') as sale_year,
        SUM(amount) as monthly_sales
    FROM transactions
    GROUP BY TO_CHAR(sale_date, 'YYYY-MM'), TO_CHAR(sale_date, 'YYYY')
)
SELECT
    month,
    monthly_sales,
    AVG(monthly_sales) OVER (PARTITION BY sale_year) AS annual_average_sales,
    monthly_sales - AVG(monthly_sales) OVER (PARTITION BY sale_year) AS difference_from_annual_avg
FROM monthly_data
ORDER BY month;
```

**Output (Sample for 2024):**

| MONTH | MONTHLY\_SALES | ANNUAL\_AVERAGE\_SALES | DIFFERENCE\_FROM\_ANNUAL\_AVG |
| :--- | :--- | :--- | :--- |
| 2024-01 | 25000 | 55000 | -30000 |
| 2024-02 | 55000 | 55000 | 0 |
| 2024-03 | 65000 | 55000 | 10000 |
| 2024-04 | 75000 | 55000 | 20000 |

-----

### 4\. Customer Segmentation (`RANK()`)

Ranks customers by their total spending to identify high-value clients for targeted marketing campaigns.

```sql
WITH ranked_customers AS (
    SELECT
        c.customer_id,
        c.name,
        c.region,
        SUM(t.amount) AS total_spent,
        RANK() OVER (ORDER BY SUM(t.amount) DESC) AS spending_rank
    FROM
        customers c
    JOIN
        transactions t ON c.customer_id = t.customer_id
    GROUP BY
        c.customer_id, c.name, c.region
)
SELECT
    customer_id,
    name,
    region,
    total_spent,
    spending_rank
FROM
    ranked_customers
WHERE
    spending_rank <= 10 -- Targeting the top 10 for promotions
ORDER BY
    total_spent DESC;
```

**Output (Sample - Top 4):**

| CUSTOMER\_ID | NAME | REGION | TOTAL\_SPENT | SPENDING\_RANK |
| :--- | :--- | :--- | :--- | :--- |
| 1004 | IRAKOZE Patrick | HUYE | 75000 | 1 |
| 1003 | NSHUTI Arsene | MUSANZE | 65000 | 2 |
| 1002 | MUGABO Ian | Gakenke | 55000 | 3 |
| 1001 | MUGISHA Salvi | Kigali | 25000 | 4 |

-----

### 5\. Running Sales Totals (`SUM() OVER()`)

Calculates a running average of sales per quarter, providing insight into sales momentum over the reporting period.

```sql
SELECT
    TO_CHAR(sale_date, 'YYYY-Q') AS quarter,
    SUM(amount) AS quarterly_sales,
    -- Running average of the quarterly sales
    AVG(SUM(amount)) OVER (ORDER BY TO_CHAR(sale_date, 'YYYY-Q')) AS moving_avg_sales
FROM
    transactions
GROUP BY
    TO_CHAR(sale_date, 'YYYY-Q')
ORDER BY
    quarter;
```

**Output (Sample):**

| QUARTER | QUARTERLY\_SALES | MOVING\_AVG\_SALES |
| :--- | :--- | :--- |
| 2024-1 | 145000 | 145000 |
| 2024-2 | 75000 | 110000 |

-----

## 🛠️ Conclusion

This project successfully transformed raw sales data into business-critical intelligence with the use of Advanced SQL Window Functions. The written queries provide important metrics like top-selling products by region and important sales momentum metrics like YoY growth. Customer segmentation by rank, on the other hand, enables promotional campaigns to be directed towards high-value customers. Ultimately, these analytical applications allow the FMCG retail company to make data-driven decisions to enhance inventory and sales optimization. The relational database structure that lies beneath supports future expansion and complex analysis.
