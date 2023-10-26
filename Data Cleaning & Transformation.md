# **Data Cleaning & Transformation**

<p align="center">
  <b><span style="font-size: 144px;">**Data Cleaning AND Transformation**</span></b>
</p>

## Getting the tables ready for analysis and the dashboard

### 1. DimDate Table

**Steps Taken:**

- Removed irrelevant columns to my analysis.
- Used the `LEFT` function to get the short form of month names.
- Used the `WHERE` clause to filter dates from 2018.

```sql
SELECT
  [DateKey],
  [FullDateAlternateKey] AS Date,
  [EnglishDayNameOfWeek] AS Day,
  [WeekNumberOfYear] AS WeekNum,
  [EnglishMonthName] AS Month,
  LEFT(EnglishMonthName, 3) AS MonthShort,
  [MonthNumberOfYear] AS MonthNum,
  [CalendarQuarter] AS Quarter,
  [CalendarYear] AS Year
FROM
  [AdventureWorksDW2019].[dbo].[DimDate]
WHERE
CalendarYear >= 2018;
```

**The Output:**
| DateKey  | Date       | Day       | WeekNum | Month   | MonthShort | MonthNum | Quarter | Year |
|----------|------------|-----------|---------|---------|------------|----------|---------|------|
| 20180101 | 01/01/2018 | Monday    | 1       | January | Jan        | 1        | 1       | 2018 |
| 20180102 | 02/01/2018 | Tuesday   | 1       | January | Jan        | 1        | 1       | 2018 |
| 20180103 | 03/01/2018 | Wednesday | 1       | January | Jan        | 1        | 1       | 2018 |
| 20180104 | 04/01/2018 | Thursday  | 1       | January | Jan        | 1        | 1       | 2018 |
| 20180105 | 05/01/2018 | Friday    | 1       | January | Jan        | 1        | 1       | 2018 |
- This is not total output (1461 rows in total)

### 2. DimCustomer Table
**Steps Taken:**

- Removed irrelevant columns to my analysis.
- Concatenated the FullName and LastName columns to get Full Name
- Used the DATEDIFF` clause to get current ages.
- Changed M and F to Male and female usin the CASE function
- Joined to the Geography table to add the City column

```sql
SELECT 
  c.CustomerKey AS[CustomerKey], 
  -- Combining First and Last Name columns
  c.FirstName + ' ' + c.LastName AS [Full Name], 
  c.BirthDate AS [BirthDate], 
  DATEDIFF(
    Year, 
    [BirthDate], 
    GETDATE()
  ) AS Age, 
  CASE WHEN Gender = 'M' THEN 'Male' ELSE 'Female' END AS Gender, 
  c.YearlyIncome AS [YearlyIncome], 
  c.DateFirstPurchase AS [DateFirstPurchase], 
  c.CommuteDistance AS [Commute Distance], 
  -- Joined in Cuustomer City from Geography Table 
  g.City 
FROM 
  dbo.DimCustomer AS c 
  JOIN dbo.DimGeography AS g ON g.GeographyKey = c.GeographyKey
```

**The Output:**
| CustomerKey | Full Name        | BirthDate  | Age | Gender | YearlyIncome | DateFirstPurchase | Commute Distance | City    |
|-------------|------------------|------------|-----|--------|--------------|-------------------|------------------|---------|
| 28866       | Aaron Adams      | 05/08/1979 | 44  | Male   | 50000        | 28/04/2020        | 5-10 Miles       | Downey  |
| 20285       | Aaron Alexander  | 20/04/1982 | 41  | Male   | 40000        | 13/12/2020        | 5-10 Miles       | Kirkland|
| 20075       | Aaron Allen      | 15/11/1958 | 65  | Male   | 10000        | 02/12/2018        | 1-2 Miles        | Sooke   |

- This is not total output (18,484 rows in total)

### 3. DimProducts Table
**Steps Taken:**

- Removed irrelevant columns to my analysis.
- Left joined ProductSubCategory & ProductCategory tables (needed a column each from there)
- Null values in Status column changed to 'Outdated'

```sql
SELECT 
  [ProductKey], 
  p.ProductAlternateKey AS [ProductItemCode], 
  p.EnglishProductName AS [ProductName], 
  ps.EnglishProductSubcategoryName AS [ProductSubCategory], 
  pc.EnglishProductCategoryName AS [Product Category], 
  p.Color AS [Color], 
  ISNULL(p.Status, 'Outdated') AS [Product Status] 
FROM 
  [dbo].[DimProduct] AS p 
  LEFT JOIN [dbo].[DimProductSubcategory] AS ps ON ps.ProductSubcategoryKey = p.ProductKey 
  LEFT JOIN [dbo].[DimProductCategory] AS pc ON pc.ProductCategoryKey = ps.ProductCategoryKey
```

**The Output:**
| ProductKey | ProductItemCode | ProductName          | ProductSubCategory | Product Category | Color | Product Status |
|------------|-----------------|-----------------------|--------------------|------------------|-------|----------------|
| 1          | AR-5381         | Adjustable Race      | Mountain Bikes     | Bikes            | NA    | Current        |
| 2          | BA-8327         | Bearing Ball         | Road Bikes         | Bikes            | NA    | Current        |
| 3          | BE-2349         | BB Ball Bearing      | Touring Bikes      | Bikes            | NA    | Current        |

- This is not total output (606 rows in total)

### 4. FactInternet Sales Table
**Steps Taken:**

- Removed irrelevant columns to my analysis.
- Joined with DimCustomers Table to get the Full Name of each customer instead of customer key
- Null values in Status column changed to 'Outdated'

**The Output:**
| ProductKey | OrderDateKey | CustomerKey | SalesOrderNumber | TotalProductCost | Full Name    | SalesAmount |
|------------|--------------|------------|------------------|-------------------|-------------|-------------|
| 529        | 20200428     | 28866      | SO56918          | 1.4923            | Aaron Adams | 3.99        |
| 539        | 20200428     | 28866      | SO56918          | 9.3463            | Aaron Adams | 24.99       |
| 214        | 20200428     | 28866      | SO56918          | 13.0863           | Aaron Adams | 34.99       |

```sql
-- Cleansed FACT_InternetSales Table --
SELECT 
  [ProductKey], 
  [OrderDateKey],  
  s.CustomerKey AS [CustomerKey], 
  [SalesOrderNumber], 
  [TotalProductCost], 
  c.FirstName + ' ' + c.LastName AS [Full Name], 
  [SalesAmount] --  [TaxAmt], 
FROM 
  [dbo].[FactInternetSales] AS s 
  JOIN dbo.DimCustomer AS c ON c.CustomerKey = s.CustomerKey 
WHERE 
  --I filter for 2018 to 2021, since I want to query those years only
  LEFT (OrderDateKey, 4) >= 2018 
ORDER BY 
  OrderDateKey ASC
```

| ProductKey | OrderDateKey | CustomerKey | SalesOrderNumber | TotalProductCost | Full Name    | SalesAmount |
|------------|--------------|------------|------------------|-------------------|-------------|-------------|
| 529        | 20200428     | 28866      | SO56918          | 1.4923            | Aaron Adams | 3.99        |
| 539        | 20200428     | 28866      | SO56918          | 9.3463            | Aaron Adams | 24.99       |
| 214        | 20200428     | 28866      | SO56918          | 13.0863           | Aaron Adams | 34.99       |

- This is not total output (60,384 rows in total)
