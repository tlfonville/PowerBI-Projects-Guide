# DAX Code Library

*Reusable DAX measures and calculations for Power BI development*

---

## Table of Contents
- [Agent Performance Measures](#agent-performance-measures)
- [Time Intelligence](#time-intelligence)
- [Utility Functions](#utility-functions)
- [Troubleshooting Snippets](#troubleshooting-snippets)

---

## Agent Performance Measures

### APS (Agent Performance Score)
**Purpose**: Calculate weighted agent performance score  
**Date Added**: [Date]  
**Last Modified**: [Date]  
**Dependencies**: Busy%, CPLH, AHT measures must exist

```dax
APS = 
VAR BusyPercentile = 
    RANKX(
        ALL(Agents[AgentID]), 
        [Busy%], 
        , 
        DESC, 
        Dense
    ) / COUNTROWS(ALL(Agents[AgentID]))

VAR CPLHPercentile = 
    RANKX(
        ALL(Agents[AgentID]), 
        [CPLH], 
        , 
        DESC, 
        Dense
    ) / COUNTROWS(ALL(Agents[AgentID]))

VAR AHTPercentile = 
    RANKX(
        ALL(Agents[AgentID]), 
        [AHT], 
        , 
        ASC,  -- Lower AHT is better
        Dense
    ) / COUNTROWS(ALL(Agents[AgentID]))

RETURN 
    (BusyPercentile * 0.40) + 
    (CPLHPercentile * 0.35) + 
    (AHTPercentile * 0.25)
```

**Notes**: 
- Uses Dense ranking to avoid gaps in percentile calculations
- AHT ranked ascending because lower handle time = better performance
- Percentile ranges from 0 to 1

---

### Busy%
**Purpose**: Calculate percentage of logged time spent on calls  
**Date Added**: [Date]  

```dax
Busy% = 
DIVIDE(
    SUM(AgentActivity[BusySeconds]),
    SUM(AgentActivity[TotalLoggedSeconds]),
    0
)
```

**Notes**: DIVIDE function handles division by zero gracefully

---

### CPLH (Calls per Logged Hour)
**Purpose**: Calculate calls handled per hour of logged time  
**Date Added**: [Date]  

```dax
CPLH = 
DIVIDE(
    SUM(CallData[HandledCalls]),
    DIVIDE(SUM(AgentActivity[TotalLoggedSeconds]), 3600),
    0
)
```

**Notes**: Converts seconds to hours by dividing by 3600

---

### AHT (Average Handle Time)
**Purpose**: Average handle time for answered inbound calls in seconds  
**Date Added**: [Date]  

```dax
AHT = 
AVERAGE(CallData[HandleTimeSeconds])
```

**Filter Context**: Should be filtered to answered inbound calls only

---

## Time Intelligence

### Previous Period Comparison
**Purpose**: Compare current vs previous period  
**Date Added**: [Date]  

```dax
[Metric] Previous Period = 
CALCULATE(
    [Your Metric],
    PREVIOUSMONTH(DateTable[Date])
)

[Metric] Change = [Your Metric] - [Metric Previous Period]

[Metric] % Change = 
DIVIDE(
    [Metric Change],
    [Metric Previous Period],
    BLANK()
)
```

---

### Year-to-Date
**Purpose**: Year-to-date calculation template  
**Date Added**: [Date]  

```dax
[Metric] YTD = 
CALCULATE(
    [Your Metric],
    DATESYTD(DateTable[Date])
)
```

---

## Utility Functions

### Safe Division
**Purpose**: Division that handles zeros gracefully  
**Date Added**: [Date]  

```dax
Safe Ratio = 
VAR Numerator = [Your Numerator]
VAR Denominator = [Your Denominator]
RETURN
    IF(
        Denominator = 0 || ISBLANK(Denominator),
        BLANK(),
        Numerator / Denominator
    )
```

---

### Conditional Formatting Helper
**Purpose**: Create traffic light colors based on thresholds  
**Date Added**: [Date]  

```dax
Performance Color = 
VAR Score = [Your Score]
RETURN
    SWITCH(
        TRUE(),
        Score >= 0.8, "Green",
        Score >= 0.6, "Yellow",
        Score < 0.6, "Red",
        "Gray"
    )
```

---

### Count Non-Blank
**Purpose**: Count non-blank values in a column  
**Date Added**: [Date]  

```dax
Non Blank Count = 
SUMX(
    VALUES(YourTable[YourColumn]),
    IF(ISBLANK(YourTable[YourColumn]), 0, 1)
)
```

---

## Troubleshooting Snippets

### Debug Measure Values
**Purpose**: See what values are being calculated  
**Date Added**: [Date]  

```dax
Debug APS = 
VAR BusyPct = [Busy%]
VAR CPLHVal = [CPLH]
VAR AHTVal = [AHT]
RETURN
    "Busy: " & BusyPct & 
    " | CPLH: " & CPLHVal & 
    " | AHT: " & AHTVal
```

---

### Row Context Check
**Purpose**: Verify if measure has row context  
**Date Added**: [Date]  

```dax
Row Context Check = 
IF(
    HASONEVALUE(YourTable[YourColumn]),
    "Single Value: " & VALUES(YourTable[YourColumn]),
    "Multiple Values or No Filter"
)
```

---

### Filter Context Debug
**Purpose**: See what filters are active  
**Date Added**: [Date]  

```dax
Active Filters = 
CONCATENATEX(
    FILTERS(YourTable[YourColumn]),
    YourTable[YourColumn],
    ", "
)
```

---

## Code Templates

### Basic Measure Template
```dax
[Measure Name] = 
VAR Variable1 = [Some Calculation]
VAR Variable2 = [Another Calculation]
RETURN
    IF(
        [Condition Check],
        [Result if True],
        [Result if False]
    )
```

### Ranking Template
```dax
[Rank Measure] = 
RANKX(
    ALL(Table[Column]),     -- What to rank against
    [Measure to Rank],      -- What measure to rank
    ,                       -- Value (optional)
    DESC,                   -- Order (DESC/ASC)
    Dense                   -- Ties handling
)
```

---

## Change Log

| Date | Measure | Change | Author |
|------|---------|--------|---------|
| [Date] | [Measure Name] | [Description of change] | [Your Name] |
| | | | |

---

## Best Practices Reminders

### Performance
- Use VAR for complex calculations
- Avoid nested CALCULATE when possible
- Use DIVIDE instead of "/" for safety

### Readability
- Use meaningful variable names
- Add comments for complex logic
- Format code consistently

### Testing
- Test with different filter contexts
- Validate edge cases (zeros, blanks)
- Compare results with source data

---

*For DAX questions or code reviews, contact: [Your Name] - [Your Email]*