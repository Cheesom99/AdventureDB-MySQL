<p align="center">
  <b><span style="font-size: 24px;">Data Cleaning & Transformation</span></b>
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
