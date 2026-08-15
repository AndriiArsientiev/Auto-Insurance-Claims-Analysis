<img width="372" height="154" alt="image" src="https://github.com/user-attachments/assets/df42b847-8d4a-43f0-89f4-fab3d8f2eedd" /># Auto-Insurance-Claims-Analysis

SQL project | Analysis of auto insurance claims

## Table of Contents

- Business Task
- Relationship Diagram
- Questions and Solutions

## Business Task

The auto insurance data appears profitable overall, but the question is: where exactly is the money being lost?

The analysis explores whether the losses come from specific segments (states, coverage types, sales channels) or from individual problem customers. It also examines whether complaints signal financial risk, and whether the retention strategy is targeting the right people.

The goal is to find the customers costing the company money, understand what makes them different, and produce a concrete action plan.

## Relationship Diagram

#Table 1 here

#Table 2 here

## Questions and Solutions

### A. Data Quality Check

**1. How many records do we have and are there duplicates?**

```sql
SELECT 'Total Records' AS metric, COUNT(*) AS value
FROM insurance_claims
UNION ALL
SELECT 'Customer - Unique', COUNT(DISTINCT "Customer")
FROM insurance_claims
UNION ALL
SELECT 'State - Unique', COUNT(DISTINCT "State")
FROM insurance_claims
UNION ALL
SELECT 'Coverage - Unique', COUNT(DISTINCT "Coverage")
FROM insurance_claims
UNION ALL
SELECT 'Policy Type - Unique', COUNT(DISTINCT "Policy Type")
FROM insurance_claims
UNION ALL
SELECT 'Sales Channel - Unique', COUNT(DISTINCT "Sales Channel")
FROM insurance_claims
UNION ALL
SELECT 'Response - Unique', COUNT(DISTINCT "Response")
FROM insurance_claims
UNION ALL
SELECT 'Renew Offer Type - Unique', COUNT(DISTINCT "Renew Offer Type")
FROM insurance_claims;

```
<img width="268" height="228" alt="image" src="https://github.com/user-attachments/assets/ed99676f-cc15-432a-ba00-35f60c32b541" />

**2. Create a cleaned table**

```sql
SELECT "Customer", "State", "Monthly Premium Auto", "Total Claim Amount"
FROM insurance_claims
LIMIT 5;
```

```sql
CREATE TABLE claims_clean AS
SELECT
  "Customer" AS customer,
  "State" AS state,
  CAST("Customer Lifetime Value" AS REAL) AS clv,
  "Response" AS response,
  "Coverage" AS coverage,
  "Education" AS education,
  "Effective To Date" AS effective_to_date,
  "Employment Status" AS employment_status,
  "Gender" AS gender,
  CAST("Income" AS REAL) AS income,
  "Location" AS location,
  "Marital Status" AS marital_status,
  CAST("Monthly Premium Auto" AS REAL) AS monthly_premium,
  CAST("Months Since Last Claim" AS INTEGER) AS months_since_last_claim,
  CAST("Months Since Policy Inception" AS INTEGER) AS months_since_inception,
  CAST("Number of Open Complaints" AS INTEGER) AS open_complaints,
  CAST("Number of Policies" AS INTEGER) AS num_policies,
  "Policy Type" AS policy_type,
  "Policy" AS policy,
  "Renew Offer Type" AS renew_offer_type,
  "Sales Channel" AS sales_channel,
  CAST("Total Claim Amount" AS REAL) AS total_claim_amount,
  "Vehicle Class" AS vehicle_class,
  "Vehicle Size" AS vehicle_size
FROM insurance_claims;
```

```sql
SELECT COUNT(*) FROM claims_clean;
```

**3. Are the financial fields complete?**

