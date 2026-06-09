# DAX Measures — Meta Ad Performance Dashboard

All measures live on the `ad_events` table unless noted.

---

## Core Event Counters

```dax
Impressions =
COUNTROWS(FILTER(ad_events, ad_events[event_type] = "Impression"))
```

```dax
Clicks =
COUNTROWS(FILTER(ad_events, ad_events[event_type] = "Click"))
```

```dax
Comments =
COUNTROWS(FILTER(ad_events, ad_events[event_type] = "Comment"))
```

```dax
Shares =
COUNTROWS(FILTER(ad_events, ad_events[event_type] = "Share"))
```

```dax
Purchases =
COUNTROWS(FILTER(ad_events, ad_events[event_type] = "Purchase"))
```

> **Note:** Engagements is referenced by other measures. Define it as a separate measure combining Clicks + Likes + Comments + Shares, or however Engagements is counted in your model.

---

## Rate Measures

```dax
CTR =
DIVIDE([Clicks], [Impressions], 0)
```

```dax
Conversion Rate =
DIVIDE([Purchases], [Clicks], 0)
```

```dax
Engagegment Rate =
DIVIDE([Engagements], [Impressions], 0)
```

```dax
Purchase Rate =
DIVIDE([Purchases], [Impressions], 0)
```

---

## Budget Measures

*(Both measures live on the `campaigns` table)*

```dax
Total Budget =
SUM(campaigns[total_budget])
```

```dax
Average Budget per Campaign =
AVERAGE(campaigns[total_budget])
```

---

## Dynamic Title Measures

These measures update chart titles automatically when the **Select Dynamic Measure** slicer changes.

```dax
Age Title =
SWITCH(
    SELECTEDVALUE('Select Dynamic Measure'[Select Dynamic Measure Order]),
    0, "Engagements by Age",
    1, "Impressions by Age",
    2, "Clicks by Age",
    3, "Shares By Age",
    4, "Comments by Age",
    5, "Purchase by Age"
)
```

```dax
Gender Title =
SWITCH(
    SELECTEDVALUE('Select Dynamic Measure'[Select Dynamic Measure Order]),
    0, "Engagements by Gender",
    1, "Impressions by Gender",
    2, "Clicks by Gender",
    3, "Shares By Gender",
    4, "Comments by Gender",
    5, "Purchase by Gender"
)
```

---

## Usage Notes

- All rate measures use `DIVIDE()` with a zero fallback to avoid division-by-zero errors.
- `Age Title` and `Gender Title` are set as the **Title** expression in each visual's formatting pane so they update dynamically with the slicer.
- The integer order values (0–5) match `Select Dynamic Measure Order` in the Field Parameter table — keep these in sync if you add or reorder metrics.
