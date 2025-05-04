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

#### **Key Findings & Business Implications:**

* **Grocery Stores and Supermarkets (MCC 5411)** recorded the highest transaction count at **18,371**, with total spending of **\$501,980.56**. This reflects high-frequency, low-to-moderate value purchases typical of everyday consumer behavior. The consistent volume underscores this category’s importance for customer engagement and operational reliability.


* **Money Transfer services (MCC 4829)** had a significantly lower transaction count of **7,257** but the highest overall spend at **\$596,560**, with an average of around **\$82 per transaction**. This indicates a high-value, low-frequency pattern often associated with financial services, pointing to elevated fraud exposure and potentially different user segments with larger transactional stakes.


* **Wholesale Clubs (MCC 5300)** also showed strong performance, with **7,573 transactions** and **\$471,078.75** in spending. The relatively high per-transaction average (about **\$62**) suggests bulk purchasing behavior, possibly by small businesses or value-conscious consumers—relevant for segmentation and pricing insights.


* **Dining-related categories** showed varied behavior:

  * **Eating Places and Restaurants (MCC 5812):** **11,673 transactions**, **\$316,035.80**
  * **Fast Food (MCC 5814):** **5,888 transactions**, **\$158,989.44**
  * **Drinking Places (Alcoholic Beverages) (MCC 5813):** **2,838 transactions**, **\$67,352.98**
    This spread reflects different levels of discretionary spending, with lower engagement in alcohol-related venues possibly due to cultural, regulatory, or regional factors.


* **Essential service categories** such as **Service Stations (MCC 5541)** with **17,055 transactions** and **\$380,788.89**, and **Tolls and Bridge Fees (MCC 4784)** with **8,242 transactions** and **\$289,230.09**, show sustained high usage. Their predictability signals reliability in recurring customer behavior and the operational importance of seamless transaction processing.


* Other key utility categories—including **Drug Stores and Pharmacies (MCC 5912)**, **Utilities (MCC 4900)**, and **Telecommunication Services (MCC 4814)**—demonstrate both **high transaction volume and significant total spend**, highlighting their role in essential daily life and their potential stability as part of the bank's transaction ecosystem.


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

#### **Key Findings & Business Implications:**

* **Online transactions** accounted for **17,921 transactions** and **\$1,048,210.76** in spend, making it the dominant channel. This reflects high customer reliance on digital platforms and signals that a significant share of business activity occurs outside physical locations, which has implications for risk exposure and digital engagement strategies.


* **Houston, TX** had the highest volume among physical locations with **2,268 transactions** and **\$123,275.36** in total spend. This suggests a concentration of customer activity in a major metropolitan area, relevant for geographic risk profiling and demand planning.


* **North Hollywood, CA** had **2,232 transactions** but only **\$60,191.31** in spend, indicating a high volume of low-value transactions. This may point to routine or necessity-based spending behavior and a customer segment with lower average spend per visit.


* **Orlando, FL** and **Yorba Linda, CA** recorded fewer transactions (**1,611 and 1,523**) but high total spending (**\$84,927.89 and \$83,119.60**), which could reflect higher-value consumer segments or merchants offering larger-ticket items.


* **Atlanta, GA** showed balanced activity with **1,321 transactions** and **\$84,536.03** in spend, suggesting both consistent customer engagement and above-average transaction value, making it a commercially significant location.


* **Bronx, NY** and **Sacramento, CA** had relatively lower volumes (**\~1,100 transactions each**) and lower spend (**\$31,241.09 and \$48,984.91**, respectively), which may indicate markets with less penetration or lower purchasing power, potentially relevant in assessing customer base diversity and regional performance variability.

---

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

**Key Findings & Business Implications:**

* **Online transactions** had the highest number of failed attempts, with **445 failed transactions** totaling **\$26,353.81**. This represents a **high-risk area for customer friction** in digital channels.


* **Orlando, FL** led among physical locations with **67 failed transactions** and **\$6,274.85** in spend. This suggests a combination of **strong customer activity** and **notable operational issues**, which may affect satisfaction and trust.


* **Newton Grove, NC** and **San Antonio, TX** had **27 and 39 failures**, with a spend of **\$2,620.00** and **\$2,496.90**, respectively. Despite lower transaction volumes, these figures highlight that **failed transactions can carry meaningful financial impact** even in smaller markets.


* **Atlanta, GA** and **Yorba Linda, CA** reported **\$2,242.77** and **\$2,137.79** in spend from failed transactions. These locations also appear in the list of high-value transaction zones, reinforcing their **dual importance in value and risk monitoring**.


* Locations like **Rome, Italy** and **Alburtis, PA** showed **low failure counts (10 and 31)** but **high average transaction values**, reflecting that **transaction risk isn’t limited to high-volume areas** and can occur in isolated but high-stakes instances.


* Multiple **California cities**—**Oakland, Sacramento, North Hollywood, Carmel Valley**—registered failed transaction attempts between **\$1,100 and \$1,700**, indicating a **broad pattern of minor disruptions** across a key regional market.