```sql
SELECT 'Premium - Null %' AS metric, ROUND(100.0 * (COUNT(*) - COUNT(monthly_premium)) / COUNT(*), 2) AS value
FROM claims_clean
UNION ALL
SELECT 'Premium - Min', MIN(monthly_premium) FROM claims_clean
UNION ALL
SELECT 'Premium - Max', MAX(monthly_premium) FROM claims_clean
UNION ALL
SELECT 'Premium - Avg', ROUND(AVG(monthly_premium), 2) FROM claims_clean
UNION ALL
SELECT 'Claim - Null %', ROUND(100.0 * (COUNT(*) - COUNT(total_claim_amount)) / COUNT(*), 2) FROM claims_clean
UNION ALL
SELECT 'Claim - Min', MIN(total_claim_amount) FROM claims_clean
UNION ALL
SELECT 'Claim - Max', MAX(total_claim_amount) FROM claims_clean
UNION ALL
SELECT 'Claim - Avg', ROUND(AVG(total_claim_amount), 2) FROM claims_clean
UNION ALL
SELECT 'CLV - Null %', ROUND(100.0 * (COUNT(*) - COUNT(clv)) / COUNT(*), 2) FROM claims_clean
UNION ALL
SELECT 'CLV - Min', MIN(clv) FROM claims_clean
UNION ALL
SELECT 'CLV - Max', MAX(clv) FROM claims_clean;
```
<img width="240" height="306" alt="image" src="https://github.com/user-attachments/assets/91bf5148-2fcf-44e6-be8a-3358eb76fb13" />

No missing values in premium, claims, or lifetime value. No negative or zero premiums, no negative claims. Premium and claim ranges look reasonable.

**4. What about behavioral fields?**

```sql
SELECT 'Last Claim - Min' AS metric, MIN(months_since_last_claim) AS value
FROM claims_clean
UNION ALL
SELECT 'Last Claim - Max', MAX(months_since_last_claim) FROM claims_clean
UNION ALL
SELECT 'Last Claim - Null %', ROUND(100.0 * (COUNT(*) - COUNT(months_since_last_claim)) / COUNT(*), 2) FROM claims_clean
UNION ALL
SELECT 'Inception - Min', MIN(months_since_inception) FROM claims_clean
UNION ALL
SELECT 'Inception - Max', MAX(months_since_inception) FROM claims_clean
UNION ALL
SELECT 'Inception - Null %', ROUND(100.0 * (COUNT(*) - COUNT(months_since_inception)) / COUNT(*), 2) FROM claims_clean
UNION ALL
SELECT 'Complaints - Min', MIN(open_complaints) FROM claims_clean
UNION ALL
SELECT 'Complaints - Max', MAX(open_complaints) FROM claims_clean
UNION ALL
SELECT 'Complaints - Null %', ROUND(100.0 * (COUNT(*) - COUNT(open_complaints)) / COUNT(*), 2) FROM claims_clean
UNION ALL
SELECT 'Policies - Min', MIN(num_policies) FROM claims_clean
UNION ALL
SELECT 'Policies - Max', MAX(num_policies) FROM claims_clean
UNION ALL
SELECT 'Income - Min', MIN(income) FROM claims_clean
UNION ALL
SELECT 'Income - Max', MAX(income) FROM claims_clean
UNION ALL
SELECT 'Income - Null %', ROUND(100.0 * (COUNT(*) - COUNT(income)) / COUNT(*), 2) FROM claims_clean;
```
<img width="257" height="385" alt="image" src="https://github.com/user-attachments/assets/2f7c74cf-462f-4c0b-991b-d3ad41130c42" />

**5. Any logical violations in the data?**

```sql
SELECT 'Impossible: last claim > inception' AS issue, COUNT(*) AS cnt
FROM claims_clean
WHERE months_since_last_claim > months_since_inception
UNION ALL
SELECT 'Zero or negative premium', COUNT(*) FROM claims_clean WHERE monthly_premium <= 0
UNION ALL
SELECT 'Negative claim amount', COUNT(*) FROM claims_clean WHERE total_claim_amount < 0
UNION ALL
SELECT 'Negative complaints', COUNT(*) FROM claims_clean WHERE open_complaints < 0
UNION ALL
SELECT 'More than 9 policies', COUNT(*) FROM claims_clean WHERE num_policies > 9
UNION ALL
SELECT 'More than 5 open complaints', COUNT(*) FROM claims_clean WHERE open_complaints > 5;
```
<img width="336" height="178" alt="image" src="https://github.com/user-attachments/assets/c3abc31a-265e-4f06-9c04-16fd548941ec" />

