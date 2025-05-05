## 2. Geographical Trends
**Identify high-spending regions or locations prone to failed transactions.**

``` sql
SELECT 
    merchant_state, 
    merchant_city, 
    COUNT(id) AS Failed_transactions,
    ROUND(SUM(amount), 2) AS Total_Spent 
FROM transactions_data 
WHERE errors NOT IN ('No Error Occurred')
GROUP BY merchant_state, merchant_city
HAVING SUM(amount) > 1000
ORDER BY Total_Spent DESC;
```

<img width="757" alt="high spending regions" src="https://github.com/user-attachments/assets/5073c4a2-e63c-46e7-9769-04d9df3f9cf8" />


**Key Findings & Business Implications:**

* **Online transactions** had the highest number of failed attempts, with **445 failed transactions** totaling **\$26,353.81**. This represents a **high-risk** area for customer friction in digital channels.


* **Orlando, FL** led among physical locations with **67 failed transactions** and **\$6,274.85** in spend. This suggests a combination of **strong customer activity** and **notable operational issues**, which may affect satisfaction and trust.


* **Newton Grove, NC** and **San Antonio, TX** had **27 and 39 failures**, with a spend of **\$2,620.00** and **\$2,496.90**, respectively. Despite lower transaction volumes, these figures highlight that failed transactions can carry meaningful financial impact even in smaller markets.


* **Atlanta, GA** and **Yorba Linda, CA** reported **\$2,242.77** and **\$2,137.79** in spend from **(26, 23, respectively)** failed transactions. These locations also appear in the list of high-value transaction zones, reinforcing their dual importance in value and risk monitoring.


* Locations like **Rome, Italy** and **Oakland, CA** showed **low failure counts** (**10** and **13**, respectively) but **high average** transaction values **($1409.94, $1710.31)**, reflecting that transaction risk isn’t limited to high volume areas and can occur in isolated but high stakes instances.


* Multiple **California cities**; **Oakland, Sacramento, North Hollywood, Carmel Valley** registered failed transaction attempts between **\$1,100 and \$1,711**, indicating a broad pattern of minor disruptions across a key regional market.


* **Houston, TX** and **Bronx, NY**, both high in overall transaction volume, also experienced **25** failed transactions each, with of **\$1,278.88** and **\$1,262.97**, suggesting that high traffic areas naturally carry greater exposure to failed attempts.

---