* **Houston, TX** and **Bronx, NY**, both high in overall transaction volume, also experienced **25 failed transactions each**, with of **\$1,278.88** and **\$1,262.97**, suggesting that **high-traffic areas naturally carry greater exposure to failed attempts**.

---

## 3. Error Trends: Most Common Transaction Errors

```sql
SELECT 
    errors, 
    COUNT(id) AS Total_Errors
FROM transactions_data
WHERE errors IS NOT NULL AND errors != 'No Error Occurred'
GROUP BY errors
ORDER BY Total_Errors DESC;
```

#### **Key Findings & Business Implications:**

* **Insufficient Balance** was the most common error, with **1,760 occurrences**, indicating a high frequency of attempted transactions without adequate funds. This points to a recurring financial strain among some users and highlights potential credit or overdraft product opportunities.

  
* **Bad PIN entries** were the second-most frequent errors with **388 instances**, The prevalence of bad PIN entries suggests either forgetfulness or potential unauthorized access attempts.


* The occurrence of **Technical Glitches(324 errors)**  indicates potential vulnerabilities or instability in the transaction processing system, which could erode customer trust and impact operational efficiency. These glitches peaked at **2 PM** and in **December 2022** (*holiday rush*), suggesting system overload.


* Other errors like **Bad Card Number (93)**, **Bad Expiration (73)**, **Bad CVV (65)**, and **Bad Zipcode (17)** occurred at lower volumes but still reflect friction in the payment process—likely due to manual entry mistakes, outdated card data, or poor form validation.


* There were a handful of **combined errors** (e.g., “Bad PIN, Insufficient Balance”), totaling **18 cases** across various combinations. These multi-error attempts suggest repeated failed efforts during a single session, potentially frustrating customers and increasing the risk of abandonment.

---

### 3.1. Location with Most Errors
**Transaction Error Rates by State**

```sql
SELECT 
    merchant_state, 
    COUNT(id) AS total_transactions,
    COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) AS total_errors,
    ROUND((COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) * 100.0) / COUNT(id), 2) AS error_rate
FROM transactions_data
GROUP BY merchant_state
HAVING ROUND((COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) * 100.0) / COUNT(id), 2) > 0
ORDER BY error_rate DESC;
```

#### **Key Findings & Implications:**

* **High Error Rates in Low-Activity Locations**
  States and countries with very few transactions such as **South Korea (33.3%), Wyoming (6.9%), and Ireland (5.56%)** show disproportionately high error rates implying that while the error rate is high, the overall impact might be limited due to the low transaction volume. These figures may also reflect outliers or irregular usage patterns rather than systemic issues.

* **Moderate to High Error Rates in High-Volume States**
  **Louisiana (3.13%), Florida (2.54%), and Pennsylvania (2.44%)** stand out for combining high transaction volumes with elevated error rates. This suggests that users in these locations encounter errors more frequently than the national average.

* **Online Transactions Show Consistently Elevated Error Rates**
  The **online** channel, which represents the highest transaction volume (**17,921**), also records a relatively high error rate of **2.48%**, indicating that transaction issues are not limited to specific geographies.

* **Error Rates Cluster in a Narrow Range Across Most States**
  Among the majority of U.S. states, error rates fall within a **1.2%** to **2.5%** range. This suggests a broadly stable transaction process, with some variability that may be influenced by regional or infrastructural factors.


### 3.2. Error Rates by Chip Usage

**Error Rate by Transaction Method**
```sql
SELECT 
    use_chip, 
    COUNT(id) AS total_transactions,
    COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) AS error_count,
    ROUND((COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) * 100.0) / COUNT(id), 2) AS error_rate
FROM transactions_data
GROUP BY use_chip;
```


#### **Key Findings & Implications:**


* **Online Transactions** have the **highest error rate** at **2.5%**, even with lower volume that translates to a significant number of failed transactions (**445**), potentially impacting a large segment of customers and requiring attention to optimize the online transaction process.

  
* **Chip Transactions** are the most used and the **most reliable**(**1.58%** error rate) compared to online and swipe, suggesting that chip technology is a relatively reliable method for processing transactions.

  
* The highest error rate observed for **Swipe Transactions** (**1.74%**) despite the lowest transaction volume (**2,927**) indicates that this method might be more prone to errors. This could be due to various factors such as card reader issues or user error during swiping.

### 3.3. Errors By Month

```sql
SELECT 
    FORMAT(date, 'yyyy-MM') AS error_month,  -- Formats as YYYY-MM
    COUNT(id) AS total_transactions,
    COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) AS error_count,
    ROUND(
        (COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) * 100.0) / COUNT(id), 
        2
    ) AS error_rate
FROM transactions_data
GROUP BY FORMAT(date, 'yyyy-MM')
ORDER BY error_rate DESC, error_month;
```
#### **Key Findings & Implications:**

- **April 2024 (2.13%)**
April 2024 recorded the highest error rate **(2.13%)** across **4,553** transactions, indicating a recent spike in failed attempts that may signal emerging operational or user-experience issues.