A group of customers have their last claim dated before their current policy started which represent history with a previous insurer.

**6. How many customers have zero income?**

```sql
SELECT COUNT(*) AS income_zero_count,
  ROUND(100.0 * COUNT(*) / (SELECT COUNT(*) FROM claims_clean), 2) AS pct_of_total
FROM claims_clean
WHERE income = 0;
```
<img width="213" height="48" alt="image" src="https://github.com/user-attachments/assets/8e6e9b63-865d-4a44-9a94-f997ac607942" />


**7. Are customers with prior insurance history different?**

```sql
SELECT
  CASE WHEN months_since_last_claim > months_since_inception
       THEN 'prior_history' ELSE 'normal' END AS group_type,
  COUNT(*) AS customers,
  ROUND(AVG(total_claim_amount), 2) AS avg_claim,
  ROUND(AVG(monthly_premium), 2) AS avg_premium,
  ROUND(AVG(open_complaints), 2) AS avg_complaints
FROM claims_clean
GROUP BY group_type;
```
<img width="447" height="72" alt="image" src="https://github.com/user-attachments/assets/e6b2d252-9f8a-44ac-8d31-a36f6755eb67" />


Comparing average claims, premiums, and complaints: the prior-history group behaves almost identically to everyone else.

**8. What about date formats?**

```sql
SELECT effective_to_date, COUNT(*) AS cnt
FROM claims_clean
GROUP BY effective_to_date
ORDER BY cnt DESC
LIMIT 10;
```

What we see: effective_to_date comes in mixed formats (2-digit and 4-digit years). So any time-based analysis uses months_since_* fields instead of parsing the dates.

### B. Customer Profitability

**1. How do we calculate profit per customer?**

```sql
CREATE TABLE customer_financials AS
SELECT
  customer,
  state,
  coverage,
  policy_type,
  sales_channel,
  response,
  renew_offer_type,
  open_complaints,
  num_policies,
  clv,
  income,
  months_since_inception,
  monthly_premium,
  total_claim_amount,
  monthly_premium * months_since_inception AS total_premium_paid,
  total_claim_amount - (monthly_premium * months_since_inception) AS net_position,
  ROUND(total_claim_amount / NULLIF(monthly_premium * months_since_inception, 0), 2) AS loss_ratio
FROM claims_clean;
```

```sql
SELECT COUNT(*) FROM customer_financials;
```

Built a table with lifetime premium (monthly × months), net position (claims minus premium), and loss ratio per customer.

**2. What does the portfolio look like overall?**

```sql
SELECT 'Profitable (premium > claims)' AS metric, SUM(CASE WHEN net_position < 0 THEN 1 ELSE 0 END) AS value
FROM customer_financials
UNION ALL
SELECT 'Loss-Making (claims > premium)', SUM(CASE WHEN net_position > 0 THEN 1 ELSE 0 END) FROM customer_financials
UNION ALL
SELECT 'Break-Even', SUM(CASE WHEN net_position = 0 THEN 1 ELSE 0 END) FROM customer_financials
UNION ALL
SELECT 'Loss-Making %', ROUND(100.0 * SUM(CASE WHEN net_position > 0 THEN 1 ELSE 0 END) / COUNT(*), 2) FROM customer_financials
UNION ALL
SELECT 'Share of Claims from Loss-Making %', ROUND(100.0 * SUM(CASE WHEN net_position > 0 THEN total_claim_amount ELSE 0 END) / SUM(total_claim_amount), 2) FROM customer_financials;
```
<img width="304" height="184" alt="image" src="https://github.com/user-attachments/assets/28466789-3cf7-4477-981d-cf7b0101d493" />

