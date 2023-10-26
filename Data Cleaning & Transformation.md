<p align="center">
  <b><span style="font-size: 48px;">Data Cleaning & Transformation</span></b>
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
