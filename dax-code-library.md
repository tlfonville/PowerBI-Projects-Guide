# DAX Code Library

This library consolidates all DAX objects from the **Genesys Dashboard Power BI Structure** document into a clean, copy-ready reference. Objects are grouped by type and listed with their original names.

---

## Tables (Calculated)

### `_Measures`
```DAX
_Measures = { BLANK() }
```

### `Calendar`
```DAX
Calendar =
CALENDAR( MIN(CallDetail[Report Date]), MAX(CallDetail[Report Date]) )
```

### `HourOfDay`
```DAX
HourOfDay = 
VAR t =
    ADDCOLUMNS (
        GENERATESERIES ( 0, 23, 1 ),                      -- 0..23
        "HourNum", [Value],
        // Base label, e.g., 7am, 6pm
        "HourLabel",
            VAR h = [Value]
            VAR twelve = IF ( h = 0 || h = 12, 12, MOD ( h, 12 ) )
            VAR suffix = IF ( h < 12, "am", "pm" )
            RETURN FORMAT ( twelve, "0" ) & suffix,

        // Shift flags
        "Is Business Hour", IF ( [Value] >= 8  && [Value] <= 17, TRUE (), FALSE () ),
        "Is Ext Day Hour",  IF ( [Value] >= 7  && [Value] <= 19, TRUE (), FALSE () ),
        "Is Ext Night Hour",IF ( [Value] >= 19 || [Value] <= 7 , TRUE (), FALSE () ),

        // Sort helpers for wrapped axes
        "ExtDaySortIndex",  IF ( [Value] >= 7,  [Value], [Value] + 24 ),   -- 7..19 then 0..6
        "NightSortIndex",   IF ( [Value] >= 19, [Value], [Value] + 24 ),   -- 19..23 then 0..7

        // Duplicate labels so each can have its own Sort By column
        "HourLabel_Regular",
            VAR h1 = [Value]
            VAR t1 = IF ( h1 = 0 || h1 = 12, 12, MOD ( h1, 12 ) )
            VAR s1 = IF ( h1 < 12, "am", "pm" )
            RETURN FORMAT ( t1, "0" ) & s1,

        "HourLabel_ExtDay",
            VAR h2 = [Value]
            VAR t2 = IF ( h2 = 0 || h2 = 12, 12, MOD ( h2, 12 ) )
            VAR s2 = IF ( h2 < 12, "am", "pm" )
            RETURN FORMAT ( t2, "0" ) & s2,

        "HourLabel_ExtNight",
            VAR h3 = [Value]
            VAR t3 = IF ( h3 = 0 || h3 = 12, 12, MOD ( h3, 12 ) )
            VAR s3 = IF ( h3 < 12, "am", "pm" )
            RETURN FORMAT ( t3, "0" ) & s3
    )
RETURN
SELECTCOLUMNS (
    t,
    "HourNum",            [HourNum],
    "HourLabel",          [HourLabel],
    "HourLabel_Regular",  [HourLabel_Regular],
    "HourLabel_ExtDay",   [HourLabel_ExtDay],
    "HourLabel_ExtNight", [HourLabel_ExtNight],
    "ExtDaySortIndex",    [ExtDaySortIndex],
    "NightSortIndex",     [NightSortIndex],
    "Is Business Hour",   [Is Business Hour],
    "Is Ext Day Hour",    [Is Ext Day Hour],
    "Is Ext Night Hour",  [Is Ext Night Hour]
)
```

### `Hours`
```DAX
Hours = 
VAR H = GENERATESERIES ( 0, 23, 1 )
RETURN
    ADDCOLUMNS (
        H,
        "Bucket",
            SWITCH (
                TRUE (),
                [Value] <= 6,  "Night",
                [Value] = 7,   "Early",
                [Value] <= 16, "Day",        -- 8–16
                "Evening"                      -- 17–23
            )
    )
```

### `Shift View`
```DAX
Shift View = 
DATATABLE(
    "Shift", STRING,
    { { "All" }, { "Regular" }, { "Ext Day" }, { "Ext Night" } }
)
```

### `StatusList`
```DAX
StatusList = 
DATATABLE(
  "StatusName", STRING,
  {
    {"Idle"},
    {"Interacting"},
    {"Finalizing Doc"}
  }
)
```