What we see: total premium, total claims, overall loss ratio. The vast majority of customers are profitable; a small share loses money.


**3. How do customers split by risk level?**

```sql
SELECT
  CASE
    WHEN loss_ratio > 2 AND open_complaints > 0 THEN '1_Critical'
    WHEN loss_ratio > 1.5 THEN '2_High'
    WHEN loss_ratio > 1 THEN '3_Medium'
    WHEN loss_ratio IS NULL THEN '4_No_Premium'
    ELSE '5_Low'
  END AS risk_band,
  COUNT(*) AS customers,
  ROUND(100.0 * COUNT(*) / (SELECT COUNT(*) FROM customer_financials), 2) AS pct,
  ROUND(SUM(total_claim_amount), 0) AS total_claims,
  ROUND(AVG(loss_ratio), 2) AS avg_loss_ratio
FROM customer_financials
GROUP BY risk_band
ORDER BY risk_band;
```
<img width="338" height="155" alt="image" src="https://github.com/user-attachments/assets/50f74fd1-c4cf-4eb2-a19b-e8efc95408dd" />


A tiny Critical band.

**4. Who are the worst customers?**

```sql
SELECT customer, state, coverage, policy_type, sales_channel,
  ROUND(total_premium_paid, 0) AS premium_paid,
  ROUND(total_claim_amount, 0) AS claims,
  ROUND(net_position, 0) AS net_loss,
  loss_ratio, open_complaints
FROM customer_financials
ORDER BY net_position DESC
LIMIT 20;
```
<img width="478" height="320" alt="image" src="https://github.com/user-attachments/assets/51b2e978-3174-4918-bd07-95e67a62d866" />

The pattern is Basic / Personal Auto via Agent; almost none of them complain.

**5. Can we trust the CLV field from the dataset?**

```sql
WITH q AS (
  SELECT *, NTILE(4) OVER (ORDER BY clv) AS clv_quartile
  FROM customer_financials
)
SELECT clv_quartile,
  ROUND(AVG(clv), 0) AS avg_clv,
  ROUND(AVG(total_premium_paid - total_claim_amount), 0) AS avg_net_profit,
  ROUND(AVG(loss_ratio), 2) AS avg_loss_ratio
FROM q
GROUP BY clv_quartile
ORDER BY clv_quartile;
```
<img width="344" height="128" alt="image" src="https://github.com/user-attachments/assets/0ee3d641-32bd-42ce-80e1-8aab721b87b9" />

### C. Portfolio Analytics

**1. What values do response and offer take?**

```sql
SELECT 'response' AS field, response AS val, COUNT(*) AS cnt
FROM customer_financials
GROUP BY response
UNION ALL
SELECT 'renew_offer', renew_offer_type, COUNT(*)
FROM customer_financials
GROUP BY renew_offer_type;
```
<img width="579" height="135" alt="image" src="https://github.com/user-attachments/assets/ca8d694c-c968-4d9f-bd07-9b0f0818a67d" />


**2. Are there loss-making segments?**

```sql
SELECT state, coverage,
  COUNT(*) AS customers,
  ROUND(SUM(total_premium_paid), 0) AS total_premium,
  ROUND(SUM(total_claim_amount), 0) AS total_claims,
  ROUND(100.0 * SUM(total_claim_amount) / NULLIF(SUM(total_premium_paid), 0), 2) AS loss_ratio_pct,
  ROUND(AVG(open_complaints), 2) AS avg_complaints
FROM customer_financials
WHERE total_premium_paid > 0
GROUP BY state, coverage
HAVING COUNT(*) >= 50
ORDER BY loss_ratio_pct DESC;
```

**3. Does the sales channel matter?**

