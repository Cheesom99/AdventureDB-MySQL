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


### 4. Categorize the customers according to their earnings and the number of customers in each category?
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

### 5. What income category spends the most?
**Steps Taken:**

- In the subquery, the total sales amount for each customer key was calculated (added).
- In the outerquery, I selected the Income Category and added the total sales amount for each, casted it to 2 dp also.
- Ordered by Highest Sales Category

```sql
SELECT 
  IncomeCategory,
  CAST(SUM(TotalSalesAmount) AS DECIMAL(18, 2)) AS Sales
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
| 50-75k       | 8714787.44   |
| 25k-50k     | 7954391.47   |
| 75k-100k     | 5531537.77   |
| Above 100k    | 3750950.96   |
| 0-25K  | 3407009.58   |


### 6. What income category does the most transactions?
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

### 7. What income category spend the most per transactions?
```sql

```
