# HVAC Service Operations Analytics

## Project Overview

This project analyzes HVAC maintenance and service operations using SQL. The objective was to answer key business questions related to maintenance costs, technician performance, customer value, equipment efficiency, and operational trends.

The analysis was performed on three relational tables:

### Tables
- Customers
- HVAC Units
- Service Records

### Skills Demonstrated
- Data Aggregation
- Multi-Table Joins
- Window Functions
- Common Table Expressions (CTEs)
- Customer Segmentation
- Time-Series Analysis
- Ranking & Performance Analysis
- Business KPI Reporting
## 1. What is the total maintenance cost across all service activities?
```sql
SELECT
    SUM(RepairCost) AS Total_Repair_Cost
FROM Service_Records;
```
## 2. How many service jobs have been successfully completed?
```sql
SELECT
    COUNT(*) AS Total_Completed_Jobs
FROM Service_Records
WHERE Status = 'Completed';
```
## 3. Which technicians completed the highest number of service jobs?
```sql
SELECT
    Technician,
    COUNT(ServiceID) AS Completed_Jobs
FROM Service_Records
WHERE Status = 'Completed'
GROUP BY Technician
ORDER BY Completed_Jobs DESC;```
## 
```sql

```
## 4. Which repair jobs generated the highest maintenance costs?
```sql
SELECT
    ServiceID,
    IssueType,
    RepairCost
FROM Service_Records
ORDER BY RepairCost DESC
LIMIT 5;
```
## 5. What is the total energy consumption associated with serviced HVAC units?
```sql
SELECT
    SUM(Energy_Consumption_kWh) AS Total_Energy_Consumption
FROM Service_Records;
```
## 6. Which customers generate the highest maintenance spending?
```sql
SELECT
    c.CustomerName,
    SUM(s.RepairCost) AS Total_Repair_Cost
FROM Customers c
JOIN HVAC_Units USING(CustomerID)
JOIN Service_Records s USING(UnitID)
GROUP BY c.CustomerID, c.CustomerName
ORDER BY Total_Repair_Cost DESC;
```
## 7. Which HVAC brands generate the highest maintenance costs?
```sql
SELECT
    h.Brand,
    SUM(s.RepairCost) AS Total_Maintenance_Cost
FROM HVAC_Units h
JOIN Service_Records s USING(UnitID)
GROUP BY h.Brand
ORDER BY Total_Maintenance_Cost DESC;
```
## 8. What is the average repair cost per service request?
```sql
SELECT
    AVG(RepairCost) AS Average_Repair_Cost
FROM Service_Records;
```
## 9. How many service requests were recorded each month?
```sql
SELECT
    MONTH(ServiceDate) AS Service_Month,
    COUNT(ServiceID) AS Service_Count
FROM Service_Records
GROUP BY MONTH(ServiceDate)
ORDER BY Service_Month;
```
## 10. Which customer segment generates higher repair costs?
```sql
SELECT
    c.CustomerType,
    SUM(s.RepairCost) AS Total_Repair_Cost
FROM Customers c
JOIN HVAC_Units USING(CustomerID)
JOIN Service_Records s USING(UnitID)
GROUP BY c.CustomerType
ORDER BY Total_Repair_Cost DESC;
```
## 11. Who are the top three customers by maintenance spending?
```sql
SELECT
    c.CustomerName,
    SUM(s.RepairCost) AS Total_Repair_Cost
FROM Customers c
JOIN HVAC_Units USING(CustomerID)
JOIN Service_Records s USING(UnitID)
GROUP BY c.CustomerID, c.CustomerName
ORDER BY Total_Repair_Cost DESC
LIMIT 3;
```
## 12. How do technicians rank based on generated repair revenue?
```sql
SELECT
    Technician,
    SUM(RepairCost) AS Revenue,
    RANK() OVER(
        ORDER BY SUM(RepairCost) DESC
    ) AS Technician_Rank
FROM Service_Records
GROUP BY Technician;
```
## 13. Which HVAC units consume more than 3000 kWh of energy?
```sql
SELECT
    UnitID,
    SUM(Energy_Consumption_kWh) AS Total_Energy
FROM Service_Records
GROUP BY UnitID
HAVING SUM(Energy_Consumption_kWh) > 3000;
```
## 14. What are the first and latest service dates for each HVAC unit?
```sql
SELECT
    UnitID,
    MIN(ServiceDate) AS First_Service_Date,
    MAX(ServiceDate) AS Latest_Service_Date
FROM Service_Records
GROUP BY UnitID;
```
## 15. Which customers require frequent service visits?
```sql
SELECT
    c.CustomerName,
    COUNT(s.ServiceID) AS Service_Visits
FROM Customers c
JOIN HVAC_Units USING(CustomerID)
JOIN Service_Records s USING(UnitID)
GROUP BY c.CustomerID, c.CustomerName
HAVING COUNT(s.ServiceID) > 2;
```
## 16. Which customers spend above the average customer maintenance cost?
```sql
WITH customer_cost AS (
    SELECT
        c.CustomerID,
        c.CustomerName,
        SUM(s.RepairCost) AS Total_Repair_Cost
    FROM Customers c
    JOIN HVAC_Units USING(CustomerID)
    JOIN Service_Records s USING(UnitID)
    GROUP BY c.CustomerID, c.CustomerName
)
SELECT *
FROM customer_cost
WHERE Total_Repair_Cost >
(
    SELECT AVG(Total_Repair_Cost)
    FROM customer_cost
);
```
## 17. How have cumulative maintenance costs evolved over time?
```sql
SELECT
    MONTH(ServiceDate) AS Service_Month,
    SUM(RepairCost) AS Monthly_Cost,
    SUM(SUM(RepairCost))
    OVER(ORDER BY MONTH(ServiceDate)) AS Running_Total
FROM Service_Records
GROUP BY MONTH(ServiceDate);
```
## 18. What percentage of total maintenance cost is contributed by each HVAC brand?
```sql
SELECT
    h.Brand,
    SUM(s.RepairCost) AS Total_Cost,
    ROUND(
        SUM(s.RepairCost) /
        SUM(SUM(s.RepairCost)) OVER() * 100,
        2
    ) AS Contribution_Percentage
FROM HVAC_Units h
JOIN Service_Records s USING(UnitID)
GROUP BY h.Brand;
```
## 19. How has maintenance spending changed month over month?
```sql
SELECT
    MONTH(ServiceDate) AS Service_Month,
    SUM(RepairCost) AS Monthly_Cost,
    LAG(SUM(RepairCost))
    OVER(ORDER BY MONTH(ServiceDate)) AS Previous_Month_Cost
FROM Service_Records
GROUP BY MONTH(ServiceDate);
```
## 20. Who is the top-performing technician within each HVAC brand?
```sql
WITH technician_ranking AS (
    SELECT
        h.Brand,
        s.Technician,
        SUM(s.RepairCost) AS Revenue,
        RANK() OVER(
            PARTITION BY h.Brand
            ORDER BY SUM(s.RepairCost) DESC
        ) AS Ranking
    FROM HVAC_Units h
    JOIN Service_Records s USING(UnitID)
    GROUP BY h.Brand, s.Technician
)
SELECT *
FROM technician_ranking
WHERE Ranking = 1;
```

