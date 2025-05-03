# Transaction-and-Risk-Analysis.

## Introduction:
This report presents an analysis of Aurora bank transaction data aimed at understanding customer spending behaviors, enhancing fraud detection capabilities, and ensuring compliance with relevant regulations. By examining various facets of transaction activity, this analysis seeks to identify patterns and trends that can inform business strategies for risk mitigation and customer engagement. The insights derived from this analysis are intended to provide a foundation for more informed decision-making regarding fraud prevention, operational efficiency, and customer relationship management.

## Objective:
The primary objective of this report is to analyze transaction data to identify key patterns and anomalies related to customer spending, geographical trends, transaction errors, high-value transactions, credit risk indicators, and debt levels. This analysis will provide a comprehensive overview of potential risks and opportunities within our transaction ecosystem.

### Focus Areas:
This report will focus on the following key areas of transaction and risk analysis:
- **Spending Patterns:** Evaluating transaction volumes and total spending across different merchant categories (MCC) and locations to understand where and how customers are spending their money.
- **Geographical Trends:** Identifying regions or specific locations with high spending activity or a higher propensity for failed transactions.
- **Error Trends:** Monitoring the occurrence and types of transaction errors, such as incorrect PIN entries, to pinpoint areas for user experience improvement.
- **High-Value Transactions:** Flagging and analyzing unusually high-value transactions to identify potential instances of fraud.
- **Credit Risk:** Identifying customers who may be at an increased risk of default based on their debt-to-income ratios and credit score trends.
- **Debt Levels:** Analyzing the distribution of total debt across the customer portfolio to better understand and manage overall portfolio risk.

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

**Key Findings:**

**1. High Transaction Volume, Moderate Spending:** Categories like **Grocery Stores and Supermarkets** (**MCC 5411**) show the highest number of transactions at **18,371**, indicating frequent purchases by our customer base. However, the total spending in this category, at **$501,980.56**, while substantial, is not the highest, suggesting that these are typically smaller, everyday purchases.


**2. Moderate Transaction Volume, High Spending:** Conversely, categories such as **Money Transfer (MCC 4829)** exhibit only **7,257** transactions but the highest overall spending of **$596,560.** This suggests that while these transactions are less frequent, they involve significantly larger amounts of money per transaction (*an **average** of approximately **$82** per transaction*).


**3. Bulk Purchasing Behavior: Wholesale Clubs (MCC 5300)** also demonstrate high total spending at **$471,078.75**, coupled with a considerable number of transactions at **7,573.** The relatively high **average** amount spent per transaction (*around **$62***) in this category likely reflects customers buying goods in bulk.


**4. Discretionary Spending Trends:** When we look at **dining, Eating Places and Restaurants (MCC 5812)** show **11,673** transactions with a total spend of **$316,035.8**, and **Fast Food Restaurants (MCC 5814)** have **5,888** transactions totaling **$158,989.44.** Interestingly, **Drinking Places (Alcoholic Beverages) (MCC 5813)** have a much lower transaction volume of **2,838** with a total spend of **$67,352.98**, which could be due to various factors such as less frequent visits or regional differences in consumption patterns.


**5. Essential Services Spending:** Categories like **Service Stations (MCC 5541)** with **17,055** transactions and a total spend of **$380,788.89**, and **Tolls and Bridge Fees (MCC 4784)** with **8,242** transactions and **$289,230.09** in spending, consistently show a high number of transactions and significant total spending, highlighting necessary expenditures related to transportation.


**Business Implications**

Based on these spending patterns, the following business implications can be identified:

- **Elevated Fraud Risk in High-Value Transfers:** The high average transaction value in the **Money Transfer (MCC 4829)** category signifies a heightened potential for significant financial losses due to fraudulent activities.


- **Opportunity for Targeted Marketing in High-Frequency Retail:** The substantial transaction volume in **Grocery Stores and Supermarkets (MCC 5411)** presents a significant opportunity for targeted marketing efforts aimed at influencing purchasing behavior and increasing customer lifetime value within this large segment.


- **Potential for Business Customer Segmentation:** The high spending observed in **Money Transfer (MCC 4829)** and **Wholesale Clubs (MCC 5300)** indicates the presence of customer segments that may include businesses or organizations with distinct financial needs.


