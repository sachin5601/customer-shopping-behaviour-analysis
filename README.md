
# 🛒 Customer Shopping Behavior Analysis

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![pandas](https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white)

> End-to-end data analytics project analyzing 3,900 retail transactions to uncover customer spending patterns, segment behavior, and actionable business insights — using Python, MySQL, and Power BI.

---

## 📌 Project Overview

A complete data analytics pipeline built from scratch:

**Raw CSV → Python (EDA & Cleaning) → MySQL → SQL Business Analysis → Power BI Dashboard → Business Recommendations**

| Attribute | Value |
|-----------|-------|
| Dataset Size | 3,900 rows |
| Features | 18 columns |
| Tools Used | Python, pandas, MySQL, Power BI |
| Output | Interactive dashboard + 5 business recommendations |

---

## 🔧 Tech Stack

| Layer | Tools |
|-------|-------|
| Data Cleaning & EDA | Python, pandas, NumPy, Matplotlib, Seaborn |
| Feature Engineering | pandas (age_group, purchase_frequency_days) |
| Database | MySQL |
| SQL Analysis | MySQL (CASE, window functions, GROUP BY, subqueries) |
| Visualization | Power BI (interactive dashboard with slicers) |

---

## 📊 Key Numbers & Findings

| Insight | Result |
|---------|--------|
| 💰 Revenue — Male vs Female | $157,890 vs $75,191 |
| 👥 Loyal Customers | 3,116 out of 3,900 (80%) |
| 📦 Top Revenue Age Group | Young Adults — $62,143 |
| 🏷️ Subscription Rate | Only 27% subscribed |
| 🔁 Repeat Buyers Not Subscribed | 72.7% — key upsell gap |
| 🛍️ Most Discount-Dependent Product | Hat — 50% of purchases |
| ⭐ Highest Rated Product | Gloves (avg: 3.86 / 5) |
| 💵 Total Revenue Modeled | ~$233,081 |

---

## 🧹 Data Cleaning (Python + pandas)

```python
# Key steps performed
import pandas as pd

df = pd.read_csv('customer_shopping_data.csv')

# 1. Initial exploration
df.info()
df.describe()

# 2. Handle 37 missing values in Review Rating
df['review_rating'] = df.groupby('category')['review_rating'].transform(
    lambda x: x.fillna(x.median())
)

# 3. Rename columns to snake_case
df.columns = df.columns.str.lower().str.replace(' ', '_')

# 4. Feature Engineering
df['age_group'] = pd.cut(df['age'], bins=[0,25,40,55,100],
                          labels=['Young Adult','Adult','Middle-aged','Senior'])

# 5. Drop redundant column
df.drop(columns=['promo_code_used'], inplace=True)

# 6. Load to MySQL
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://user:password@localhost/shopping_db')
df.to_sql('customer_data', con=engine, if_exists='replace', index=False)
```

---

## 🗄️ SQL Business Questions (MySQL)

```sql
-- 1. Revenue by Gender
SELECT gender, SUM(purchase_amount) AS revenue
FROM customer_data GROUP BY gender;
-- Result: Male $157,890 | Female $75,191

-- 2. High-Spending Discount Users (839 customers)
SELECT customer_id, purchase_amount FROM customer_data
WHERE discount_applied = 'Yes'
AND purchase_amount > (SELECT AVG(purchase_amount) FROM customer_data);

-- 3. Top 5 Products by Avg Rating
SELECT item_purchased, ROUND(AVG(review_rating), 2) AS avg_rating
FROM customer_data GROUP BY item_purchased
ORDER BY avg_rating DESC LIMIT 5;

-- 4. Shipping Type Comparison
SELECT shipping_type, ROUND(AVG(purchase_amount), 2) AS avg_purchase
FROM customer_data GROUP BY shipping_type;
-- Standard: $58.46 | Express: $60.48

-- 5. Subscribers vs Non-Subscribers
SELECT subscription_status,
       COUNT(*) AS total_customers,
       ROUND(AVG(purchase_amount), 2) AS avg_spend,
       SUM(purchase_amount) AS total_revenue
FROM customer_data GROUP BY subscription_status;

-- 6. Discount-Dependent Products (Top 5)
SELECT item_purchased,
       ROUND(SUM(CASE WHEN discount_applied='Yes' THEN 1 ELSE 0 END)*100.0/COUNT(*), 2) AS discount_rate
FROM customer_data GROUP BY item_purchased
ORDER BY discount_rate DESC LIMIT 5;

-- 7. Customer Segmentation
SELECT customer_id,
  CASE
    WHEN previous_purchases >= 10 THEN 'Loyal'
    WHEN previous_purchases BETWEEN 3 AND 9 THEN 'Returning'
    ELSE 'New'
  END AS customer_segment
FROM customer_data;
-- Loyal: 3116 | Returning: 701 | New: 83

-- 8. Top 3 Products per Category (Window Function)
SELECT * FROM (
  SELECT category, item_purchased,
         COUNT(*) AS total_orders,
         RANK() OVER (PARTITION BY category ORDER BY COUNT(*) DESC) AS item_rank
  FROM customer_data GROUP BY category, item_purchased
) ranked WHERE item_rank <= 3;

-- 9. Repeat Buyers & Subscription Correlation
SELECT subscription_status, COUNT(*) AS repeat_buyers
FROM customer_data WHERE previous_purchases > 5
GROUP BY subscription_status;

-- 10. Revenue by Age Group
SELECT age_group, SUM(purchase_amount) AS total_revenue
FROM customer_data GROUP BY age_group ORDER BY total_revenue DESC;
-- Young Adult: $62,143 | Middle-aged: $59,197 | Adult: $55,978 | Senior: $55,763
```

---

## 📈 Power BI Dashboard

**6 Visuals + 4 Slicers (Subscription Status, Gender, Category, Shipping Type)**

KPIs displayed:
- 🧑‍🤝‍🧑 Total Customers: **3.9K**
- 💳 Avg Purchase Amount: **$59.76**
- ⭐ Avg Review Rating: **3.75**
- Revenue by Category, Age Group, and Subscription Status

> 📸 *(Add screenshot of your Power BI dashboard here)*

---

## 💡 Business Recommendations

1. **Boost Subscriptions** — 73% non-subscribers is a massive upsell opportunity; launch exclusive subscriber benefits campaign
2. **Loyalty Program** — 3,116 loyal customers need retention incentives to prevent churn
3. **Discount Policy Review** — Hat, Sneakers, Coat over-reliant on discounts (47–50%); review margin impact
4. **Target Young Adults** — Top revenue segment at $62K; prioritize in seasonal marketing campaigns
5. **Product Spotlight** — Promote top-rated products (Gloves, Sandals) in email and social campaigns

---

## 📁 Repository Structure

```
customer-shopping-behaviour-analysis/
├── data/
│   └── customer_shopping_data.csv
├── notebooks/
│   └── EDA_Cleaning.ipynb
├── sql/
│   └── business_queries.sql
├── dashboard/
│   └── Customer_Behaviour_Dashboard.pbix
└── README.md
```

---

## 👤 Author

**Sachin Shinde** — B.E. AI & Data Science, ACPCE Mumbai

[![GitHub](https://img.shields.io/badge/GitHub-Sachin5601-181717?style=flat&logo=github)](https://github.com/Sachin5601)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-sachin5601-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/sachin5601)
[![Email](https://img.shields.io/badge/Email-eng.sachinshinde@gmail.com-D14836?style=flat&logo=gmail&logoColor=white)](mailto:eng.sachinshinde@gmail.com)