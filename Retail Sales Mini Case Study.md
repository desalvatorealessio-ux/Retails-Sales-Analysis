# 🛍️ Retail Sales Mini Case Study

**Dataset:** [Retail Sales Dataset](https://www.kaggle.com/datasets/mohammadtalib786/retail-sales-dataset) (Kaggle) — 1,000 transactions, Jan 2023 – Jan 2024
**Tools:** SQLite (DB Browser for SQLite), Power BI
**Scope:** Descriptive analysis only — what happened, and how the data breaks down. No forecasting or predictive modeling; these findings are meant to support a recommendation, not make one.

## 📌 Solution

### 1. How does customer age and gender influence their purchasing behavior?

**By age group:**

```sql
SELECT age_group,
       SUM(total_amount) AS revenue,
       COUNT(*)          AS transactions,
       ROUND(AVG(total_amount), 2) AS avg_order_value
FROM retail_sales_clean
GROUP BY age_group
ORDER BY age_group;
```

**Answer:**

<img width="400" alt="Image1_Age Group" src="https://github.com/user-attachments/assets/626f1c11-d3eb-4883-8938-aded20df96e9" />

**By gender and category:**

```sql
SELECT gender, product_category,
       SUM(total_amount) AS revenue,
       COUNT(*)          AS transactions
FROM retail_sales_clean
GROUP BY gender, product_category
ORDER BY gender, revenue DESC;
```

<img width="400" alt="Image2_GenderxCategory" src="https://github.com/user-attachments/assets/dd604c93-25d3-4622-9ff5-9595953f5afa" />


Age influences total spend more clearly than gender does: revenue climbs from 18-25 up through a peak at 46-55 ($100,690) before dropping at 56+, and the 18-25 group has the highest average order value ($500.30) despite fewer transactions. Gender makes almost no difference — Female ($456.55) and Male ($455.43) average order values are within $2 of each other — and category preference splits close to evenly by gender, with Female slightly favoring Clothing and Male slightly favoring Electronics.

---

### 2. Are there discernible patterns in sales across different time periods?

**By month:**

```sql
SELECT sale_year, sale_month, sale_month_name,
       SUM(total_amount) AS monthly_revenue,
       COUNT(*)          AS monthly_transactions
FROM retail_sales_clean
GROUP BY sale_year, sale_month
ORDER BY sale_year, sale_month;
```

**Answer:**

<img width="400" alt="Image4_Salesbymonth" src="https://github.com/user-attachments/assets/da2f5e33-4bf8-4d42-bfd0-640b842bdcd8" />


**By day of week:**

```sql
SELECT sale_weekday,
       SUM(total_amount)           AS revenue,
       COUNT(*)                    AS transactions,
       ROUND(AVG(total_amount),2)  AS avg_order_value
FROM retail_sales_clean
GROUP BY sale_weekday
ORDER BY revenue DESC;
```

<img width="400" alt="Image4_Revbyweekday" src="https://github.com/user-attachments/assets/dc1d7d32-3cbb-41aa-be5a-5c8adc66a29b" />


Monthly revenue bounces rather than trends — May is the strongest full month ($53,150) and September the weakest ($23,620), with no steady ramp up or down across the year (2024-01 shows only $1,530 because the data cuts off one day into the month, not a real drop). By day of week, Saturday is the strongest ($78,815 revenue, $525.43 average order value) and Thursday the weakest on both measures — sales are consistently higher on weekends than midweek.

---

### 3. Which product categories hold the highest appeal among customers?

```sql
SELECT product_category,
       SUM(total_amount) AS category_revenue,
       SUM(quantity)     AS units_sold,
       COUNT(*)          AS transactions,
       ROUND(AVG(total_amount), 2) AS avg_order_value,
       ROUND(100.0 * SUM(total_amount) / (SELECT SUM(total_amount) FROM retail_sales_clean), 1) AS pct_of_total_revenue
FROM retail_sales_clean
GROUP BY product_category
ORDER BY category_revenue DESC;
```

**Answer:**

<img width="400" alt="Image6_CategoryPerformace" src="https://github.com/user-attachments/assets/44590718-d1e2-46d7-984a-6f3f76b9e16a" />


The three categories are close to evenly split — Electronics leads with 34.4% of revenue ($156,905), Clothing follows closely at 34.1%, and Beauty at 31.5% — so no single category dominates customer demand.

---

### 4. What are the relationships between age, spending, and product preferences?

```sql
SELECT age_group, product_category,
       SUM(total_amount) AS revenue
FROM retail_sales_clean
GROUP BY age_group, product_category
ORDER BY age_group, revenue DESC;
```

**Answer:**

<img width="400" alt="Image3_AgexCategory" src="https://github.com/user-attachments/assets/f68d168a-33e2-4007-a03c-5f59e90b4074" />


The top category shifts with age — 18-25 favors Beauty, 26-35 favors Clothing, and every group from 36 upward favors Electronics — while total spend rises from 18-25 up through a peak at 46-55 before declining at 56+. Age relates to both how much customers spend and what they buy; the two don't move independently.

---

### 5. How do customers adapt their shopping habits during seasonal trends?

**By season:**

```sql
SELECT
    CASE
        WHEN sale_month IN (12, 1, 2) THEN 'Winter'
        WHEN sale_month IN (3, 4, 5)  THEN 'Spring'
        WHEN sale_month IN (6, 7, 8)  THEN 'Summer'
        WHEN sale_month IN (9, 10, 11) THEN 'Fall'
    END AS season,
    SUM(total_amount)          AS revenue,
    COUNT(*)                   AS transactions,
    ROUND(AVG(total_amount),2) AS avg_order_value
FROM retail_sales_clean
GROUP BY season
ORDER BY revenue DESC;
```

**Answer:**

<img width="400" alt="Image7_SeasonRevenue" src="https://github.com/user-attachments/assets/2ca3126f-9ee2-428e-8485-adfae2c39646" />


**By season and category:**

```sql
SELECT
    CASE
        WHEN sale_month IN (12, 1, 2) THEN 'Winter'
        WHEN sale_month IN (3, 4, 5)  THEN 'Spring'
        WHEN sale_month IN (6, 7, 8)  THEN 'Summer'
        WHEN sale_month IN (9, 10, 11) THEN 'Fall'
    END AS season,
    product_category,
    SUM(total_amount) AS revenue,
    COUNT(*)           AS transactions
FROM retail_sales_clean
GROUP BY season, product_category
ORDER BY season, revenue DESC;
```

<img width="400" alt="Image8_SeasonxCategory" src="https://github.com/user-attachments/assets/73458f77-e322-4ac1-801d-2ef2b728f0fa" />


Winter is the strongest season for both revenue ($125,730) and average order value ($495.00), and the category mix shifts with it: Electronics leads in Winter and Summer, Clothing leads in Spring, and Beauty never leads a single season outright. Customers' category preferences genuinely change across the year, not just how much they spend.

---

### 6. Are there distinct purchasing behaviors based on the number of items bought per transaction?

**Quantity vs. order value and price:**

```sql
SELECT quantity,
       COUNT(*)                        AS transactions,
       SUM(total_amount)               AS revenue,
       ROUND(AVG(total_amount), 2)     AS avg_order_value,
       ROUND(AVG(price_per_unit), 2)   AS avg_price_per_unit
FROM retail_sales_clean
GROUP BY quantity
ORDER BY quantity;
```

**Answer:**

<img width="400" alt="Image9_QuantityxPrice" src="https://github.com/user-attachments/assets/eddbcf71-6d6b-4bd1-be9b-df0c287c5a30" />


**Average basket size by category:**

```sql
SELECT product_category,
       ROUND(AVG(quantity), 2) AS avg_quantity,
       COUNT(*)                 AS transactions
FROM retail_sales_clean
GROUP BY product_category;
```

<img width="400" alt="Image10_QuantitybyCategory" src="https://github.com/user-attachments/assets/1d003d88-5740-42a8-ad48-8cec86cdd2e1" />


Basket size doesn't correlate with item price — average price per unit stays roughly flat ($166–200) regardless of whether 1 or 4 items are purchased — and average basket size is nearly identical across all three categories (2.48–2.55 items). Purchase quantity behaves independently of both price tier and category.

---

### 7. What insights can be gleaned from the distribution of product prices within each category?

```sql
SELECT product_category,
    CASE
        WHEN price_per_unit <= 50 THEN 'Budget ($25-50)'
        WHEN price_per_unit = 300 THEN 'Mid ($300)'
        ELSE 'Premium ($500)'
    END AS price_tier,
    COUNT(*)           AS transactions,
    SUM(total_amount)  AS revenue,
    ROUND(100.0 * SUM(total_amount) / SUM(SUM(total_amount)) OVER (PARTITION BY product_category), 1) AS pct_of_category_revenue
FROM retail_sales_clean
GROUP BY product_category, price_tier
ORDER BY product_category,
    CASE price_tier WHEN 'Budget ($25-50)' THEN 1 WHEN 'Mid ($300)' THEN 2 ELSE 3 END;
```

**Answer:**

<img width="400" alt="Image11_Pricetier" src="https://github.com/user-attachments/assets/cd8acec8-9955-471f-be5d-f43721b9b1d5" />


Price per unit only takes three effective tiers in this data ($25–50 Budget, $300 Mid, $500 Premium — not a continuous spread). Despite Premium being a minority of transactions in every category (18–22%), it generates 51–59% of that category's revenue. A small share of higher-priced transactions disproportionately drives revenue, consistently across all three categories.

---

## 📝 Notes on the data

- **No repeat customers:** every `Customer ID` appears exactly once across all 1,000 transactions, so repeat-purchase rate, customer lifetime value, and retention analysis aren't possible with this file.
- **Price isn't continuous:** `Price per Unit` only takes 5 exact values (25, 30, 50, 300, 500) — this shaped the Budget/Mid/Premium tiers used in Question 7 instead of arbitrary price bands.