---

## Calculated Columns

### `AgentShifts[Shift Type]`
```DAX
Shift Type = 
VAR s =
    LOWER (
        SUBSTITUTE (
            SUBSTITUTE ( TRIM ( AgentShifts[Assigned Shift] ), ".", "" ),  -- handle "Ext. Days"
            "-", " "
        )
    )
RETURN
SWITCH (
    TRUE (),
    s IN { "ext days", "extended days", "front half days", "back half days", "days", "day" }, "Ext Day",
    s IN { "ext nights", "extended nights", "front half nights", "back half nights", "nights", "night" }, "Ext Night",
    "Regular"
)
```

### `CallDetail[Call Start Hour (0-23)]`
```DAX
Call Start Hour (0-23) = 
VAR dt = CallDetail[Call Start Time]         // must be Date/Time type
RETURN VALUE ( FORMAT ( dt, "H" ) )          // 0..23, locale-proof
```

### `HourOfDay[Shift Hour Group]`
```DAX
Shift Hour Group = 
VAR h = HourOfDay[HourNum]
RETURN
    SWITCH ( TRUE(),
        h >= 8  && h <= 17, "Regular Hours (8–17)",
        h >= 7  && h <= 19, "Extended Day (7–19)",
        /* else */           "Extended Night (19–7)"
    )
```

---

## Measures