```sql
SELECT sales_channel,
  COUNT(*) AS customers,
  ROUND(SUM(total_premium_paid), 0) AS total_premium,
  ROUND(100.0 * SUM(total_claim_amount) / NULLIF(SUM(total_premium_paid), 0), 2) AS loss_ratio_pct,
  ROUND(AVG(open_complaints), 2) AS avg_complaints,
  SUM(CASE WHEN net_position > 0 THEN 1 ELSE 0 END) AS loss_making_count
FROM customer_financials
GROUP BY sales_channel
ORDER BY loss_ratio_pct DESC;
```
<img width="579" height="135" alt="image" src="https://github.com/user-attachments/assets/09c959ee-6a58-47b9-8681-b97cec78df07" />

**4. Does geography matter?**

```sql
SELECT state,
  COUNT(*) AS customers,
  ROUND(SUM(total_premium_paid), 0) AS total_premium,
  ROUND(100.0 * SUM(total_claim_amount) / NULLIF(SUM(total_premium_paid), 0), 2) AS loss_ratio_pct,
  ROUND(100.0 * SUM(CASE WHEN open_complaints > 0 THEN 1 ELSE 0 END) / COUNT(*), 2) AS complaint_rate_pct,
  ROUND(AVG(income), 0) AS avg_income
FROM customer_financials
GROUP BY state
ORDER BY loss_ratio_pct DESC;
```
<img width="552" height="158" alt="image" src="https://github.com/user-attachments/assets/dfc2fa98-0aba-4872-b53d-f908b0e23ce0" />


### D. Exceptions

**1. How many customers fall into each problem category?**

```sql
SELECT 'Loss ratio > 2' AS exception_type, COUNT(*) AS cnt, ROUND(SUM(total_claim_amount), 0) AS total_claims
FROM customer_financials WHERE loss_ratio > 2
UNION ALL
SELECT 'Zero premium + claims', COUNT(*), ROUND(SUM(total_claim_amount), 0)
FROM customer_financials WHERE total_premium_paid = 0 AND total_claim_amount > 0
UNION ALL
SELECT 'Claims > $2000 (outliers)', COUNT(*), ROUND(SUM(total_claim_amount), 0)
FROM customer_financials WHERE total_claim_amount > 2000
UNION ALL
SELECT 'Critical (LR>2 + complaints)', COUNT(*), ROUND(SUM(total_claim_amount), 0)
FROM customer_financials WHERE loss_ratio > 2 AND open_complaints > 0;
```
<img width="358" height="115" alt="image" src="https://github.com/user-attachments/assets/ec75e580-9dd3-40f0-865b-869430f2519c" />


**2. Are complaints linked to losing money?**

```sql
SELECT
  CASE WHEN open_complaints > 0 THEN 'with_complaints' ELSE 'no_complaints' END AS grp,
  COUNT(*) AS customers,
  ROUND(AVG(loss_ratio), 2) AS avg_loss_ratio,
  ROUND(100.0 * SUM(CASE WHEN net_position > 0 THEN 1 ELSE 0 END) / COUNT(*), 2) AS loss_making_pct,
  ROUND(100.0 * SUM(CASE WHEN response = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS retention_pct
FROM customer_financials
GROUP BY grp;
```
<img width="491" height="78" alt="image" src="https://github.com/user-attachments/assets/a6b3d797-86ed-4b74-95b7-16edaed097a0" />


**3. Which group renews more often: losing or profitable?**

```sql
SELECT
  CASE
    WHEN loss_ratio > 2 AND open_complaints > 0 THEN '1_Critical'
    WHEN loss_ratio > 1.5 THEN '2_High'
    WHEN loss_ratio > 1 THEN '3_Medium'
    WHEN loss_ratio IS NULL THEN '4_No_Premium'
    ELSE '5_Low'
  END AS risk_band,
  COUNT(*) AS customers,
  ROUND(100.0 * SUM(CASE WHEN response = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS retention_pct,
  ROUND(AVG(open_complaints), 2) AS avg_complaints
FROM customer_financials
GROUP BY risk_band
ORDER BY risk_band;
```
<img width="372" height="154" alt="image" src="https://github.com/user-attachments/assets/246d1b78-33c3-48d2-aa1b-0e7ed3a5c893" />


