# **Data Exploration**

## To answer more questions for better understanding of customer purchases

### 1. What is the number of transations since 2018?

```sql
SELECT 
  COUNT ([ProductKey]) AS No_of_transactions
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE
  LEFT (OrderDateKey, 4) >= 2018
```

**The Output:**
| No_of_transactions  | 
|----------|
| 60384 | 

### 2. What is the number of customers since 2018?

```sql
SELECT  
  COUNT (DISTINCT [CustomerKey]) AS No_of_customers
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE
  LEFT (OrderDateKey, 4) >= 2018
```

**The Output:**
| No_of_customers  | 
|----------|
| 18483 | 



### 3. Number of Customers for each year since 2018?

**Steps Taken:**

- Made sure I used DISTINCT while using COUNT to get the unique number of customers
- Grouped Year column to get each year together.

```sql
SELECT 
  LEFT(OrderDateKey, 4) AS CalendarYear,
  COUNT (DISTINCT [CustomerKey]) AS No_of_customers
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE
  LEFT (OrderDateKey, 4) >= 2018
GROUP BY LEFT (OrderDateKey, 4)
ORDER BY LEFT (OrderDateKey, 4)
```

**The Output:**
| CalendarYear  | No_of_customers       | 
|----------|------------|
| 2018 | 2216 |  
| 2019 | 3255 |  
| 2020 | 17429 |  
| 2021 | 834 | 

- 2020 has a high number of internet sales because it was the COVID year and isolation
- 2021 has a low value because the data stopped at January


### 4. List the countries with stores?
**Steps Taken:**

- Used DISTINCT function to get the countries
- Renamed the column

```sql
SELECT 
  DISTINCT [EnglishCountryRegionName] AS Country
FROM 
  [AdventureWorksDW2019].[dbo].[DimGeography]
```

**The Output:**
| Countries |
|-----------|
| Australia |
| Canada    |
| France    |
| Germany   |
| United Kingdom|
| United States|


### 5. List the countries with the most number of customers since 2018?
**Steps Taken:**

- Used DISTINCT function to get the countries
- Used COUNT and DISTINCT function to get the number of unique customers
- Grouped by the Country and Ordered by Number of customers

```sql
SELECT 
  DISTINCT [SalesTerritoryCountry] AS Country, 
  COUNT(DISTINCT(s.[CustomerKey])) AS No_of_customers 
FROM 
  [AdventureWorksDW2019].[dbo].[DimSalesTerritory] AS st 
  JOIN [dbo].[FactInternetSales] AS s ON st.SalesTerritoryKey = s.SalesTerritoryKey 
WHERE 
  LEFT (s.[OrderDateKey], 4) >= 2018 
GROUP BY 
  [SalesTerritoryCountry] 
ORDER BY 
  No_of_customers DESC;

```

**The Output:**
| Country         | No. of Customers |
| --------------- | ---------------- |
| United States   | 7,819            |
| Australia       | 3,591            |
| United Kingdom  | 1,913            |
| France          | 1,809            |
| Germany         | 1,780            |
| Canada          | 1,571            |



### 6. Categorize the customers according to their earnings and the number of customers in each category?
**Steps Taken:**

- Used CASE statements to categorize the annual incomes
- Used COUNT to get the number of each occurence

```sql
SELECT 
  CASE 
    WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
    WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
    WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
    WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
    ELSE 'Above 100k' 
  END AS IncomeCategory,
  COUNT(*) AS No_of_customers
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] 
GROUP BY 
  CASE 
    WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
    WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
    WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
    WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
    ELSE 'Above 100k' 
  END
ORDER BY No_of_customers DESC;
```

**The Output:**
| IncomeCategory  | No_of_customers |
|-------------|------------|
| 25k-50k       | 5704       |
| 50-75k     | 5476       |
| 0-25K     | 2922       |
| 75k-100k    | 2755       |
| Above 100k  | 1627       |

### 7. What income category spends the most?
**Steps Taken:**

- In the subquery, the total sales amount for each customer key was calculated (added).
- In the outerquery, I selected the Income Category and added the total sales amount for each, rounded it to 2 dp also.
- Ordered by Highest Sales Category

```sql
SELECT 
  IncomeCategory,
  ROUND(SUM(TotalSalesAmount,2)) AS Sales
FROM (
  SELECT 
    CASE 
      WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
      WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
      WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
      WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
      ELSE 'Above 100k' 
    END AS IncomeCategory,
    SUM(s.SalesAmount) AS TotalSalesAmount
  FROM 
    [AdventureWorksDW2019].[dbo].[DimCustomer] AS c
  JOIN [dbo].[FactInternetSales] AS s
  ON s.CustomerKey = c.CustomerKey
  GROUP BY [YearlyIncome]
) AS Subquery
GROUP BY IncomeCategory
ORDER BY Sales DESC;
```