### `APS`
```DAX
APS = 
VAR AgentsBase =
    FILTER(
        ADDCOLUMNS(
            ALLSELECTED(AgentDetails[Agent Name]),
            "Answered",
                CALCULATE(
                    COUNTROWS(
                        FILTER(
                            CallDetail,
                            CallDetail[Agent Name] <> BLANK()
                                && CallDetail[Call Direction] = "Inbound"
                                && CallDetail[Ghost Call] = FALSE()
                                && CallDetail[Abandoned] = FALSE()
                        )
                    )
                ),
            "OnQueueSec", CALCULATE(SUM(AgentDetails[ON QUEUE])),
            "IdleSec",    CALCULATE(SUM(AgentDetails[IDLE])),
            "BusySec",    CALCULATE(SUM(AgentDetails[BUSY])),
            "FinalSec",   CALCULATE(COALESCE(SUM(AgentDetails[FINALIZING_DOC]), 0)),
            "AHTs",
                CALCULATE(
                    AVERAGEX(
                        FILTER(
                            CallDetail,
                            CallDetail[Agent Name] <> BLANK()
                                && CallDetail[Call Direction] = "Inbound"
                                && CallDetail[Ghost Call] = FALSE()
                                && CallDetail[Abandoned] = FALSE()
                        ),
                        CallDetail[Handle Time]
                    )
                )
        ),
        [Answered] > 0 && [OnQueueSec] > 0
    )

VAR AgentsEval =
    ADDCOLUMNS(
        AgentsBase,
        "CPH",       DIVIDE([Answered], DIVIDE([OnQueueSec], 3600)),
        "BusyShare", DIVIDE([BusySec], [OnQueueSec] + [IdleSec] + [BusySec] + [FinalSec])
    )

VAR N = COUNTROWS(AgentsEval)

-- current-row stats
VAR CurAnswered =
    CALCULATE(
        COUNTROWS(
            FILTER(
                CallDetail,
                CallDetail[Agent Name] <> BLANK()
                    && CallDetail[Call Direction] = "Inbound"
                    && CallDetail[Ghost Call] = FALSE()
                    && CallDetail[Abandoned] = FALSE()
            )
        )
    )
VAR CurOnQueueSec = CALCULATE(SUM(AgentDetails[ON QUEUE]))
VAR CurOnQueueHrs = DIVIDE(CurOnQueueSec, 3600)
VAR CurCPH        = DIVIDE(CurAnswered, CurOnQueueHrs)
VAR CurAHT =
    CALCULATE(
        AVERAGEX(
            FILTER(
                CallDetail,
                CallDetail[Agent Name] <> BLANK()
                    && CallDetail[Call Direction] = "Inbound"
                    && CallDetail[Ghost Call] = FALSE()
                    && CallDetail[Abandoned] = FALSE()
            ),
            CallDetail[Handle Time]
        )
    )
VAR CurBusySec    = CALCULATE(SUM(AgentDetails[BUSY]))
VAR CurIdleSec    = CALCULATE(SUM(AgentDetails[IDLE]))
VAR CurFinalSec   = CALCULATE(COALESCE(SUM(AgentDetails[FINALIZING_DOC]), 0))
VAR CurOnShiftSec = CurOnQueueSec + CurIdleSec + CurBusySec + CurFinalSec
VAR CurBusyShare  = DIVIDE(CurBusySec, CurOnShiftSec)

-- percentile ranks (equal-weight 3-way)
VAR CPHRank  = IF(N > 1, RANKX(AgentsEval, [CPH],       CurCPH,       DESC, DENSE), 1)
VAR AHTRank  = IF(N > 1, RANKX(AgentsEval, [AHTs],      CurAHT,       ASC,  DENSE), 1)
VAR BusyRank = IF(N > 1, RANKX(AgentsEval, [BusyShare], CurBusyShare, ASC,  DENSE), 1)

VAR CPHPct   = IF(N <= 1, 1, 1 - DIVIDE(CPHRank  - 1, N - 1))
VAR AHTPct   = IF(N <= 1, 1, 1 - DIVIDE(AHTRank  - 1, N - 1))
VAR BusyPct  = IF(N <= 1, 1, 1 - DIVIDE(BusyRank - 1, N - 1))

VAR APS_Raw_PerAgent = 100 * DIVIDE( CPHPct + AHTPct + BusyPct, 3 )

-- raw APS for all visible agents (for mean)
VAR APSTable =
    ADDCOLUMNS(
        AgentsEval,
        "APSraw",
            VAR r1 = IF(N > 1, RANKX(AgentsEval, [CPH],       [CPH],       DESC, DENSE), 1)
            VAR r2 = IF(N > 1, RANKX(AgentsEval, [AHTs],      [AHTs],      ASC,  DENSE), 1)
            VAR r3 = IF(N > 1, RANKX(AgentsEval, [BusyShare], [BusyShare], ASC,  DENSE), 1)
            VAR p1 = IF(N <= 1, 1, 1 - DIVIDE(r1 - 1, N - 1))
            VAR p2 = IF(N <= 1, 1, 1 - DIVIDE(r2 - 1, N - 1))
            VAR p3 = IF(N <= 1, 1, 1 - DIVIDE(r3 - 1, N - 1))
            RETURN 100 * DIVIDE( p1 + p2 + p3, 3 )
    )

VAR APS_Mean = AVERAGEX(APSTable, [APSraw])

-- === TUNING (even, linear) ===
VAR TARGET_MEAN = 80     -- set where you want the average to land
VAR STRETCH     = 1.15   -- >1 raises everyone and widens spread evenly; 1.00 = no stretch
-- ============================

VAR APS_Adjusted =
    MIN(100,
        MAX(0, (APS_Raw_PerAgent - APS_Mean) * STRETCH + TARGET_MEAN)
    )

VAR Result_Overall =
    AVERAGEX(
        APSTable,
        MIN(100, MAX(0, ([APSraw] - APS_Mean) * STRETCH + TARGET_MEAN))
    )

RETURN IF( HASONEVALUE(AgentDetails[Agent Name]), APS_Adjusted, Result_Overall )
```

### `Time by Status`
```DAX
Time by Status = 
VAR CurrentStatus =
    SELECTEDVALUE ( 'StatusList'[StatusName] )
VAR StatusTime =
    SWITCH (
        CurrentStatus,
        "Idle",         SUM ( AgentDetails[IDLE] ),
        "Interacting",  SUM ( AgentDetails[INTERACTING] ),
        "Finalizing Doc", SUM ( AgentDetails[FINALIZING_DOC] ),
        0
    )
VAR TotalTime =
    SUM ( AgentDetails[IDLE] ) +
    SUM ( AgentDetails[INTERACTING] ) +
    SUM ( AgentDetails[FINALIZING_DOC] )
RETURN
    DIVIDE ( StatusTime, TotalTime, 0 )
```

