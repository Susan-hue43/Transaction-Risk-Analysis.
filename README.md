# Transaction-and-Risk-Analysis.

## Introduction:
Transaction Risk Analysis involves businesses evaluating their transaction data to pinpoint anomalies or unusual patterns indicative of fraud or regulatory violations. Key risk factors in this analysis often include location, time, and spending patterns. 

This project focuses on analyzing transaction data from Aurora Bank with the primary goals of understanding customer spending behaviors, strengthening fraud detection measures, and support regulatory compliance. By examining various aspects of transaction activity, this analysis aims to uncover patterns and trends that can inform business strategies for both risk mitigation and customer engagement. The resulting insights are intended to provide a solid foundation for more informed decision-making concerning fraud prevention, operational efficiency, and customer relationship management.

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


* **Wholesale Clubs (MCC 5300)** also showed strong performance, with **7,573**  transactions and **\$471,078.75** in spending. The relatively high per-transaction average (about **\$62**) suggests bulk purchasing behavior, possibly by small businesses or value-conscious consumers—relevant for segmentation and pricing insights.


* **Dining-related categories** showed varied behavior:

  * **Eating Places and Restaurants (MCC 5812):** **11,673** transactions, with total spending of **\$316,035.80**
  * **Fast Food (MCC 5814):** **5,888** transactions, **\$158,989.44** in spending.
  * **Drinking Places (Alcoholic Beverages) (MCC 5813):** **2,838** transactions, and **\$67,352.98** spend.
    
  This spread reflects different levels of discretionary spending, with lower engagement in alcohol-related venues possibly due to cultural, regulatory, or regional factors.


* **Essential service categories** such as **Service Stations (MCC 5541)** with **17,055** transactions and **\$380,788.89** in spendng, **Tolls and Bridge Fees (MCC 4784)** with **8,242** transactions and **\$289,230.09** spend, show sustained high usage. Their predictability signals reliability in recurring customer behavior and the operational importance of seamless transaction processing.


* Other key utility categories - including **Drug Stores and Pharmacies (MCC 5912)**, **Utilities (MCC 4900)**, and **Telecommunication Services (MCC 4814)** demonstrate both **high transaction volume** and **significant total spend**, highlighting their role in essential daily life and their potential stability as part of the bank's transaction ecosystem.


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


* **Orlando, FL** and **Yorba Linda, CA** recorded fewer transactions (**1,611 and 1,523**, respectively) but high total spending (**\$84,927.89 and \$83,119.60**, respectively), which could reflect high-value consumer segments or merchants offering large ticket items.


* **Atlanta, GA** showed balanced activity with **1,321 transactions** and **\$84,536.03** in spending, suggesting both consistent customer engagement and above-average transaction value, making it a commercially significant location.


* **Bronx, NY** and **Sacramento, CA** had relatively lower volumes (**\~1,100 transactions each**) and lower spending (**\$31,241.09 and \$48,984.91**, respectively), which may indicate markets with less penetration or lower purchasing power, potentially relevant in assessing customer base diversity and regional performance variability.

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

* **Online transactions** had the highest number of failed attempts, with **445 failed transactions** totaling **\$26,353.81**. This represents a **high-risk** area for customer friction in digital channels.


* **Orlando, FL** led among physical locations with **67 failed transactions** and **\$6,274.85** in spend. This suggests a combination of **strong customer activity** and **notable operational issues**, which may affect satisfaction and trust.


* **Newton Grove, NC** and **San Antonio, TX** had **27 and 39 failures**, with a spend of **\$2,620.00** and **\$2,496.90**, respectively. Despite lower transaction volumes, these figures highlight that failed transactions can carry meaningful financial impact even in smaller markets.


* **Atlanta, GA** and **Yorba Linda, CA** reported **\$2,242.77** and **\$2,137.79** in spend from failed transactions. These locations also appear in the list of high-value transaction zones, reinforcing their dual importance in value and risk monitoring.


* Locations like **Rome, Italy** and **Alburtis, PA** showed **low failure counts** (**10** and **31**, respectively) but **high average** transaction values, reflecting that transaction risk isn’t limited to high volume areas and can occur in isolated but high stakes instances.


