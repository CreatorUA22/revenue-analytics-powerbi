# Revenue Report — DAX Measures

This file consolidates the DAX measures created for the Revenue Report.

**Status labels**
- `[Підтверджено]` — exact formula recovered from the project history/current conversation.
- `[Неперевірено]` — exact historical text was not fully recoverable; formula reconstructed from confirmed logic and should be checked against the current semantic model.

## 01 Base

### Revenue
`[Підтверджено]`
```DAX
Revenue =
SUM(Fact_Revenue[Revenue])
```

### Active Clients
`[Підтверджено]`
```DAX
Active Clients =
COUNTROWS(
    FILTER(
        VALUES(Dim_Customer_Group[Customer_Group_ID]),
        CALCULATE([Revenue]) <> 0
    )
)
```

### Avg Revenue per Active Client
`[Підтверджено]`
```DAX
Avg Revenue per Active Client =
DIVIDE([Revenue], [Active Clients])
```

## 02 Time Intelligence

### Revenue PY
`[Підтверджено]`
```DAX
Revenue PY =
CALCULATE(
    [Revenue],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

### Revenue YoY Change
`[Підтверджено]`
```DAX
Revenue YoY Change =
[Revenue] - [Revenue PY]
```

### Revenue YoY %
`[Підтверджено — current version]`
```DAX
Revenue YoY % =
VAR PreviousYearRevenue = [Revenue PY]
RETURN
    IF(
        ISBLANK(PreviousYearRevenue) || PreviousYearRevenue = 0,
        BLANK(),
        DIVIDE([Revenue] - PreviousYearRevenue, PreviousYearRevenue)
    )