### `Abandonment Rate`
```DAX
Abandonment Rate = 
VAR Offered =
    FILTER (
        CallDetail,
        CallDetail[Call Direction] = "Inbound" &&
        NOT ISBLANK ( CallDetail[Queue] )      &&
        [Shift Row]
    )
VAR OfferedN  = COUNTROWS ( Offered )
VAR AbandonN  = COUNTROWS ( FILTER ( Offered, CallDetail[Abandoned] = TRUE () ) )
VAR Rate      = DIVIDE ( AbandonN, OfferedN, 0 )
RETURN FORMAT ( Rate, "0.00%" )
```

### `Avg Calls per Agent per Hour`
```DAX
Avg Calls per Agent per Hour = 
VAR hr =
    SELECTEDVALUE ( HourOfDay[HourNum] )          -- 0..23 from the axis
VAR SelectedShift =
    SELECTEDVALUE ( 'Shift View'[Shift], "All" )

-- classify the current hour
VAR isBiz      = hr >= 8  && hr <= 17              -- 8a..5p
VAR isExtDay   = hr >= 7  && hr <= 19              -- 7a..7p
VAR isExtNight = hr >= 19 || hr <= 7               -- 7p..7a

-- denominator: total agent-hours on-queue in this hour, respecting shift rules
VAR OnQSec =
    CALCULATE (
        SUM ( AgentDetails[ON QUEUE] ),            -- seconds
        FILTER (
            AgentDetails,
            VALUE ( AgentDetails[Hour] ) = hr      -- ensure numeric comparison
                &&
            (
                SelectedShift = "All"
                || ( SelectedShift = "Regular" && isBiz )
                || ( SelectedShift = "Ext Day"   &&
                     VAR aType = LOOKUPVALUE (
                                      AgentShifts[Shift Type],
                                      AgentShifts[Agent Name], AgentDetails[Agent Name],
                                      "Regular"
                                   )
                     RETURN ( (aType = "Ext Day"  && isExtDay)
                           || (aType = "Regular"  && isExtDay && NOT isBiz) )
                  )
                || ( SelectedShift = "Ext Night" &&
                     VAR aType2 = LOOKUPVALUE (
                                        AgentShifts[Shift Type],
                                        AgentShifts[Agent Name], AgentDetails[Agent Name],
                                        "Regular"
                                      )
                     RETURN ( (aType2 = "Ext Night" && isExtNight)
                           || (aType2 = "Regular"   && isExtNight && NOT isBiz) )
                  )
            )
        )
    )
VAR OnQHours = DIVIDE ( OnQSec, 3600, 0 )

-- numerator: handled calls in this hour, shift-aware (same hour gate)
VAR Calls =
    CALCULATE (
        DISTINCTCOUNT ( CallDetail[Call ID] ),
        CallDetail[Call Direction] = "Inbound",
        CallDetail[Ghost Call] = FALSE (),
        CallDetail[Abandoned]  = FALSE (),
        FILTER ( CallDetail, VALUE ( CallDetail[Call Start Hour (0-23)] ) = hr ),
        FILTER ( CallDetail, [Shift Row] )
    )

RETURN DIVIDE ( Calls, OnQHours, 0 )
```

### `Avg Handle`
```DAX
Avg Handle = 
VAR AnsweredRows =
    FILTER (
        CallDetail,
        CallDetail[Call Direction] = "Inbound" &&
        NOT ISBLANK ( CallDetail[Queue] )      &&
        CallDetail[Abandoned] = FALSE ()       &&
        [Shift Row]
    )
VAR AvgSec = AVERAGEX ( AnsweredRows, CallDetail[Handle Time] )
VAR SecRounded = ROUND ( AvgSec, 0 )
RETURN FORMAT ( INT ( SecRounded / 60 ), "00" ) & ":" & FORMAT ( MOD ( SecRounded, 60 ), "00" )
```

### `Avg Wait`
```DAX
Avg Wait = 
VAR AnsweredRows =
    FILTER (
        CallDetail,
        CallDetail[Call Direction] = "Inbound" &&
        NOT ISBLANK ( CallDetail[Queue] )      &&
        CallDetail[Abandoned] = FALSE ()       &&
        [Shift Row]
    )
VAR AvgSec =
    DIVIDE ( SUMX ( AnsweredRows, CallDetail[Wait Time] ), COUNTROWS ( AnsweredRows ) )
VAR SecRounded = ROUND ( AvgSec, 0 )
RETURN FORMAT ( INT ( SecRounded / 60 ), "00" ) & ":" & FORMAT ( MOD ( SecRounded, 60 ), "00" )
```

