# plsql-window-functions-Hagenimana-AimeDeDieu
#  Problem definition 

ADD'S CUISINE is a catering & delivery business specializing in weddings, funerals, corporate events and other ceremonies in Rwanda. They operate regional kitchens and deliver to venues.
Data Challenge: Management needs analytical queries to identify top-selling menu items by region/quarter, track running monthly revenue, compute month-to-month growth for planning, segment customers by revenue, and compute 3-month moving averages for demand forecasting.
Expected Outcome: Produce actionable insights: (1) top menu items per region/quarter to prioritize inventory and promotions; (2) customer segments for VIP targeting; (3) demand trends for staffing and ingredient procurement.

#  Success criteria — exactly 5 measurable goals 
-------------------------------------------------
1.Top 5 menu items per region per quarter : use RANK() to list top 5 by revenue (goal met if query returns 5 items per region/quarter).

2.Running monthly revenue totals for each kitchen/region : SUM() OVER (PARTITION BY region ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) (goal met if cumulative totals computed).

3.Month-over-month growth rate for each region :LAG() to compute previous month revenue and growth% (goal met if growth% computed for all months after first).

4.Customer revenue quartiles : NTILE(4) to assign customers to quartiles (goal met if customers distributed into 4 buckets).

5.3-month moving average of monthly sales for forecasting : AVG() OVER (PARTITION BY region ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) (goal met if 3-month averages computed).

##  Database schema 
-------------------
I have created 5 tables which are these:
----------------------------------
customers: stores customer profiles.

kitchens: regional kitchens (delivery origin).

menu_items: products / items (dishes / packages).

orders: transaction header (one order per event).

order_lines: order details (item & qty) this is essential for proper revenue calculations.

Table of Customers and data
-------------------------
![customer table](screenshots/customers.png)
---------------------------------------
This is the the table and data of customers i created and in folder of screenshots i pushed contains other screenshots of tables i created

## Step 4: Window Functions Implementation (4 pts)
 Navigation:
---------------------------------
Goal: Compute month-over-month revenue and growth%
```sql
WITH monthly_sales AS (
  SELECT k.region,
         TRUNC(o.event_date, 'MM') AS month,
         SUM(ol.quantity * ol.unit_price) AS month_revenue
  FROM orders o
  JOIN order_lines ol ON o.order_id = ol.order_id
  JOIN kitchens k ON o.kitchen_id = k.kitchen_id
  GROUP BY k.region, TRUNC(o.event_date, 'MM')
)
SELECT region,
       month,
       month_revenue,
       LAG(month_revenue) OVER (PARTITION BY region ORDER BY month) AS prev_month_revenue,
       CASE
         WHEN LAG(month_revenue) OVER (PARTITION BY region ORDER BY month) IS NULL THEN NULL
         WHEN LAG(month_revenue) OVER (PARTITION BY region ORDER BY month) = 0 THEN NULL
         ELSE ROUND( (month_revenue - LAG(month_revenue) OVER (PARTITION BY region ORDER BY month)) / LAG(month_revenue) OVER (PARTITION BY region ORDER BY month) * 100, 2)
       END AS mom_growth_pct
FROM monthly_sales
ORDER BY region, month;
```
Output:
![navigation](screenshots/Navigation.png)
-------------------------------------------
LAG() fetches previous month revenue; growth% is calculated safely (avoid division by zero). Use LEAD() similarly for forecasting or next-period comparisons

Running monthly sales totals (SUM() OVER) with frame
---------------------------------------
![aggregate](screenshots/aggregate.png)
-----------------------------------------
This shows cumulative revenue for each region over time. Use ROWS frame for an exact preceding-rows window; for date-aggregates, ordering by month is appropriate.


#  References
Oracle Corporation. (2025). Error Messages Reference (ORA-00933, ORA-02291, etc.).
https://docs.oracle.com/error-help/

Itzik Ben-Gan (2019). T-SQL Window Functions: For Data Analysis and Beyond. Redgate Publishing.

Markus Winand (2012). SQL Performance Explained. Vienna: Markus Winand.
https://use-the-index-luke.com

Mode Analytics (2023). Introduction to SQL Window Functions.
https://mode.com/sql-tutorial/sql-window-functions/

Vertabelo Academy (2024). SQL Window Functions Tutorial.
https://academy.vertabelo.com/blog/sql-window-functions/
