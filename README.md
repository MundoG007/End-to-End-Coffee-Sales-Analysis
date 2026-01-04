# End-to-End-Coffee-Sales-Analysis

## Overview

This project combines and analyzes sales data across multiple years to provide comprehensive insights into business performance. The SQL query consolidates order data from 2023-2025 and enriches it with customer and product information to enable detailed analysis of revenue, profitability, and sales trends.

## Data Sources

The analysis integrates data from three primary tables:

- **orders_2023, orders_2024, orders_2025**: Historical order data spanning three years
- **customers**: Customer information including region and join dates
- **products**: Product catalog with pricing and cost information

## Query Functionality

### Data Consolidation

The query uses a Common Table Expression (CTE) to combine order data across multiple years:

```sql
WITH all_orders AS (
    SELECT * FROM orders_2023
    UNION ALL
    SELECT * FROM orders_2024
    UNION ALL
    SELECT * FROM orders_2025
)

SELECT 
a.OrderID,
a.CustomerID,
c.Region,
a.ProductID,
a.OrderDate,
DATEADD(WEEK, DATEDIFF(WEEK, 0, a.OrderDate),0) AS Week_Date,
c.CustomerJoinDate,
a.Quantity,
ROUND(a.Revenue, 2) AS Revenue,
CASE WHEN ROUND(a.Revenue, 2) IS NULL THEN p.Price * a.Quantity ELSE a.Revenue END AS CleanedRevenue,
ROUND(a.COGS,2) AS COGS,
ROUND(a.Revenue - a.COGS, 2) AS Profit,
p.ProductName,
p.ProductCategory,
ROUND(p.Price, 2) AS Price,
ROUND(p.Base_Cost, 2) Base_Cost
FROM all_orders AS a

LEFT JOIN customers AS c
ON a.customerID = c.CustomerID
LEFT JOIN products AS p
ON a.ProductID = p.ProductID

WHERE a.CustomerID IS NOT NULL
;

```

This approach ensures a unified view of all orders regardless of the year they were placed.

### Key Features

**Week-Based Analysis**: Converts order dates to week start dates for time-series analysis and trend identification.

**Revenue Data Cleaning**: Implements fallback logic to handle null revenue values by calculating from price and quantity when needed.

**Profit Calculation**: Computes profit margins by subtracting COGS (Cost of Goods Sold) from revenue.

**Customer Segmentation**: Links orders to customer data including region and join date for cohort analysis.

**Product Categorization**: Enriches orders with product details for category-level reporting.

## Output Fields

The query produces a comprehensive dataset with the following fields:

- **OrderID**: Unique order identifier
- **CustomerID**: Customer identifier
- **Region**: Customer's geographic region
- **ProductID**: Product identifier
- **OrderDate**: Original order date
- **Week_Date**: Week start date for time-series grouping
- **CustomerJoinDate**: Date when customer joined
- **Quantity**: Number of units ordered
- **Revenue**: Original revenue value (rounded to 2 decimals)
- **CleanedRevenue**: Revenue with null handling (Price × Quantity fallback)
- **COGS**: Cost of goods sold (rounded to 2 decimals)
- **Profit**: Calculated profit (Revenue - COGS)
- **ProductName**: Name of the product
- **ProductCategory**: Product category classification
- **Price**: Product price (rounded to 2 decimals)
- **Base_Cost**: Product base cost (rounded to 2 decimals)

## Data Quality Considerations

**Null Revenue Handling**: The `CleanedRevenue` field uses a CASE statement to calculate revenue from price and quantity when the original revenue value is null, ensuring complete data for analysis.

**Customer Filtering**: The WHERE clause filters out orders with null CustomerID values to maintain data integrity.

**Precision**: All monetary values are rounded to 2 decimal places for consistency and readability.

## Use Cases

This dataset supports various analytical use cases:

- Revenue and profit trend analysis over time
- Customer lifetime value calculations
- Product performance comparisons
- Regional sales analysis
- Weekly sales forecasting
- Customer cohort analysis based on join dates
- Category-level profitability assessment

## Technical Details

**Database**: SQL Server (uses DATEADD and DATEDIFF functions)

**Join Strategy**: LEFT JOINs ensure all orders are retained even if customer or product data is missing

**Performance**: Consider indexing CustomerID, ProductID, and OrderDate fields for optimal query performance on large datasets

## Future Enhancements

Potential improvements to consider:

- Add date range parameters for flexible time period analysis
- Include additional KPIs such as average order value or customer acquisition cost
- Implement data validation checks for anomalies
- Add year-over-year comparison metrics
- Create aggregated summary tables for dashboard consumption

## License

This project is available for educational and commercial use.

## Contact

For questions or collaboration opportunities, please open an issue in this repository.