## 21. What is the chronological sequence of service visits for each customer?
```sql
SELECT
    c.CustomerName,
    s.ServiceDate,
    ROW_NUMBER() OVER(
        PARTITION BY c.CustomerName
        ORDER BY s.ServiceDate
    ) AS Visit_Number
FROM Customers c
JOIN HVAC_Units USING(CustomerID)
JOIN Service_Records s USING(UnitID);
```

## 22. How do HVAC brands rank by total maintenance cost?
```sql
SELECT
    h.Brand,
    SUM(s.RepairCost) AS Total_Cost,
    DENSE_RANK() OVER(
        ORDER BY SUM(s.RepairCost) DESC
    ) AS Brand_Rank
FROM HVAC_Units h
JOIN Service_Records s USING(UnitID)
GROUP BY h.Brand;
```

## 23. How does each month's maintenance cost compare with the following month?
```sql
SELECT
    MONTHNAME(ServiceDate) AS Service_Month,
    SUM(RepairCost) AS Monthly_Cost,
    LEAD(SUM(RepairCost))
    OVER(ORDER BY MONTH(ServiceDate)) AS Next_Month_Cost
FROM Service_Records
GROUP BY MONTH(ServiceDate),
         MONTHNAME(ServiceDate);
```

## 24. Which month experienced the largest increase in maintenance spending?
```sql
WITH monthly_cost AS (
    SELECT
        MONTH(ServiceDate) AS Month_Num,
        MONTHNAME(ServiceDate) AS Month_Name,
        SUM(RepairCost) AS Monthly_Cost
    FROM Service_Records
    GROUP BY MONTH(ServiceDate),
             MONTHNAME(ServiceDate)
)
SELECT
    Month_Name,
    LAG(Monthly_Cost)
    OVER(ORDER BY Month_Num) AS Previous_Month_Cost,
    Monthly_Cost,
    Monthly_Cost -
    LAG(Monthly_Cost)
    OVER(ORDER BY Month_Num) AS Increase_Amount
FROM monthly_cost
ORDER BY Increase_Amount DESC
LIMIT 1;
```
## 25. What was the most expensive repair handled by each technician?
```sql
WITH technician_jobs AS (
    SELECT
        Technician,
        ServiceID,
        RepairCost,
        RANK() OVER(
            PARTITION BY Technician
            ORDER BY RepairCost DESC
        ) AS Ranking
    FROM Service_Records
)
SELECT
    Technician,
    ServiceID,
    RepairCost
FROM technician_jobs
WHERE Ranking = 1;
```





















