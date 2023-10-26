# **Data Exploration**

## To answer more questions for better analysis

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

'''sql
SELECT 
  CASE 
    WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
    WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
    WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
    WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
    ELSE 'Above 100k' 
  END AS AmountRange,
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
'''

**The Output:**
| AmountRange  | No_of_customers |
|-------------|------------|
| 25k-50k       | 5704       |
| 50-75k     | 5476       |
| 0-25K     | 2922       |
| 75k-100k    | 2755       |
| Above 100k  | 1627       |

### 5. Customers that earn between 25k to 75k are the most populous, but do they spend the most?
**Steps Taken:**

- Used CASE statements to categorize the annual incomes
- Used COUNT to get the number of each occurence

'''sql
SELECT 
  AmountRange,
  CAST(SUM(TotalSalesAmount) AS DECIMAL(18, 2)) AS Sales
FROM (
  SELECT 
    CASE 
      WHEN [YearlyIncome] >= 0 AND [YearlyIncome] <= 25000 THEN '0-25k' 
      WHEN [YearlyIncome] >= 25001 AND [YearlyIncome] <= 50000 THEN '25k-50k' 
      WHEN [YearlyIncome] >= 50001 AND [YearlyIncome] <= 75000 THEN '50-75k' 
      WHEN [YearlyIncome] >= 75001 AND [YearlyIncome] <= 100000 THEN '75k-100k' 
      ELSE 'Above 100k' 
    END AS AmountRange,
    SUM(s.SalesAmount) AS TotalSalesAmount
  FROM 
    [AdventureWorksDW2019].[dbo].[DimCustomer] AS c
  JOIN [dbo].[FactInternetSales] AS s
  ON s.CustomerKey = c.CustomerKey
  GROUP BY [YearlyIncome]
) AS Subquery
GROUP BY AmountRange
ORDER BY Sales DESC;
'''


**The Output:**
| AmountRange  | No_of_customers |
|-------------|------------|
| 50-75k       | 8714787.44   |
| 25k-50k     | 7954391.47   |
| 75k-100k     | 5531537.77   |
| Above 100k    | 3750950.96   |
| 0-25K  | 3407009.58   |
