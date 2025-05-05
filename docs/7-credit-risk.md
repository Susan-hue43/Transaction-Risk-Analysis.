## 5. Credit Risk: 

**Identify customers at risk of default based on debt-to-income ratios and credit score trends.**

```sql
WITH Risk_Categorization AS (
    SELECT 
        id AS user_id, 
        total_debt, 
        yearly_income, 
        credit_score,
        ROUND((total_debt / NULLIF(yearly_income, 0)) * 100, 2) AS debt_to_income_ratio,
        CASE 
            WHEN (total_debt / NULLIF(yearly_income, 0)) * 100 > 100 AND credit_score < 500 THEN 'Very High Risk'
            WHEN (total_debt / NULLIF(yearly_income, 0)) * 100 > 60 AND credit_score BETWEEN 500 AND 599 THEN 'High Risk'
            WHEN (total_debt / NULLIF(yearly_income, 0)) * 100 BETWEEN 40 AND 60 AND credit_score BETWEEN 600 AND 699 THEN 'Moderate Risk'
            ELSE 'Low Risk'
        END AS risk_category
    FROM users_data
)
SELECT * 
FROM Risk_Categorization
WHERE risk_category IN ('High Risk', 'Very High Risk')
ORDER BY risk_category DESC, debt_to_income_ratio DESC;
```

<img width="763" alt="credit scores" src="https://github.com/user-attachments/assets/68592c97-ef29-4406-a5f3-38fe95a205a5" />


#### Key Findings and Implications:

#### **Critical Risk Profiles**

* **Debt-to-Income Ratio >200%.**

  * *Example*: User 1801 has a **DTI of 221.41%** and **credit score of 480**.
  * **Implication**: These users are deep in the red. They are likely relying entirely on borrowed money, suggesting high probability of default without immediate corrective action.


* **Credit Score Collapse.**

  * *Observation*: **100% of “Very High Risk” users have credit scores <500**, well below the average of 704.
  * **Implication**: These customers have either defaulted in the past or are structurally uncreditworthy. Conventional recovery or repayment strategies are unlikely to be effective.
    

#### **Hidden Risk Segments.**

* **Moderate Debt, Low-Income Profiles.**

  * *Example*: User 1330 with **\$30K debt on \$35K income**.
  * **Implication**: These users don’t appear high-risk at first glance, but limited income leaves them **one unexpected expense away from default**. Traditional risk metrics may underestimate their vulnerability.


---