* Multiple **California cities**; **Oakland, Sacramento, North Hollywood, Carmel Valley** registered failed transaction attempts between **\$1,100 and \$1,700**, indicating a broad pattern of minor disruptions across a key regional market.


* **Houston, TX** and **Bronx, NY**, both high in overall transaction volume, also experienced **25** failed transactions each, with of **\$1,278.88** and **\$1,262.97**, suggesting that high traffic areas naturally carry greater exposure to failed attempts.

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


* Other errors like **Bad Card Number (93)**, **Bad Expiration (73)**, **Bad CVV (65)**, and **Bad Zipcode (17)** occurred at lower volumes but still reflect friction in the payment process likely due to manual entry mistakes, outdated card data, or poor form validation.


* There were a handful of **combined errors** (e.g., “Bad PIN, Insufficient Balance”), totaling **18 cases** across various combinations. These multi-error attempts suggest repeated failed efforts during a single session, potentially frustrating customers and increasing the risk of abandonment.


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

---

## 4. High-Value Transactions

### 4.1. **Flag unusually high-value transactions for potential fraud using stats.**

```sql
--unusually high-value transactions for potential fraud detection where amt > 3 * stddev, percentile95, percentile 99
WITH percentiles AS (
    SELECT 
        client_id,
        id,
        amount,
        mcc,
        date,
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) OVER (PARTITION BY client_id) AS q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) OVER (PARTITION BY client_id) AS q3,
        AVG(amount) OVER (PARTITION BY client_id) AS avg_amt,
        STDEV(amount) OVER (PARTITION BY client_id) AS stddev_amt,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY amount) OVER (PARTITION BY client_id) AS p95,
        PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY amount) OVER (PARTITION BY client_id) AS p99
    FROM transactions_data
),
flagged AS (
    SELECT 
        client_id,
        id,
        amount,
        mcc,
        date,
        q1,
        q3,
        avg_amt,
        stddev_amt,
        p95,
        p99,
        (q3 - q1) AS iqr,
        
        -- Rule 1: IQR Flag
        CASE WHEN amount > q3 + 1.5 * (q3 - q1) THEN 1 ELSE 0 END AS flag_iqr,
        
        -- Rule 2: 3 StdDev Flag
        CASE WHEN amount > avg_amt + 3 * stddev_amt THEN 1 ELSE 0 END AS flag_stddev,
        
        -- Rule 3: Percentile Flags
        CASE WHEN amount > p95 THEN 1 ELSE 0 END AS flag_p95,
        CASE WHEN amount > p99 THEN 1 ELSE 0 END AS flag_p99,
        
        -- Rule 4: MCC Flag (e.g., 7995 = gambling, 6051 = crypto, 4829 = wire transfer)
        CASE WHEN mcc IN (7995, 6051, 4829) THEN 1 ELSE 0 END AS flag_mcc,
        
        -- Rule 5: Time Flag (between 12 a.m. and 5 a.m.)
        CASE 
            WHEN DATEPART(HOUR, date) BETWEEN 0 AND 5 THEN 1
            ELSE 0 
        END AS flag_time
    FROM percentiles
),
scored AS (
    SELECT *,
        -- Sum of all fraud indicators
        flag_iqr + flag_stddev + flag_p95 + flag_p99 + flag_mcc + flag_time AS fraud_score,
        
        -- Risk Label
        CASE 
            WHEN flag_iqr + flag_stddev + flag_p95 + flag_p99 + flag_mcc + flag_time >= 4 THEN 'High Risk'
            WHEN flag_iqr + flag_stddev + flag_p95 + flag_p99 + flag_mcc + flag_time = 3 THEN 'Medium Risk'
            WHEN flag_iqr + flag_stddev + flag_p95 + flag_p99 + flag_mcc + flag_time = 2 THEN 'Low Risk'
            ELSE 'Normal'
        END AS risk_level
    FROM flagged
)
SELECT *
FROM scored
WHERE fraud_score > 0 AND risk_level != 'Normal'
ORDER BY fraud_score DESC, amount DESC;
```

The SQL detects potentially fraudulent transactions by applying a combination of rules:

