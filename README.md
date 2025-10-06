## Credit_Strategy_Analyst
# Assessment 1

## Dataset Overview

All queries use the table Loan_Snapshot_Cleaned, which contains daily loan-level snapshots with fields like:

Column
| Column                 | Description                                |
| ---------------------- | ------------------------------------------ |
| `customer_id`          | Unique customer identifier                 |
| `date`                 | Snapshot date                              |
| `cumulative_repayment` | Total repayment made by that date          |
| `cumulative_interest`  | Total interest accrued by that date        |
| `utilization_pct`      | Utilization percentage of the credit limit |
| `days_in_arrears`      | Number of days a customer is overdue       |
| `risk_band`            | Customer risk rating (Low, Medium, High)   |
| `outstanding_balance`  | Remaining loan balance                     |
| `cumulative_paid`      | Total amount paid (principal + interest)   |
 

1. # Aggregation: 
 ## Write a query to calculate the total cumulative repayment and cumulative interest for each customer after 60 days.

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


This query:

Uses a CTE (loan_days) to calculate how many days have passed since each customer's first record.
The julianday() function computes the difference between dates.
Filters for customers at or after day 60 to see their cumulative repayment and interest status.


2. # Utilization Analysis: 
 ## Find the average utilization (%) across all customers on the final day

SELECT customer_id,AVG(utilization_pct) AS Avg_Utilization_Final_Day
FROM Loan_Snapshot_Cleaned
WHERE date = (SELECT MAX(date) FROM Loan_Snapshot_Cleaned)

OR

SELECT 
    customer_id,date,
    Round(utilization_pct,2) || '%' AS Utilization_Final_DATE
FROM Loan_Snapshot_Cleaned
WHERE date = (SELECT MAX(date) FROM Loan_Snapshot_Cleaned);


# Explanation:

Both queries extract utilization data from the most recent snapshot.

Query (a) computes the average utilization rate.

Query (b) presents each customer’s utilization percentage.


3. # Arrears Tracking: 
 ## For each customer, return the maximum days_in_arrears observed over the 
60-day period

SELECT customer_id,MAX(days_in_arrears)
  FROM Loan_Snapshot_Cleaned
  GROUP BY customer_id

  # Explanation:

Finds the maximum number of days each customer has been in arrears during the observation period.
This is useful for identifying customers with the worst delinquency behavior.
  

4. # Risk Segmentation: 
 ## Count the number of customers in each risk_band on the final day.


SELECT risk_band,COUNT(customer_id) AS Count_Of_Customers,MAX(date)
FROM Loan_Snapshot_Cleaned
GROUP BY risk_band

# Explanation:

Groups customers by their risk_band.
Counts how many fall in each category.
Uses MAX(date) to confirm it’s the latest available data.


5. # Cohort Question: 
 ## How many customers had outstanding_balance = 0 by day 60 (i.e., fully paid 
off)?

SELECT customer_id,COUNT(customer_id) AS Fully_Paid
FROM Loan_Snapshot_Cleaned
WHERE outstanding_balance = 0
and date = (SELECT max(date) FROM Loan_Snapshot_Cleaned)
GROUP BY customer_id

# Explanation:

Identifies customers with zero outstanding balance by the final snapshot.
These are customers who fully repaid their loans within 60 days.


6. # Trend Question: 
## For a given customer, calculate the day-by-day change in cumulative_paid 
(hint: use LAG() in SQL).

SELECT
    customer_id,
    date,
   cumulative_paid AS cumulative_paid,
   cumulative_paid - LAG(cumulative_paid) OVER(PARTITION BY customer_id ORDER BY date) AS daily_change
FROM Loan_Snapshot_Cleaned
ORDER BY customer_id, date;

Explanation:

Uses the LAG() window function to look back at the previous day’s cumulative_paid value.
Calculates the daily change (i.e., how much more was paid each day).
Useful for spotting repayment patterns — e.g., large one-time payments vs steady payments.