**The Output:**
| IncomeCategory  | Sales |
|-------------|------------|
| 50-75k       | $8,714,787.44   |
| 25k-50k     | $7,954,391.47   |
| 75k-100k     | $5,531,537.77   |
| Above 100k    | $3,750,950.96   |
| 0-25K  | $3,407,009.58   |


### 8. What income category does the most transactions?
**Steps Taken:**

- Join Customer and Internet Sales Table on Customer key
- Used CASE statements to get the Income category
- Count the Product keys for each category by grouping by the categories.

```sql
SELECT 
  CASE 
    WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
    WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
    WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
    WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
    ELSE 'Above 100k' 
  END AS IncomeCategory,
  COUNT(s.ProductKey) AS No_of_transactions
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] AS c
JOIN [dbo].[FactInternetSales] AS s
ON s.CustomerKey = c.CustomerKey
GROUP BY 
  CASE 
    WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
    WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
    WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
    WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
    ELSE 'Above 100k' 
  END
ORDER BY No_of_transactions DESC;
```

**The Output:**
| IncomeCategory  | No_of_transactions |
|-------------|------------|
| 25k-50k       | 18152  |
| 50-75k     | 17737  |
| 75k-100k     | 9946  |
| 0-25K    | 8480   |
| Above 100k  | 6083  |

**NOTE**
- This can be solved using subqueries just like the previous ones.


### 9. What income category spend the most per transactions?

**Steps Taken:**

- In the subquery, the total sales amount for each customer key was calculated (added).
- Also, the number of transactions were also counted
- In the outerquery, I selected the Income Category and added the total sales amount for each and number of transactions, rounded it to 2 dp also.
- Ordered by Average Order Value

```sql
SELECT 
  IncomeCategory, 
  ROUND(
    SUM(TotalSalesAmount) / SUM(No_of_transactions), 
    2
  ) AS Average_Order_Value 
FROM 
  (
    SELECT 
      CASE WHEN [YearlyIncome] >= 0 
      AND [YearlyIncome] <= 25000 THEN '0-25k' WHEN [YearlyIncome] >= 25001 
      AND [YearlyIncome] <= 50000 THEN '25k-50k' WHEN [YearlyIncome] >= 50001 
      AND [YearlyIncome] <= 75000 THEN '50-75k' WHEN [YearlyIncome] >= 75001 
      AND [YearlyIncome] <= 100000 THEN '75k-100k' ELSE 'Above 100k' END AS IncomeCategory, 
      COUNT(s.ProductKey) AS No_of_transactions, 
      SUM(s.SalesAmount) AS TotalSalesAmount 
    FROM 
      [AdventureWorksDW2019].[dbo].[DimCustomer] AS c 
      JOIN [dbo].[FactInternetSales] AS s ON s.CustomerKey = c.CustomerKey 
    GROUP BY 
      [YearlyIncome]
  ) AS Subquery 
GROUP BY 
  IncomeCategory
ORDER BY 
  Average_Order_Value DESC;
```

**The Output:**
| Income Category | AOV    |
|-----------------|--------|
| Above 100k      | $616.63 |
| 75k-100k        | $556.16 |
| 50-75k          | $491.33 |
| 25k-50k         | $438.21 |
| 0-25k           | $401.77 |



### 10. What country spend the most per transactions?

**Steps Taken:**

- Joined the Internet Sales tbale with the Sales territory table.
- Selected important columns 
- Calculated the AOV in the last column
- Ordered by the AOV

```sql
SELECT 
  st.SalesTerritoryCountry AS Country, 
  ROUND(SUM([SalesAmount]), 2) AS Total_Sales, 
  COUNT(s.ProductKey) AS No_of_transactions, 
  ROUND(SUM([SalesAmount])/ COUNT(s.ProductKey), 2) AS Average_Order_Value  
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales] AS s
JOIN [dbo].[DimSalesTerritory] AS st ON st.SalesTerritoryKey = s.SalesTerritoryKey
GROUP BY 
  st.SalesTerritoryCountry
ORDER BY 
  Average_Order_Value DESC;
```

**The Output:**
| Country         | Total Sales   | No. of Transactions | Average Order Value |
| --------------- | ------------- | ------------------- | ------------------- |
| Australia       | $9,061,000.58 | 13,345              | $678.98             |
| Germany         | $2,894,312.34 | 5,625               | $514.54             |
| United Kingdom  | $3,391,712.21 | 6,906               | $491.13             |
| France          | $2,644,017.71 | 5,558               | $475.71             |
| United States   | $9,389,789.51 | 21,344              | $439.93             |
| Canada          | $1,977,844.86 | 7,620               | $259.56             |