* **IQR (Interquartile Range) Rule:** Flags transactions that are significantly **higher** than the **third quartile (Q3)** of a customer's typical spending.
  
* **Standard Deviation Rule:** Flags transactions that are **more** than **3 standard deviations** away from the customer's average transaction amount.
  
* **Percentile Rules:** Flags transactions **exceeding** the **95th and 99th percentiles** of a customer's transaction amounts.
  
* **MCC (Merchant Category Code) Rule:** Flags transactions associated with specific **MCCs** considered high-risk (e.g., **gambling(MCC 7995), cryptocurrency(MCC 6051), wire transfer(MCC 4829)**).
  
* **Time Rule:** Flags transactions occurring **between midnight and 5 a.m**.

The query then assigns a "fraud\_score" based on the number of rules a transaction violates and categorizes transactions into risk levels (**"High Risk," "Medium Risk," "Low Risk", "Normal"**). Finally, it selects transactions with a **fraud score** greater than **0** and a risk level **other than "Normal**," ordering them by fraud score and amount.


#### Observations and Business Implications:

**1. High Frequency of Flagged Transaction.**
The query identified **7,168** transactions as potentially unusual based on statistical thresholds and business rules. The institution handles a significant number of transactions that deviate sharply from typical client behavior. This suggests heightened operational exposure to unusual patterns that may require scrutiny from risk, compliance, and audit teams.


**2. Uniformly High Fraud Scores in Flagged Set.**
Most transactions received a fraud score of **5**, meaning all six detection flags were triggered. The confluence of multiple red flags per transaction implies that these are not isolated anomalies but **high-risk signals**. From a business lens, this raises the probability that systemic vulnerabilities or behavioral shifts are present within certain customer segments or transaction channels.


**3. Repeated Appearance of Risk-Tagged MCCs.**
Flagged transactions include merchant category codes (**MCCs**) tied to activities often linked with regulatory concerns such as ***gambling, cryptocurrency**, and **wire transfers***. This insight may indicate segments of the business that are particularly exposed to *compliance, regulatory, or reputational risks*. Their frequency across **high-risk** transactions may also influence how external auditors or regulators assess the bank’s transaction monitoring rigor.


**4. Transactions During Off-Hours.**
The flagged dataset includes transactions executed between **12 a.m. and 5 a.m.**, a known window for lower oversight and elevated fraud attempts. This pattern suggests a temporal risk exposure where the institution’s infrastructure or controls may face reduced effectiveness, impacting how risk is distributed throughout the day.


**5. Outlier Transaction Amounts Well Beyond Client Norms.**
The amounts exceed not just statistical expectations like **standard deviation**, but also extreme thresholds such as the **99th percentile** and **interquartile range boundaries**. This discrepancy highlights the existence of **exceptionally high-value** activity that falls outside normal behavioral patterns. These extremes may influence how capital and operational risk is assessed for certain accounts or channels and could impact decisions around client monitoring intensity or account provisioning structures.


#### 4.2. Unusually high-value transactions for potential fraud detection using new merchants, no-chip transactions, critical errors.

