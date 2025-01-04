### **Data Cleaning and Quality Checks**  
```sql
-- Check for missing values in all tables
SELECT 
    'Customers' AS TableName, 
    COUNT(*) AS TotalRows, 
    SUM(CASE WHEN Segment IS NULL THEN 1 ELSE 0 END) AS Missing_Segment,
    SUM(CASE WHEN Country IS NULL THEN 1 ELSE 0 END) AS Missing_Country,
    SUM(CASE WHEN City IS NULL THEN 1 ELSE 0 END) AS Missing_City,
    SUM(CASE WHEN State IS NULL THEN 1 ELSE 0 END) AS Missing_State,
    SUM(CASE WHEN Region IS NULL THEN 1 ELSE 0 END) AS Missing_Region
FROM Customers;

-- Repeat similar queries for Orders, Products, and Sales tables to check for NULL values

-- Verify data types for correctness
SELECT 
    COLUMN_NAME, 
    DATA_TYPE 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_NAME = 'Customers';

-- Check for duplicate records in the Customers table
SELECT 
    CustomerID, 
    COUNT(*) AS DuplicateCount 
FROM Customers 
GROUP BY CustomerID 
HAVING COUNT(*) > 1;

-- Check for duplicate OrderIDs in the Orders table
SELECT 
    OrderID, 
    COUNT(*) AS DuplicateCount 
FROM Orders 
GROUP BY OrderID 
HAVING COUNT(*) > 1;

-- Check for unrealistic or outlier values in sales data
SELECT 
    SalesID, 
    Quantity, 
    Sales, 
    Discount, 
    Profit 
FROM Sales 
WHERE Quantity < 0 
   OR Sales < 0 
   OR Discount < 0 
   OR Discount > 1 
   OR Profit < 0;

-- Check for mismatched relationships between tables
SELECT 
    o.OrderID, 
    o.CustomerID 
FROM Orders o
LEFT JOIN Customers c ON o.CustomerID = c.CustomerID 
WHERE c.CustomerID IS NULL;
```

### **Basic Business Analysis**  

#### Top Products by Sales and Profit
```sql
SELECT 
    p.ProductName, 
    p.Category, 
    SUM(s.Sales) AS TotalSales, 
    SUM(s.Profit) AS TotalProfit 
FROM Sales s
JOIN Products p ON s.ProductID = p.ProductID
GROUP BY p.ProductName, p.Category
ORDER BY TotalSales DESC
LIMIT 10;
```

#### Regional Sales and Profit Analysis
```sql
SELECT 
    c.Region, 
    SUM(s.Sales) AS TotalSales, 
    SUM(s.Profit) AS TotalProfit 
FROM Sales s
JOIN Orders o ON s.OrderID = o.OrderID
JOIN Customers c ON o.CustomerID = c.CustomerID
GROUP BY c.Region
ORDER BY TotalSales DESC;
```

#### Peak Sales Days and Shipping Efficiency
```sql
SELECT 
    DAYNAME(o.OrderDate) AS DayOfWeek, 
    COUNT(o.OrderID) AS TotalOrders, 
    SUM(s.Sales) AS TotalSales 
FROM Orders o
JOIN Sales s ON o.OrderID = s.OrderID
GROUP BY DAYNAME(o.OrderDate)
ORDER BY TotalSales DESC;

-- Analyze shipping delays
SELECT 
    AVG(DATEDIFF(o.ShipDate, o.OrderDate)) AS AvgShippingDelay, 
    MAX(DATEDIFF(o.ShipDate, o.OrderDate)) AS MaxShippingDelay 
FROM Orders o;
```