# DAX — Calendar Table & Dynamic Measure Parameter

---

## Calendar Table

The Calendar Table is created entirely inside Power BI using DAX. It is **not imported from Excel**.

### Base Table

```dax
Calender Table =
CALENDAR(
    MIN(ad_events[Event Date]),
    MAX(ad_events[Event Date])
)
```

This generates a contiguous date spine from the earliest to the latest event date in `ad_events`, automatically expanding when new data is loaded.

### Calculated Columns on Calender Table

```dax
Day name = FORMAT('Calender Table'[Date], "ddd")
```

```dax
Day Number = FORMAT('Calender Table'[Date], "d")
```

```dax
Month = FORMAT('Calender Table'[Date], "mmm")
```

```dax
WEEK Day = WEEKDAY('Calender Table'[Date], 2)
-- Returns 1 (Monday) through 7 (Sunday) per ISO convention
```

```dax
Week Number = WEEKNUM('Calender Table'[Date], 2)
-- ISO week numbering, week starts Monday
```

### Relationship

`Calender Table[Date]` → `ad_events[Event Date]` (one-to-many, single direction)

The `Event Date` calculated column on `ad_events` is required to establish this relationship:

```dax
-- Calculated column on ad_events table
Event Date = DATEVALUE(ad_events[timestamp])
```

---

## Select Dynamic Measure (Field Parameter)

The `Select Dynamic Measure` table is a **Field Parameter**, created via DAX. It powers the dynamic measure slicer that switches the KPI displayed in the Age and Gender charts.

```dax
Select Dynamic Measure = {
    ("Engagements", NAMEOF('ad_events'[Engagements]), 0),
    ("Impressions", NAMEOF('ad_events'[Impressions]), 1),
    ("Clicks",      NAMEOF('ad_events'[Clicks]),      2),
    ("Shares",      NAMEOF('ad_events'[Shares]),       3),
    ("Comments",    NAMEOF('ad_events'[Comments]),     4),
    ("Purchases",   NAMEOF('ad_events'[Purchases]),    5)
}
```

This generates three auto-named columns:

| Column | Purpose |
|---|---|
| `Select Dynamic Measure` | Label displayed in the slicer |
| `Select Dynamic Measure Fields` | `NAMEOF()` reference — points to the actual measure field |
| `Select Dynamic Measure Order` | Integer (0–5) — controls slicer sort order and drives the SWITCH in title measures |

### How the Dynamic Measure Works

1. User selects a metric (e.g. "Clicks") in the **Select Dynamic Measure** slicer.
2. The selected field reference (`Select Dynamic Measure Fields`) is used as the **Values** field in the Age bar chart and Gender donut chart.
3. The `Age Title` and `Gender Title` measures read `SELECTEDVALUE([Select Dynamic Measure Order])` via `SWITCH` and return the matching title string.
4. Both chart titles are configured to use the respective title measure as a dynamic expression.
