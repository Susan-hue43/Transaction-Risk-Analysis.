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

<img width="770" alt="fraud 1" src="https://github.com/user-attachments/assets/6250e860-1bb5-4241-a43e-da5b288317da" />


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
Most transactions received a fraud score of above **4**, meaning all six detection flags were triggered. The confluence of multiple red flags per transaction implies that these are not isolated anomalies but **high-risk signals**. From a business lens, this raises the probability that systemic vulnerabilities or behavioral shifts are present within certain customer segments or transaction channels.


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

<img width="766" alt="fraud 2" src="https://github.com/user-attachments/assets/c0bd1468-3c7d-480d-9e34-a61f45d984dd" />


#### **Observation and Business Implication**

**1. 33,569 transactions flagged for post-May 2023 anomalies**

* The SQL query identifies **33,569 transactions** occurring after **May 1, 2023**, involving either new merchants, non-chip usage, or critical errors. This high volume reflects a **material level of non-standard transaction behavior** within a single timeframe. It implies a widening gap between legacy customer patterns and newly emerging transactional risks, increasing potential liability exposure.


**3. >70% of sample high-risk transactions are Online**

*  At least **10 out of 15** flagged transactions in the sample have `merchant_city = Online`, and `merchant_state = Online`, indicating card-not-present usage. The prevalence of online transactions in the high-risk pool indicates that **digital payment channels are the dominant vector** for flagged behavior. This implies heightened exposure to CNP fraud and the need for targeted risk signaling in digital operations.


**4. Frequent error types: "Insufficient Balance" and "Bad Card Number"**

* These two error types appear repeatedly, especially for Online transactions with new merchants. These errors imply a pattern of either **credential testing or overleveraged users** attempting payments beyond their means. These indicate synthetic fraud or real-user distress.


**5. Transactions span multiple states and international cities (e.g. California, Mississippi, Rome)**

* Locations flagged include **U.S.** states like **California, Florida, Pennsylvania**, and international ones like **Rome, Italy**. The geographic spread implies that fraud risk is **not concentrated in one market or merchant profile**. For the business, this indicates a **distributed exposure surface**, which may reflect systemic onboarding or verification gaps rather than isolated fraud hotspots.

---