```sql
--unusually high-value transactions for potential fraud detection using new merchants, no chip transaction, critical errors
-- Step 1: Known merchants/locations before the cutoff date
WITH known_merchants AS (
    SELECT DISTINCT client_id, merchant_id, merchant_city, merchant_state
    FROM transactions_data
    WHERE date < '2023-05-01'
),

-- Step 2: Transactions on or after the cutoff date
current_transactions AS (
    SELECT *
    FROM transactions_data
    WHERE date >= '2023-05-01'
),

-- Step 3: Flag new merchants/locations
merchant_flags AS (
    SELECT 
        t.*,
        CASE 
            WHEN k.merchant_id IS NULL THEN 1 ELSE 0 
        END AS flag_new_merchant
    FROM current_transactions t
    LEFT JOIN known_merchants k
        ON t.client_id = k.client_id 
        AND t.merchant_id = k.merchant_id
        AND t.merchant_city = k.merchant_city
        AND t.merchant_state = k.merchant_state
),

-- Step 4: Flag non-chip transactions
chip_flags AS (
    SELECT *,
        CASE 
            WHEN use_chip <> 'Chip Transaction' THEN 1 ELSE 0 
        END AS flag_no_chip
    FROM merchant_flags
),

-- Step 5: Flag only critical suspicious errors
error_flags AS (
    SELECT *,
        CASE 
            WHEN errors LIKE '%Bad CVV%' 
              OR errors LIKE '%Bad Card Number%'
              OR errors LIKE '%Bad PIN%'
              OR errors LIKE '%Insufficient Balance%'
            THEN 1 ELSE 0 
        END AS flag_error
    FROM chip_flags
)

-- Final fraud scoring and risk classification
SELECT 
    id,
    date,
    client_id,
    card_id,
    amount,
    merchant_id,
    merchant_city,
    merchant_state,
    use_chip,
    errors,
    flag_new_merchant,
    flag_no_chip,
    flag_error,
    flag_new_merchant + flag_no_chip + flag_error AS fraud_score,
    CASE 
        WHEN flag_new_merchant + flag_no_chip + flag_error = 3 THEN 'High Risk'
        WHEN flag_new_merchant + flag_no_chip + flag_error = 2 THEN 'Medium Risk'
        WHEN flag_new_merchant + flag_no_chip + flag_error = 1 THEN 'Low Risk'
        ELSE 'Normal'
    END AS risk_level
FROM error_flags
WHERE flag_new_merchant + flag_no_chip + flag_error > 0
ORDER BY fraud_score DESC, amount DESC;
```

#### **Observation and Business Implication**

**1. 33,569 transactions flagged for post-May 2023 anomalies**

* The SQL logic identifies **33,569 transactions** occurring after **May 1, 2023**, involving either new merchants, non-chip usage, or critical errors. This high volume reflects a **material level of non-standard transaction behavior** within a single timeframe. It implies a widening gap between legacy customer patterns and newly emerging transactional risks, increasing potential liability exposure.



**3. >70% of sample high-risk transactions are Online**

*  At least **10 out of 15** flagged transactions in the sample have `merchant_city = Online`, and `merchant_state = Online`, indicating card-not-present usage. The prevalence of online transactions in the high-risk pool indicates that **digital payment channels are the dominant vector** for flagged behavior. This implies heightened exposure to CNP fraud and the need for targeted risk signaling in digital operations.


**4. Frequent error types: "Insufficient Balance" and "Bad Card Number"**

* These two error types appear repeatedly, especially for Online transactions with new merchants. These errors imply a pattern of either **credential testing or overleveraged users** attempting payments beyond their means. This stresses both fraud controls and customer risk models, and may indicate synthetic fraud or real-user distress.


**5. Transactions span multiple states and international cities (e.g. California, Mississippi, Rome)**

* Locations flagged include **U.S.** states like **California, Florida, Pennsylvania**, and international ones like **Rome, Italy**. The geographic spread implies that fraud risk is **not concentrated in one market or merchant profile**. For the business, this indicates a **distributed exposure surface**, which may reflect systemic onboarding or verification gaps rather than isolated fraud hotspots.

---

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

#### Key Findings and Implications:

#### **Critical Risk Profiles**

* **Debt-to-Income Ratio >200%.**

  * *Example*: User 1801 has a **DTI of 221%** and **credit score of 480**.
  * **Implication**: These users are deep in the red. They are likely relying entirely on borrowed money, suggesting high probability of default without immediate corrective action.


* **Credit Score Collapse.**

  * *Observation*: **89% of “Very High Risk” users have credit scores <500**, well below the U.S. average of 704.
  * **Implication**: These customers have either defaulted in the past or are structurally uncreditworthy. Conventional recovery or repayment strategies are unlikely to be effective.
    

#### **Hidden Risk Segments.**

* **Moderate Debt, Low-Income Profiles.**

  * *Example*: User 1330 with **\$30K debt on \$35K income**.
  * **Implication**: These users don’t appear high-risk at first glance, but limited income leaves them **one unexpected expense away from default**. Traditional risk metrics may underestimate their vulnerability.


---

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

#### Key Findings and Implications:

