## 1. Spending Patterns

```sql
SELECT  m.mcc_id, 
        m.Description, 
        COUNT(t.id) AS Total_Transactions,
        ROUND(SUM(t.amount), 2) AS Total_Spent
FROM mcc_codes AS m
JOIN transactions_data AS t
ON m.mcc_id = t.mcc
GROUP BY m.mcc_id, m.Description
ORDER BY Total_Transactions DESC;
```

<img width="765" alt="spending patterns" src="https://github.com/user-attachments/assets/78ee9cea-2353-42e0-8c1e-87bd0b856efa" />


#### **Key Findings & Business Implications:**

* **Grocery Stores and Supermarkets (MCC 5411)** recorded the highest transaction count at **18,371**, with total spending of **\$501,980.56**. This reflects high-frequency, low-to-moderate value purchases typical of everyday consumer behavior. The consistent volume underscores this category’s importance for customer engagement and operational reliability.


* **Money Transfer services (MCC 4829)** had a significantly lower transaction count of **7,257** but the highest overall spend at **\$596,560**, with an average of around **\$82 per transaction**. This indicates a high-value, low-frequency pattern often associated with financial services, pointing to elevated fraud exposure and potentially different user segments with larger transactional stakes.


* **Wholesale Clubs (MCC 5300)** also showed strong performance, with **7,573**  transactions and **\$471,078.75** in spending. The relatively high per-transaction average (about **\$62**) suggests bulk purchasing behavior, possibly by small businesses or value-conscious consumers—relevant for segmentation and pricing insights.


* **Dining-related categories** showed varied behavior:

  * **Eating Places and Restaurants (MCC 5812):** **11,673** transactions, with total spending of **\$316,035.80**
  * **Fast Food (MCC 5814):** **5,888** transactions, **\$158,989.44** in spending.
  * **Drinking Places (Alcoholic Beverages) (MCC 5813):** **2,838** transactions, and **\$67,352.98** spend.
    
  This spread reflects different levels of discretionary spending, with lower engagement in alcohol-related venues possibly due to cultural, regulatory, or regional factors.


* **Essential service categories** such as **Service Stations (MCC 5541)** with **17,055** transactions and **\$380,788.89** in spendng, **Tolls and Bridge Fees (MCC 4784)** with **8,242** transactions and **\$289,230.09** spend, show sustained high usage. Their predictability signals reliability in recurring customer behavior and the operational importance of seamless transaction processing.


* Other key utility categories - including **Drug Stores and Pharmacies (MCC 5912)**, **Utilities (MCC 4900)**, and **Telecommunication Services (MCC 4814)** demonstrate both **high transaction volume** and **significant total spend($430,486.73, $357,798.43, $313491.88, respectively)**, highlighting their role in essential daily life and their potential stability as part of the bank's transaction ecosystem.


### 1.1. Geographical Spending Patterns 

```sql
SELECT TOP 10
    merchant_state, 
    merchant_city,
    COUNT(id) AS Total_Transactions, 
    ROUND(COALESCE(SUM(amount), 0), 2) AS Total_Spent 
FROM transactions_data 
GROUP BY merchant_state, merchant_city
ORDER BY TOTAL_TRANSACTIONS DESC;
```

<img width="763" alt="transaction by location" src="https://github.com/user-attachments/assets/c106a2d2-4eeb-4428-8da9-a45ab0880626" />


#### **Key Findings & Business Implications:**

* **Online transactions** accounted for **17,921 transactions** and **\$1,048,210.76** in spend, making it the dominant channel. This reflects high customer reliance on digital platforms and signals that a significant share of business activity occurs outside physical locations, which has implications for risk exposure and digital engagement strategies.


* **Houston, TX** had the highest volume among physical locations with **2,268 transactions** and **\$123,275.36** in total spend. This suggests a concentration of customer activity in a major metropolitan area, relevant for geographic risk profiling and demand planning.


* **North Hollywood, CA** had **2,232 transactions** but only **\$60,191.31** in spend, indicating a high volume of low-value transactions. This may point to routine or necessity-based spending behavior and a customer segment with lower average spend per visit.


* **Orlando, FL** and **Yorba Linda, CA** recorded fewer transactions (**1,611 and 1,523**, respectively) but high total spending (**\$84,927.89 and \$83,119.60**, respectively), which could reflect high-value consumer segments or merchants offering large ticket items.


* **Atlanta, GA**, **San Antonio, TX** and **Nevada**  showed balanced activity with **(1,321, 1399, 1122 transactions)** and **(\$84,536.03, $65834.85, $71067.07, respectively)** in spending, suggesting both consistent customer engagement and above-average transaction value, making it a commercially significant location.


* **Bronx, NY** and **Sacramento, CA** had relatively lower volumes (**\~1,116, 1097 transactions**) and lower spending (**\$31,241.09 and \$48,984.91**, respectively), which may indicate markets with less penetration or lower purchasing power, potentially relevant in assessing customer base diversity and regional performance variability.

---
