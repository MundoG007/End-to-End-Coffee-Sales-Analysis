# End-to-End-Coffee-Sales-Analysis

/* COFFEE SALES DATA CONSOLIDATION & CLEANING
   Objective: Combine multi-year sales data and enrich with Customer/Product dimensions.
*/

-- Step 1: Consolidate yearly tables into a single temporary result set (CTE)
WITH all_orders AS (
    SELECT * FROM orders_2023
    UNION ALL
    SELECT * FROM orders_2024
    UNION ALL
    SELECT * FROM orders_2025
)

-- Step 2: Final Selection and Data Enrichment
SELECT 
    a.OrderID,
    a.CustomerID,
    c.Region,
    a.ProductID,
    a.OrderDate,
    
    -- TIME INTELLIGENCE: Creating a standard 'Week Start' date for time-series analysis
    DATEADD(WEEK, DATEDIFF(WEEK, 0, a.OrderDate), 0) AS Week_Date,
    
    c.CustomerJoinDate,
    a.Quantity,
    
    -- DATA FORMATTING: Standardizing revenue to 2 decimal places
    ROUND(a.Revenue, 2) AS Revenue,
    
    -- DATA CLEANING: Logic to handle missing values (Imputation)
    -- If Revenue is NULL, we reconstruct it using (Product Price * Quantity)
    CASE 
        WHEN ROUND(a.Revenue, 2) IS NULL THEN p.Price * a.Quantity 
        ELSE a.Revenue 
    END AS CleanedRevenue,
    
    ROUND(a.COGS, 2) AS COGS,
    
    -- CALCULATED FIELD: Real-time Profit calculation
    ROUND(a.Revenue - a.COGS, 2) AS Profit,
    
    p.ProductName,
    p.ProductCategory,
    ROUND(p.Price, 2) AS Price,
    ROUND(p.Base_Cost, 2) AS Base_Cost

FROM all_orders AS a
-- Enrichment via Left Joins ensures we keep all sales records even if metadata is missing
LEFT JOIN customers AS c ON a.customerID = c.CustomerID
LEFT JOIN products AS p ON a.ProductID = p.ProductID

-- VALIDATION: Ensuring data integrity by removing records without a valid CustomerID
WHERE a.CustomerID IS NOT NULL;