### `Calls by Hour`
```DAX
Calls by Hour = 
VAR h = SELECTEDVALUE ( HourOfDay[HourNum] )
RETURN
COUNTROWS (
    FILTER (
        CallDetail,
        CallDetail[Call Direction] = "Inbound" &&
        NOT ISBLANK ( CallDetail[Queue] )      &&
        CallDetail[Abandoned] = FALSE ()       &&
        CallDetail[Call Start Hour (0-23)] = h &&
        [Shift Row]
    )
)
```

### `Handled Calls`
```DAX
Handled Calls = 
VAR CallsInScope =
    FILTER (
        CallDetail,
        CallDetail[Call Direction] = "Inbound" &&
        CallDetail[Ghost Call] = FALSE ()     &&
        CallDetail[Abandoned]  = FALSE ()     &&
        [Shift Row]
    )
RETURN COUNTROWS ( CallsInScope )
```

### `QLBI`
```DAX
QLBI = 
VAR Agents =
    FILTER (
        VALUES ( CallDetail[Agent Name] ),
        NOT ISBLANK ( CallDetail[Agent Name] )
    )

VAR CallsPerAgent =
    ADDCOLUMNS (
        Agents,
        "Calls",
            CALCULATE (
                DISTINCTCOUNT ( CallDetail[Call ID] ),
                CallDetail[Call Direction] = "Inbound",
                CallDetail[Ghost Call] = FALSE (),
                CallDetail[Abandoned]  = FALSE (),
                FILTER ( CallDetail, [Shift Row] ),
                KEEPFILTERS ( CallDetail[Agent Name] = EARLIER ( CallDetail[Agent Name] ) )
            )
    )

VAR NonZero =
    FILTER ( CallsPerAgent, [Calls] > 0 )

VAR n = COUNTROWS ( NonZero )
RETURN
IF (
    n <= 1,
    0,
    DIVIDE (
        STDEVX.P ( NonZero, [Calls] ),
        AVERAGEX ( NonZero, [Calls] ),
        0
    )
)
```

### `Shift Row`
```DAX
Shift Row = 
VAR SelectedShift = SELECTEDVALUE ( 'Shift View'[Shift], "All" )
VAR h = SELECTEDVALUE ( CallDetail[Call Start Hour (0-23)] )
VAR isBiz      = h >= 8  && h <= 17         // 8a..5p
VAR isExtDay   = h >= 7  && h <= 19         // 7a..7p
VAR isExtNight = h >= 19 || h <= 7          // 7p..7a

// For answered calls we know the agent; for abandons we often don't.
VAR agentName  = SELECTEDVALUE ( CallDetail[Agent Name] )
VAR agentType  = LOOKUPVALUE (
                    AgentShifts[Shift Type], AgentShifts[Agent Name], agentName, "Regular"
                 )
VAR hasAgent   = NOT ISBLANK ( agentName )

VAR inRegular =
    // Everyone’s 8–5 should be counted on the Regular view
    isBiz

VAR inExtDay =
    IF (
        hasAgent,
        // All hours for true Ext Day agents; only 7am/6pm/7pm for Regular agents
        (agentType = "Ext Day"  && isExtDay) ||
        (agentType = "Regular"  && isExtDay && NOT isBiz),
        // Abandon/unknown-agent: classify by hour only
        isExtDay && NOT isBiz
    )

VAR inExtNight =
    IF (
        hasAgent,
        (agentType = "Ext Night" && isExtNight) ||
        (agentType = "Regular"   && isExtNight && NOT isBiz),
        isExtNight
    )

RETURN
SWITCH (
    SelectedShift,
    "Regular",   inRegular,
    "Ext Day",   inExtDay,
    "Ext Night", inExtNight,
    TRUE()   // "All"
)
```

---

## Notes
- Measures that format time as `mm:ss` (`Avg Handle`, `Avg Wait`) return **text**; if you need numeric seconds, expose separate measures (e.g., `Avg Handle (s)`).
- `Shift Row` is a **boolean measure** designed to be used as a visual/filter predicate to keep shift-aware logic consistent across visuals.
- `HourOfDay` provides wrapped sort indexes for correct visual ordering of extended-day/night axes.
