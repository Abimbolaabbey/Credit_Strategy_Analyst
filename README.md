## Credit_Strategy_Analyst

# Assessment_1

## Dataset Overview

# Kindly note that the table name was changed to "Loan_Snapshot_Cleaned" for my Analysis.
 

1. # Aggregation: 
 ## Write a query to calculate the total cumulative repayment and cumulative interest for each customer after 60 days.

```sql
WITH loan_days AS (
    SELECT 
        customer_id,
        date,
        CAST(julianday(date) - julianday(MIN(date) OVER (PARTITION BY customer_id)) + 1 AS INT) AS day_number,
        cumulative_repayment,
        cumulative_interest
    FROM Loan_Snapshot_Cleaned
)
SELECT
    customer_id,
    cumulative_repayment,
    cumulative_interest
FROM loan_days
WHERE day_number >= 60;
```

> Explanation:

Uses a CTE (loan_days) to calculate how many days have passed since each customer's first record.
The julianday() function computes the difference between dates.
Filters for customers at or after day 60 to see their cumulative repayment and interest status.


2. # Utilization Analysis: 
 ## Find the average utilization (%) across all customers on the final day
```sql
SELECT customer_id,AVG(utilization_pct) AS Avg_Utilization_Final_Day
FROM Loan_Snapshot_Cleaned
WHERE date = (SELECT MAX(date) FROM Loan_Snapshot_Cleaned)
```
OR
```sql
SELECT 
    customer_id,date,
    Round(utilization_pct,2) || '%' AS Utilization_Final_DATE
FROM Loan_Snapshot_Cleaned
WHERE date = (SELECT MAX(date) FROM Loan_Snapshot_Cleaned);
```

> Explanation:

Both queries extract utilization data from the most recent snapshot.

Query (a) computes the average utilization rate.

Query (b) presents each customer’s utilization percentage.


3. # Arrears Tracking: 
 ## For each customer, return the maximum days_in_arrears observed over the 
60-day period
```sql
SELECT customer_id,MAX(days_in_arrears)
  FROM Loan_Snapshot_Cleaned
  GROUP BY customer_id
```
  > Explanation:

Finds the maximum number of days each customer has been in arrears during the observation period.
This is useful for identifying customers with the worst delinquency behavior.
  

4. # Risk Segmentation: 
 ## Count the number of customers in each risk_band on the final day.
```sql

SELECT risk_band,COUNT(customer_id) AS Count_Of_Customers,MAX(date)
FROM Loan_Snapshot_Cleaned
GROUP BY risk_band
```
> Explanation:

Groups customers by their risk_band.
Counts how many fall in each category.
Uses MAX(date) to confirm it’s the latest available data.


5. # Cohort Question: 
 ## How many customers had outstanding_balance = 0 by day 60 (i.e., fully paid 
off)?
```sql
SELECT customer_id,COUNT(customer_id) AS Fully_Paid
FROM Loan_Snapshot_Cleaned
WHERE outstanding_balance = 0
and date = (SELECT max(date) FROM Loan_Snapshot_Cleaned)
GROUP BY customer_id
```
> Explanation:

Identifies customers with zero outstanding balance by the final snapshot.
These are customers who fully repaid their loans within 60 days.


6. # Trend Question: 
## For a given customer, calculate the day-by-day change in cumulative_paid 
(hint: use LAG() in SQL).
```sql
SELECT
    customer_id,
    date,
   cumulative_paid AS cumulative_paid,
   cumulative_paid - LAG(cumulative_paid) OVER(PARTITION BY customer_id ORDER BY date) AS daily_change
FROM Loan_Snapshot_Cleaned
ORDER BY customer_id, date;
```
> Explanation:

Uses the LAG() window function to look back at the previous day’s cumulative_paid value.
Calculates the daily change (i.e., how much more was paid each day).
This is useful for spotting repayment patterns e.g., large one-time payments vs steady payments.



## Assessment_2

# Statistics & Data Interpretation

1. Looking at utilization (%), how would you measure which customers are progressing
fastest in repaying their loans?


# Solution:
```>
Utilization (%) = Outstanding Balance ÷ Loan Amount × 100

As customers repay, utilization (%) decreases over time.

While faster repayment leads to steeper decline in utilization.

I will measure Slope of Utilization Over Time (Regression Approach)

For each customer, I will fit a simple linear regression:
Utilization (%) and Time (days)

This slope tells me how quickly utilization is dropping, While more negative slope = faster repayment.
```


2. ## If a customer’s cumulative interest / cumulative repayment ratio is high, what does that tell you about their repayment structure?
```>
# Solution:
What this means is that large share of the borrower’s repayments is going toward interest and not principal.
Or Low Principal reduction.
```

3. ## Suggest a way to visualize repayment performance for all customers over time (linechart, cohort chart, etc.).

```>
# Solution:
I would use a Line Chart (Individual + Portfolio Trend)

X-axis = Time (days or months)

Y-axis = Average utilization (%)

with an average portfolio line to show repayment trajectories.
```



4. ## If we had thousands of customers, how would you identify outliers in repayment behavior?

```>
# Solution:

With thousands of customers, I would use a combination of statistical thresholds and clustering. 
For example, I would calculate repayment speed and utilization trends, then flag customers in the extreme percentiles 
or those whose repayment patterns deviate significantly from their peer group. 
This way, i can quickly identify both high-risk slow payers and unusual fast payers without manually scanning every account.
```

5. ## What metric from this dataset would you use to evaluate portfolio risk over time?
```>
# Solution:

From this dataset, I would evaluate portfolio risk over time using DPD buckets and Portfolio at Risk (PAR). 
Tracking how much of the outstanding balance moves into >30, >60, past due buckets gives a clear view of delinquency trends and default risk. 
I would also complement this with utilization trends to see whether customers are meaningfully reducing their balances or just servicing interest.
```
