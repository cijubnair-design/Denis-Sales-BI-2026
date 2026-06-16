# Key DAX Measures — Denis Sales BI

All measures built in Power BI Desktop using best practices:
- Explicit measures (no implicit aggregations)
- Variables (VAR/RETURN) for readability
- DIVIDE() for safe division
- Proper use of filter context and CALCULATE

---

## Core KPI Measures

### Total Revenue
```dax
Total Revenue = SUM(Denis_Data[TotalRevenue])
```

### Total Cost
```dax
Total Cost = SUM(Denis_Data[TotalCost])
```

### Total Gross Profit
```dax
Total Gross Profit = [Total Revenue] - [Total Cost]
```

### Total Transactions
```dax
Total Transactions = COUNTROWS(Denis_Data)
```

### Avg Revenue per Transaction
```dax
Avg Revenue = DIVIDE([Total Revenue], [Total Transactions])
```

---

## Time Intelligence Measures

### YoY% (Year-over-Year Growth)
```dax
YoY% = 
VAR CurrentYearProfit = [Total Gross Profit]
VAR PreviousYearProfit = 
    CALCULATE(
        [Total Gross Profit],
        SAMEPERIODLASTYEAR(DateMaster[Date])
    )
RETURN
    DIVIDE(
        CurrentYearProfit - PreviousYearProfit,
        PreviousYearProfit
    )
```

### Profit Growth % by Product
```dax
%Growth = 
VAR CurrentProfit = [Total Gross Profit]
VAR BaselineProfit = 
    CALCULATE(
        [Total Gross Profit],
        ALLEXCEPT(Denis_Data, Denis_Data[ProductName])
    )
RETURN
    DIVIDE(CurrentProfit, BaselineProfit)
```

### Baseline Profit (Yearly Reference Line)
```dax
Baseline Profit (Yearly) = 
CALCULATE(
    [Total Gross Profit],
    ALL(DateMaster[MonthNo])
)
```

---

## CALCULATE Patterns

### Profit for General Category Only
```dax
General Category Profit = 
CALCULATE(
    [Total Gross Profit],
    Denis_Data[Category] = "General"
)
```

### Profit with Date Range Filter
```dax
Profit 2016-2017 = 
CALCULATE(
    [Total Gross Profit],
    DATESBETWEEN(
        DateMaster[Date],
        DATE(2016, 1, 1),
        DATE(2017, 12, 31)
    )
)
```

### Profit % of Total (Share of Wallet)
```dax
Profit % of Total = 
DIVIDE(
    [Total Gross Profit],
    CALCULATE([Total Gross Profit], ALL(Denis_Data))
)
```

---

## Conditional Formatting Measures

### Growth Direction (for icon sets)
```dax
Growth Direction = 
SWITCH(
    TRUE(),
    [YoY%] > 0.10, 1,    -- Strong growth (green arrow up)
    [YoY%] > 0, 0,        -- Moderate growth (yellow arrow)
    -1                     -- Decline (red arrow down)
)
```

---

## Row Level Security (Dynamic)

### Location Filter (applied to Security Role)
```dax
// DAX filter expression in RLS role definition:
[Location_Access] = "All" ||
[Location_Access] = 
    LOOKUPVALUE(
        Access_Table[Location_Access],
        Access_Table[Username], USERPRINCIPALNAME()
    )
```

**How it works:**
- Users with `Location_Access = "All"` (e.g., managers) see all regions
- Users mapped to a specific location (e.g., "Germany") only see that region's data
- USERPRINCIPALNAME() captures the logged-in user's email at runtime
- Access_Table.csv maps email → location, loaded as a separate query

---

## Design Principles Used

1. **Base measures first** — Total Revenue, Total Cost, Total Gross Profit defined once, reused everywhere
2. **VAR/RETURN pattern** — all multi-step calculations use variables for readability and performance
3. **DIVIDE() always** — never use `/` directly; DIVIDE handles zero denominators gracefully
4. **No implicit measures** — all aggregations are explicit named measures in the model
5. **Measure table** — all measures housed in a dedicated hidden "Measures" table for clean model organization