Losing customers renew more often than profitable ones.

**4. Who is on the final exception list?**

```sql
SELECT customer, state, coverage, policy_type, sales_channel,
  ROUND(total_premium_paid, 0) AS premium_paid,
  ROUND(total_claim_amount, 0) AS claims,
  loss_ratio, open_complaints, response,
  CASE
    WHEN total_premium_paid = 0 AND total_claim_amount > 0 THEN 'New client, immediate claim'
    WHEN loss_ratio > 2 AND open_complaints > 0 THEN 'Big loss + complaint'
    WHEN loss_ratio > 2 THEN 'Big loss'
    WHEN total_claim_amount > 2000 THEN 'Very big claim'
    WHEN open_complaints >= 3 THEN 'Frequent complaints'
  END AS exception_type
FROM customer_financials
WHERE (total_premium_paid = 0 AND total_claim_amount > 0)
   OR loss_ratio > 2
   OR total_claim_amount > 2000
   OR open_complaints >= 3
ORDER BY exception_type, net_position DESC
LIMIT 50;
```
<img width="929" height="1112" alt="image" src="https://github.com/user-attachments/assets/9dedae29-15c0-4c7a-9321-31f6d605c9e8" />


### E. Summary



**Final KPIs**

```sql
SELECT 'Customers' AS metric, COUNT(*) AS value FROM customer_financials
UNION ALL
SELECT 'Total Premium', ROUND(SUM(total_premium_paid), 0) FROM customer_financials
UNION ALL
SELECT 'Total Claims', ROUND(SUM(total_claim_amount), 0) FROM customer_financials
UNION ALL
SELECT 'Overall Loss Ratio %', ROUND(100.0 * SUM(total_claim_amount) / SUM(total_premium_paid), 2) FROM customer_financials
UNION ALL
SELECT 'Loss-Making Customers', SUM(CASE WHEN net_position > 0 THEN 1 ELSE 0 END) FROM customer_financials
UNION ALL
SELECT 'Customers with Complaints', SUM(CASE WHEN open_complaints > 0 THEN 1 ELSE 0 END) FROM customer_financials
UNION ALL
SELECT 'Retention %', ROUND(100.0 * SUM(CASE WHEN response = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) FROM customer_financials
UNION ALL
SELECT 'Exceptions Flagged', SUM(CASE WHEN (total_premium_paid = 0 AND total_claim_amount > 0) OR loss_ratio > 2 OR total_claim_amount > 2000 OR open_complaints >= 3 THEN 1 ELSE 0 END) FROM customer_financials;
```
<img width="314" height="235" alt="image" src="https://github.com/user-attachments/assets/7e72fff3-101d-4f63-bbb6-ca8b2347409e" />

**Action Plan**

```sql
SELECT 'Review big loss clients' AS action, COUNT(*) AS customers, ROUND(SUM(total_claim_amount), 0) AS claims_at_risk
FROM customer_financials WHERE loss_ratio > 2
UNION ALL
SELECT 'Check underwriting for new clients with immediate claims', COUNT(*), ROUND(SUM(total_claim_amount), 0)
FROM customer_financials WHERE total_premium_paid = 0 AND total_claim_amount > 0
UNION ALL
SELECT 'Investigate very big claims (possible fraud check)', COUNT(*), ROUND(SUM(total_claim_amount), 0)
FROM customer_financials WHERE total_claim_amount > 2000
UNION ALL
SELECT 'Call frequent complainers before they leave', COUNT(*), ROUND(SUM(total_claim_amount), 0)
FROM customer_financials WHERE open_complaints >= 3;
```
<img width="476" height="125" alt="image" src="https://github.com/user-attachments/assets/ee44373a-3527-40b3-9152-ec6ac9cdae8a" />

**Final Findings**

- Data is healthy, only a small share of customers loses money
- Segments look the same
- Most losses come from very big claims
- Losing customers renew more often

**Limitations**

- Claim amount is the customer's last claim
- Missing income for 25% of customers
- Dates in mixed formats