```

### Revenue YTD
`[Підтверджено]`
```DAX
Revenue YTD =
CALCULATE(
    [Revenue],
    DATESYTD(Dim_Date[Date])
)
```

### Revenue YTD PY
`[Неперевірено]`
```DAX
Revenue YTD PY =
CALCULATE(
    [Revenue YTD],
    SAMEPERIODLASTYEAR(Dim_Date[Date])
)
```

### Revenue PM
`[Підтверджено]`
```DAX
Revenue PM =
CALCULATE(
    [Revenue],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

### Revenue MoM Change
`[Підтверджено]`
```DAX
Revenue MoM Change =
[Revenue] - [Revenue PM]
```

### Revenue MoM %
`[Підтверджено]`
```DAX
Revenue MoM % =
DIVIDE(
    [Revenue] - [Revenue PM],
    [Revenue PM]
)
```

### Rolling 12M Revenue
`[Підтверджено]`
```DAX
Rolling 12M Revenue =
CALCULATE(
    [Revenue],
    DATESINPERIOD(
        Dim_Date[Date],
        MAX(Dim_Date[Date]),
        -12,
        MONTH
    )
)
```

## Latest Month Measures

### Latest Month Revenue
`[Підтверджено]`
```DAX
Latest Month Revenue =
VAR LatestMonth =
    CALCULATE(
        MAX(Fact_Revenue[Invoice_Month]),
        ALLSELECTED(Dim_Date),
        REMOVEFILTERS(Dim_Customer),
        REMOVEFILTERS(Dim_Customer_Group)
    )
RETURN
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        Dim_Date[Date] = LatestMonth
    )
```

### Latest Month MoM %
`[Підтверджено]`
```DAX
Latest Month MoM % =
VAR LatestMonth =
    CALCULATE(
        MAX(Fact_Revenue[Invoice_Month]),
        ALLSELECTED(Dim_Date),
        REMOVEFILTERS(Dim_Customer),
        REMOVEFILTERS(Dim_Customer_Group)
    )
VAR PrevMonthDate = EDATE(LatestMonth, -1)
VAR CurrentRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        Dim_Date[Date] = LatestMonth
    )
VAR PreviousRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        Dim_Date[Date] = PrevMonthDate
    )
RETURN
    DIVIDE(CurrentRevenue - PreviousRevenue, PreviousRevenue)
```

### Latest Month YoY %
`[Неперевірено]`
```DAX
Latest Month YoY % =
VAR LatestMonth =
    CALCULATE(
        MAX(Fact_Revenue[Invoice_Month]),
        ALLSELECTED(Dim_Date),
        REMOVEFILTERS(Dim_Customer),
        REMOVEFILTERS(Dim_Customer_Group)
    )
VAR PrevYearMonthDate = EDATE(LatestMonth, -12)
VAR CurrentRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        Dim_Date[Date] = LatestMonth
    )
VAR PreviousYearRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        Dim_Date[Date] = PrevYearMonthDate
    )
RETURN
    DIVIDE(CurrentRevenue - PreviousYearRevenue, PreviousYearRevenue)
```

## 03 Customers

### Customer Share %
`[Підтверджено]`
```DAX
Customer Share % =
DIVIDE(
    [Revenue],
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Customer_Group),
        REMOVEFILTERS(Dim_Customer)
    )
)
```

### Top 5 Revenue
`[Підтверджено]`
```DAX
Top 5 Revenue =
SUMX(
    TOPN(
        5,
        CALCULATETABLE(
            VALUES(Dim_Customer_Group[Customer_Group_ID]),
            REMOVEFILTERS(Dim_Customer_Group)
        ),
        CALCULATE([Revenue]),
        DESC
    ),
    CALCULATE([Revenue])
)
```

### Top 5 Share %
`[Підтверджено]`
```DAX
Top 5 Share % =
DIVIDE(
    [Top 5 Revenue],
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Customer_Group),
        REMOVEFILTERS(Dim_Customer)
    )
)
```

### Selected Months
`[Підтверджено]`
```DAX
Selected Months =
DISTINCTCOUNT(Dim_Date[Year_Month])
```

### Avg Monthly Revenue
`[Підтверджено]`
```DAX
Avg Monthly Revenue =
DIVIDE([Revenue], [Selected Months])
```

### Absolute YoY Revenue Change
`[Підтверджено]`
```DAX
Absolute YoY Revenue Change =
ABS([Revenue YoY Change])
```

## Trend Measures

These measures use the disconnected `_Trend_Axis` table so a single selected month still shows a 12-month history ending in the selected period.

### Trend Revenue
`[Неперевірено]`
```DAX
Trend Revenue =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        CALCULATE(
            [Revenue],
            REMOVEFILTERS(Dim_Date),
            TREATAS({AxisMonth}, Dim_Date[Date])
        )
    )
```

### Trend Revenue PY
`[Неперевірено]`
```DAX
Trend Revenue PY =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
VAR PYMonth =
    EDATE(AxisMonth, -12)
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        CALCULATE(
            [Revenue],
            REMOVEFILTERS(Dim_Date),
            TREATAS({PYMonth}, Dim_Date[Date])
        )
    )
```

### Trend Rolling 12M Revenue
`[Неперевірено]`
```DAX
Trend Rolling 12M Revenue =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        CALCULATE(
            [Rolling 12M Revenue],
            REMOVEFILTERS(Dim_Date),
            TREATAS({AxisMonth}, Dim_Date[Date])
        )
    )
```

### Trend MoM %
`[Неперевірено]`
```DAX
Trend MoM % =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
VAR PrevMonth =
    EDATE(AxisMonth, -1)
VAR CurrentRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        TREATAS({AxisMonth}, Dim_Date[Date])
    )
VAR PrevRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        TREATAS({PrevMonth}, Dim_Date[Date])
    )
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        DIVIDE(CurrentRevenue - PrevRevenue, PrevRevenue)
    )
```

### Trend YoY %
`[Неперевірено]`
```DAX
Trend YoY % =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
VAR PYMonth =
    EDATE(AxisMonth, -12)
VAR CurrentRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        TREATAS({AxisMonth}, Dim_Date[Date])
    )
VAR PYRevenue =
    CALCULATE(
        [Revenue],
        REMOVEFILTERS(Dim_Date),
        TREATAS({PYMonth}, Dim_Date[Date])
    )
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        DIVIDE(CurrentRevenue - PYRevenue, PYRevenue)
    )
```

### Trend Revenue YTD
`[Неперевірено]`
```DAX
Trend Revenue YTD =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
VAR YearStart =
    DATE(YEAR(AxisMonth), 1, 1)
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        CALCULATE(
            [Revenue],
            REMOVEFILTERS(Dim_Date),
            DATESBETWEEN(Dim_Date[Date], YearStart, AxisMonth)
        )
    )
```

### Trend Revenue YTD PY
`[Неперевірено]`
```DAX
Trend Revenue YTD PY =
VAR LastSelectedDate =
    MAXX(ALLSELECTED(Dim_Date[Date]), Dim_Date[Date])
VAR LastSelectedMonth =
    DATE(YEAR(LastSelectedDate), MONTH(LastSelectedDate), 1)
VAR AxisMonth =
    SELECTEDVALUE(_Trend_Axis[Month_Start])
VAR FirstMonth =
    EDATE(LastSelectedMonth, -11)
VAR PYMonth =
    EDATE(AxisMonth, -12)
VAR PYYearStart =
    DATE(YEAR(PYMonth), 1, 1)
RETURN
    IF(
        AxisMonth >= FirstMonth && AxisMonth <= LastSelectedMonth,
        CALCULATE(
            [Revenue],
            REMOVEFILTERS(Dim_Date),
            DATESBETWEEN(Dim_Date[Date], PYYearStart, PYMonth)
        )
    )
```

## Waterfall

### Waterfall Revenue Contribution
`[Неперевірено]`
```DAX
Waterfall Revenue Contribution =
VAR CurrentHelperID =
    SELECTEDVALUE(Dim_Waterfall_Customer[Customer_Group_ID])

VAR CustomerChangesBase =
    ADDCOLUMNS(
        ALL(Dim_Customer_Group[Customer_Group_ID]),
        "__Change",
            VAR CurrentID = Dim_Customer_Group[Customer_Group_ID]
            RETURN
                CALCULATE(
                    [Revenue YoY Change],
                    REMOVEFILTERS(Dim_Customer_Group),
                    REMOVEFILTERS(Dim_Customer),
                    TREATAS(
                        {CurrentID},
                        Dim_Customer_Group[Customer_Group_ID]
                    )
                )
    )

VAR CustomerChanges =
    ADDCOLUMNS(
        CustomerChangesBase,
        "__AbsChange",
        ABS([__Change])
    )

VAR Top5Customers =
    TOPN(
        5,
        FILTER(
            CustomerChanges,
            NOT ISBLANK([__Change]) && [__Change] <> 0
        ),
        [__AbsChange], DESC,
        Dim_Customer_Group[Customer_Group_ID], ASC
    )

VAR TotalChange =
    CALCULATE(
        [Revenue YoY Change],
        REMOVEFILTERS(Dim_Customer_Group),
        REMOVEFILTERS(Dim_Customer)
    )

VAR Top5Change =
    SUMX(Top5Customers, [__Change])

VAR OthersChange =
    TotalChange - Top5Change

VAR CurrentCustomerChange =
    CALCULATE(
        [Revenue YoY Change],
        REMOVEFILTERS(Dim_Customer_Group),
        REMOVEFILTERS(Dim_Customer),
        TREATAS(
            {CurrentHelperID},
            Dim_Customer_Group[Customer_Group_ID]
        )
    )

VAR IsTop5 =
    COUNTROWS(
        FILTER(
            Top5Customers,
            Dim_Customer_Group[Customer_Group_ID] = CurrentHelperID
        )
    ) > 0

RETURN
    SWITCH(
        TRUE(),
        CurrentHelperID = -1, OthersChange,
        IsTop5, CurrentCustomerChange,
        BLANK()
    )
```

## Supporting DAX Table

### _Trend_Axis
`[Підтверджено]`
```DAX
_Trend_Axis =
VAR StartMonth = DATE(2023, 1, 1)
VAR EndMonth = DATE(2030, 12, 1)
RETURN
    SELECTCOLUMNS(
        FILTER(
            CALENDAR(StartMonth, EndMonth),
            DAY([Date]) = 1
        ),
        "Month_Start", [Date],
        "Year_Month", FORMAT([Date], "yyyy-MM"),
        "Year_Month_Sort", YEAR([Date]) * 100 + MONTH([Date])
    )
```

`_Trend_Axis` is intentionally disconnected from the main model.

## Measure inventory

1. Revenue
2. Active Clients
3. Avg Revenue per Active Client
4. Revenue PY
5. Revenue YoY Change
6. Revenue YoY %
7. Revenue YTD
8. Revenue YTD PY
9. Revenue PM
10. Revenue MoM Change
11. Revenue MoM %
12. Rolling 12M Revenue
13. Latest Month Revenue
14. Latest Month MoM %
15. Latest Month YoY %
16. Customer Share %
17. Top 5 Revenue
18. Top 5 Share %
19. Selected Months
20. Avg Monthly Revenue
21. Absolute YoY Revenue Change
22. Trend Revenue
23. Trend Revenue PY
24. Trend Rolling 12M Revenue
25. Trend MoM %
26. Trend YoY %
27. Trend Revenue YTD
28. Trend Revenue YTD PY
29. Waterfall Revenue Contribution