* **Dominance of Moderate Debt (771 users):** The large number of users in the `'Moderate Debt (50K-150K)'` category implies a significant portion of the portfolio is susceptible to economic downturns or localized financial strains affecting this segment.

  
* **Substantial High Debt Exposure (548 users):** The considerable number of users with `'High Debt (150K-500K)'` suggests a heightened sensitivity to interest rate fluctuations and changes in individual financial stability.

  
* **Concentrated Risk in Very High Debt (381 users, avg.143K):** The substantial average debt and total exposure **(54.6 million)** in the `'Very High Debt (500K+)'` category indicates a significant vulnerability to large financial losses should even a small number of these users default within the local economic context.

  
* **Low Debt Segment (200 users, avg.3K):** The small size of the `'Low Debt (1-50K)'` segment implies a limited immediate risk but also a potentially untapped market for increased credit product penetration.

  
* **Minimal No Debt (102 users):** The very small `'No Debt'` segment suggests a high existing penetration of credit products within your current customer base in the region.
 
  
* **Average Debt Trend (Increasing with Category):** The escalating average debt across categories implies a progressively higher financial risk associated with each increase in debt level within the local lending environment.


## **Overall Recommendations:**

Considering the entirety of the transaction risk analysis conducted, encompassing unusually high-value transactions, debt-to-income ratios, credit score trends (where available in previous turns), and the more recent focus on new merchants, non-chip usage, and critical errors, Aurora Bank should adopt a **holistic and adaptive fraud and credit risk management strategy.** This strategy should integrate insights from all analytical layers to create a more robust and nuanced risk assessment framework. Key recommendations include:

* **Enhanced Integration of Risk Indicators:** Develop a unified risk scoring model that incorporates insights from spending behavior anomalies (statistical outliers and percentile breaches), customer creditworthiness (DTI, credit score trends), and transactional red flags (new merchants, non-chip usage, critical errors). A weighted scoring system should prioritize factors demonstrating the strongest correlation with actual fraud.

  
* **Dynamic Threshold Adjustments:** Implement mechanisms for dynamically adjusting risk thresholds based on evolving fraud patterns, seasonal variations in spending, and macroeconomic factors impacting credit risk. Regularly recalibrating these thresholds will minimize false positives while maintaining sensitivity to emerging threats.

  
* **Proactive Customer Communication and Education:** Engage with customers exhibiting early warning signs of financial distress (e.g., high DTI, declining credit scores, frequent large transactions deviating from their norm) with proactive communication and financial literacy resources. Educate customers on secure transaction practices, particularly regarding chip card usage and awareness of potential fraud schemes involving new merchants.

  
* **Strengthened Verification and Authorization Protocols:** Implement more strict verification and authorization protocols for high-risk transactions identified through the combined analysis, particularly those involving new merchants, non-chip transaction methods, and significant deviations from usual spending behavior. Consider multi-factor authentication and transaction confirmations for these scenarios.

  
* **Continuous Monitoring and Feedback Loop:** Establish a continuous monitoring system that tracks the performance of the integrated risk model and incorporates feedback from fraud investigations and credit risk assessments. This feedback loop will enable ongoing refinement of the detection rules and scoring weights, ensuring the system remains adaptive and effective against evolving threats in the financial landscape.

  
* **Investment in Advanced Analytics and Technology:** Explore and invest in advanced analytics techniques, such as machine learning, to identify more complex fraud patterns and predict credit default with greater accuracy. These technologies can learn from historical data to identify subtle anomalies that rule-based systems might miss.


## **Conclusion:**

We've looked at different ways to find risks in Aurora Bank's transactions, from single unusual events to bigger patterns about how people spend and pay. The key takeaway is that the best way to keep the bank and its customers safe is to have **one smart system that looks at all the clues together.**

By connecting the dots between unusual spending, potential payment problems, and risky transaction types, the bank can get a much clearer picture of where the real dangers are. This will allow them to focus their efforts on the most serious risks, keep customers happy by not causing unnecessary alarms, and ultimately protect everyone's money in the Kenyan financial landscape. The bank needs to be ready to adapt and use the latest technology to stay ahead of new threats.
