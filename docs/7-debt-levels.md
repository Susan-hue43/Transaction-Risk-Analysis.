## 6. Debt Levels: 

**Analyze the distribution of total debt across customers for portfolio risk management.**

```sql
WITH Debt_Distribution AS (
    SELECT 
        id AS user_id, 
        total_debt,
        yearly_income,
        ROUND((total_debt / NULLIF(yearly_income, 0)) * 100, 2) AS debt_to_income_ratio,
        CASE 
            WHEN total_debt = 0 THEN 'No Debt'
            WHEN total_debt BETWEEN 1 AND 10000 THEN 'Low Debt (1-10K)'
            WHEN total_debt BETWEEN 10001 AND 50000 THEN 'Moderate Debt (10K-50K)'
            WHEN total_debt BETWEEN 50001 AND 100000 THEN 'High Debt (50K-100K)'
            ELSE 'Very High Debt (100K+)'
        END AS debt_category
    FROM users_data
)
SELECT 
    debt_category, 
    COUNT(user_id) AS total_users,
    ROUND(AVG(total_debt), 2) AS avg_debt,
    ROUND(MIN(total_debt), 2) AS min_debt,
    ROUND(MAX(total_debt), 2) AS max_debt
FROM Debt_Distribution
GROUP BY debt_category
ORDER BY total_users DESC;
```

<img width="763" alt="debt levels" src="https://github.com/user-attachments/assets/b11bb2fb-3645-4330-877d-cf4d6bfc334e" />


#### Key Findings and Implications:

* **Dominance of High Debt (771 users):** The large number of users in the `'High Debt (50K-100K)'` category implies a significant portion of the portfolio is susceptible to economic downturns or localized financial strains affecting this segment.

  
* **Substantial Moderate Debt Exposure (546 users):** The considerable number of users with `'Moderate Debt (10K-50K)'` suggests a heightened sensitivity to interest rate fluctuations and changes in individual financial stability.

  
* **Very High Debt (381 users):** The substantial average debt **(143K)** in the `'Very High Debt (100K+)'` category indicates a significant vulnerability to large financial losses should even a small number of these users default within the local economic context.

  
* **Low Debt Segment (200 users, avg.3K):** The small size of the `'Low Debt (1-10K)'` segment implies a limited immediate risk but also a potentially untapped market for increased credit product penetration.

  
* **Minimal No Debt (102 users):** The very small `'No Debt'` segment suggests a high existing penetration of credit products within the current customer base in the region.
 
  
* **Average Debt Trend (Increasing with Category):** The escalating average debt across categories implies a progressively higher financial risk associated with each increase in debt level within the local lending environment.


---