- **Importance of Efficient Transaction Processing for Essential Services:** The high transaction volumes in categories like **Grocery Stores (MCC 5411), Service Stations (MCC 5541), and Tolls and Bridge Fees (MCC 4784)** underscore the critical need for reliable and efficient transaction systems to support these high-frequency, essential services.


- **Possible Data Anomalies or Regional Factors in Low-Volume Categories:** The lower transaction volume in Drinking Places **(Alcoholic Beverages) (MCC 5813)** suggests potential data inconsistencies or the influence of regional regulations or cultural consumption patterns that require further understanding.


- The consistent activity in essential services like **Drug Stores and Pharmacies(MCC 5912)**, **Utilities - Electric, Gas, Water, Sanitary(MCC 4900)**, and **Telecommunication Services(MCC 4814)** underscores the reliance of customers on these sectors and their predictable spending patterns within them. 


### 1.2. Geographical Spending Patterns 

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

**Key Findings & Business Implications:**

* **Online transactions** accounted for **17,921 transactions** and **\$1,048,210.76** in spend, making it the dominant channel. This reflects high customer reliance on digital platforms and signals that a significant share of business activity occurs outside physical locations, which has implications for risk exposure and digital engagement strategies.


* **Houston, TX** had the highest volume among physical locations with **2,268 transactions** and **\$123,275.36** in total spend. This suggests a concentration of customer activity in a major metropolitan area, relevant for geographic risk profiling and demand planning.


* **North Hollywood, CA** had **2,232 transactions** but only **\$60,191.31** in spend, indicating a high volume of low-value transactions. This may point to routine or necessity-based spending behavior and a customer segment with lower average spend per visit.


* **Orlando, FL** and **Yorba Linda, CA** recorded fewer transactions (**1,611 and 1,523**) but high total spending (**\$84,927.89 and \$83,119.60**), which could reflect higher-value consumer segments or merchants offering larger-ticket items.


* **Atlanta, GA** showed balanced activity with **1,321 transactions** and **\$84,536.03** in spend, suggesting both consistent customer engagement and above-average transaction value, making it a commercially significant location.


* **Bronx, NY** and **Sacramento, CA** had relatively lower volumes (**\~1,100 transactions each**) and lower spend (**\$31,241.09 and \$48,984.91**, respectively), which may indicate markets with less penetration or lower purchasing power, potentially relevant in assessing customer base diversity and regional performance variability.

---

## 2. Geographical Trends

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

**Key Findings & Business Implications:**

* **Online transactions** had the highest number of failed attempts, with **445 failed transactions** totaling **\$26,353.81**. This represents a **high-risk area for customer friction** in digital channels.


* **Orlando, FL** led among physical locations with **67 failed transactions** and **\$6,274.85** in spend. This suggests a combination of **strong customer activity** and **notable operational issues**, which may affect satisfaction and trust.


* **Newton Grove, NC** and **San Antonio, TX** had **27 and 39 failures**, with a spend of **\$2,620.00** and **\$2,496.90**, respectively. Despite lower transaction volumes, these figures highlight that **failed transactions can carry meaningful financial impact** even in smaller markets.


* **Atlanta, GA** and **Yorba Linda, CA** reported **\$2,242.77** and **\$2,137.79** in spend from failed transactions. These locations also appear in the list of high-value transaction zones, reinforcing their **dual importance in value and risk monitoring**.


* Locations like **Rome, Italy** and **Alburtis, PA** showed **low failure counts (10 and 31)** but **high average transaction values**, reflecting that **transaction risk isn’t limited to high-volume areas** and can occur in isolated but high-stakes instances.


* Multiple **California cities**—**Oakland, Sacramento, North Hollywood, Carmel Valley**—registered failed transaction attempts between **\$1,100 and \$1,700**, indicating a **broad pattern of minor disruptions** across a key regional market.


* **Houston, TX** and **Bronx, NY**, both high in overall transaction volume, also experienced **25 failed transactions each**, with of **\$1,278.88** and **\$1,262.97**, suggesting that **high-traffic areas naturally carry greater exposure to failed attempts**.

---