- **February 2023 (1.87%)**
February had the highest error rate of the year with **80** errors from **4,272** transactions. This spike early in the year suggests potential onboarding issues, post-holiday user behavior shifts, or lagging system performance after peak-season stress.


- **September 2023 (1.79%)**
September logged **82** errors from **4,579** transactions. This combination of high error count and elevated rate indicates ongoing friction, possibly linked to increased digital activity heading into Q4.


- **November 2023 (1.71%)**
With **77** errors and an error rate of **1.71%**, November reflects growing pressure on the system during the lead-up to holiday shopping. This period may reveal bottlenecks in processing or error handling under volume surges.


- **December 2022 (2.05%)**
Despite being a holiday season with likely high consumer activity, **December 2022**’s elevated error rate **(2.05%)** suggests that increased demand may have strained systems or caused more user mistakes.


- **January 2022 (1.98%)**
The start of **2022** saw a near **-2%** error rate from **4,642** transactions, implying that system performance or user readiness might dip after year-end transitions.


- **July 2022 (1.95%)**
With **91** errors out of **4,656** transactions, **July 2022** shows an above-average error rate, possibly reflecting seasonal usage patterns or technical inconsistencies during summer months.


- **August 2022 (1.92%)**
August’s error rate remained elevated at **1.92%**, suggesting a persistence of the same underlying issue that affected July, which could point to unresolved system or interface challenges.

### 3.4. Errors by Hour

```sql
SELECT 
    DATEPART(HOUR, date) AS error_hour, 
    COUNT(id) AS total_transactions,
    COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) AS error_count,
    ROUND(
        (COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) * 100.0) / COUNT(id), 
        2
    ) AS error_rate
FROM transactions_data
GROUP BY DATEPART(HOUR, date)
ORDER BY error_rate DESC;
```

- **Hour 4: Highest Error Rate (2.85%) with Low Volume**
Although only **1,578** transactions occurred at **4 AM**, it had the highest error rate of all hours. This likely reflects system maintenance windows or lower supervision during off-peak hours, suggesting increased risk of transaction failure in early-morning processing.


- **Hour 14 (2 PM): Peak Activity with Elevated Errors**
**2 PM** saw one of the highest transaction volumes (**11,046**) and **241** errors, resulting in a **2.18%** error rate. The combination of high traffic and elevated errors indicates this is a critical stress point in the day.


- **Hour 3: Consistently High Error Rate (2.84%)**
At **3 AM**, despite just **1,020** transactions, the error rate nearly matches the peak hour. This pattern reinforces concerns around night-time system reliability or user fatigue leading to higher input errors, requiring review of processes active during these hours.


**low-error hours insights**

- **Hour 8 (8 AM): Low Error Rate (1.32%) with High Usage**
With over **10,000** transactions and only a **1.32%** error rate, **8 AM** shows both high activity and system stability. This indicates well-functioning infrastructure and user readiness in early business hours, reinforcing it as a low-risk period for transaction processing.


- **Hour 22 (10 PM): Lowest Error Rate (1.32%) in Late Hours**
Despite being outside typical business hours, **10 PM** maintains the lowest error rate among all periods. This suggests improved system reliability or fewer distractions for users late at night, highlighting it as an unexpectedly stable time for processing.


- **Hour 9 (9 AM): Strong Volume with Consistent Performance (1.54%)**
With over **11,000** transactions and a low error rate, **9 AM** represents a key hour of efficient throughput. This balance of volume and reliability makes it an optimal period for handling high transaction loads with minimal disruption.


### 3.5. Error Rate by day

```sql
SELECT 
    CAST(date AS DATE) AS error_date,  -- Extracts only the date part
    COUNT(id) AS total_transactions,
    COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) AS error_count,
    ROUND(
        (COUNT(CASE WHEN errors IS NOT NULL AND errors != 'No Error Occurred' THEN 1 END) * 100.0) / COUNT(id), 
        2
    ) AS error_rate
FROM transactions_data
GROUP BY CAST(date AS DATE)
ORDER BY error_rate DESC, error_date;
```

* **Observation (Highest Error Rate):** The highest error rate is **6.16%** on **2024-03-18**, with **11** errors out of **179** total transactions.

    * **Implication:** This single day in March 2024 experienced a significantly higher proportion of failed transactions compared to other days in the dataset. This warrants investigation into any specific events, system issues, or anomalies that might have occurred on this date.

      
* **Observation (Lowest Error Rate):** Several days show the lowest error rate of **4.26%**, including **2024-06-01** (***7** errors out of **165 transactions***). Other days with similarly low rates include **2023-02-26** (6 errors out of 128 transactions) and **2024-04-04** (6 errors out of 159 transactions).

    * **Implication:** These days represent periods of relatively stable transaction processing with a lower proportion of errors. Analyzing the system status and transaction patterns on these days might provide insights into best practices or conditions that contribute to higher transaction success rates.






