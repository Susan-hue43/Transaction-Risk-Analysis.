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

<img width="766" alt="most common errors" src="https://github.com/user-attachments/assets/8a5709de-61f9-4821-8f00-5a03c8e91c00" />


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

<img width="766" alt="location with most errors" src="https://github.com/user-attachments/assets/84d0f46f-9c66-4516-a3ee-db4e921be692" />


#### **Key Findings & Implications:**

* **High Error Rates in Low-Activity Locations**
  States and countries with very few transactions such as **South Korea (33.3%), Wyoming (6.9%), Ireland (5.56%), and Costa Rica (4.76%)** show disproportionately high error rates implying that while the error rate is high, the overall impact might be limited due to the low transaction volume. These figures may also reflect outliers or irregular usage patterns rather than systemic issues.

* **Moderate to High Error Rates in High-Volume States**
  **Louisiana (3.13%), Florida (2.54%), Pennsylvania (2.44%) and North Carolina (2.23)** stand out for combining high transaction volumes with elevated error rates. This suggests that users in these locations encounter errors more frequently than the national average.

* **Online Transactions Show Consistently Elevated Error Rates**
  The **online** channel, which represents the highest transaction volume (**17,921**), also records a relatively high error rate of **2.48%**, indicating that transaction issues are not limited to specific geographies.

* **Error Rates Cluster in a Narrow Range Across Most States**
  Among the majority of U.S. states, error rates fall within a **1.2%** to **3.13%** range. This suggests a broadly stable transaction process, with some variability that may be influenced by regional or infrastructural factors.


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

<img width="765" alt="errors by chip usage" src="https://github.com/user-attachments/assets/35d9fdba-3689-4037-96a8-eadb6f45916d" />


#### **Key Findings & Implications:**


* **Online Transactions** have the **highest error rate** at **2.5%**, even with lower volume that translates to a significant number of failed transactions (**445**), potentially impacting a large segment of customers and requiring attention to optimize the online transaction process.

  
* **Chip Transactions** are the most used **(112,114)** transaction volume and the **most reliable**(**1.62% error rate)** compared to online and swipe, suggesting that chip technology is a relatively reliable method for processing transactions.

  
* The highest error rate observed for **Swipe Transactions** (**1.74% error rate**) despite the lowest transaction volume (**27,327**) indicates that this method might be more prone to errors. This could be due to various factors such as card reader issues or user error during swiping.

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

<img width="762" alt="errors by month" src="https://github.com/user-attachments/assets/c4164ef9-0dba-44c6-b2e7-2caaa511391c" />


#### **Key Findings & Implications:**

- **April 2024 (2.13%)**
April 2024 recorded the highest error rate of the year with **(2.13%)** across **4,553** transactions, indicating a recent spike in failed attempts that may signal emerging operational or user-experience issues.


- **February 2024 (1.91%)**
February had the second highest error rate of the year with **81** errors from **4,235** transactions.


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

<img width="763" alt="errors per hour" src="https://github.com/user-attachments/assets/83f721d8-fd0d-479a-8b82-46f6d73d8513" />


- **Hour 4: Highest Error Rate (2.85%) with Low Volume**
Although only **1,578** transactions occurred at **4 AM**, it had the highest error rate of all hours. This likely reflects system maintenance windows or lower supervision during off-peak hours, suggesting increased risk of transaction failure in early-morning processing.


- **Hour 14 (2 PM): Peak Activity with Elevated Errors**
**2 PM** saw one of the highest transaction volumes (**11,046**) and **241** errors, resulting in a **2.18%** error rate. The combination of high traffic and elevated errors indicates this is a critical stress point in the day.


- **Hour 3: Consistently High Error Rate (2.84%)**
At **3 AM**, despite just **1,020** transactions, the error rate nearly matches the peak hour. This pattern reinforces concerns around night-time system reliability or user fatigue leading to higher input errors, requiring review of processes active during these hours.


**low-error hours insights**

- **Hour 8 (8 AM): Low Error Rate (1.32%) with High Usage**
With **10,057** transactions and only a **1.32%** error rate, **8 AM** shows both high activity and system stability. This indicates well-functioning infrastructure and user readiness in early business hours, reinforcing it as a low-risk period for transaction processing.


- **Hour 22 (10 PM): Lowest Error Rate (1.32%) in Late Hours**
Despite being outside typical business hours, **10 PM** maintains the lowest error rate among all periods. This suggests improved system reliability or fewer distractions for users late at night, highlighting it as an unexpectedly stable time for processing.


- **Hour 9 (9 AM): Strong Volume with Consistent Performance (1.54%)**
With **11,003** transactions and a low error rate, **9 AM** represents a key hour of efficient throughput. This balance of volume and reliability makes it an optimal period for handling high transaction loads with minimal disruption.


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

<img width="766" alt="errors per day" src="https://github.com/user-attachments/assets/f32608b3-7ff6-44fb-95be-8866514f28a9" />


* **Observation (Highest Error Rate):** The highest error rate is **6.18%** on **2024-04-19**, with **11** errors out of **178** total transactions.

    * **Implication:** This single day in April 2024 experienced a significantly higher proportion of failed transactions compared to other days in the dataset. This warrants investigation into any specific events, system issues, or anomalies that might have occurred on this date.

      
* **Observation (Lowest Error Rate ):** Several days show the lowest error rate of **0%**, starting from **2024-01-21** (***0** errors out of **131 transactions***) all the way to **2024-10-27** (***0** errors out of **164** transactions).
  
    * **Implication:** These days represent periods of relatively stable transaction processing with no errors. Analyzing the system status and transaction patterns on these days might provide insights into best practices or conditions that contribute to higher transaction success rates.

----
