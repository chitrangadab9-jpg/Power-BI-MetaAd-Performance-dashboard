# Meta Ad Performance Dashboard — Power BI Project

A full-stack Power BI analytics project built on simulated Meta (Facebook & Instagram) advertising data. The dashboard tracks campaign performance across key metrics — impressions, clicks, conversions, engagement, and spend — with dynamic measure switching, platform filtering, and calendar-driven time intelligence.

---

## Repository Structure

```
meta-ad-performance-dashboard/
│
├── data/
│   ├── raw/
│   │   ├── campaigns_sample.xlsx       # Campaign metadata (id, name, dates, budget)
│   │   ├── ads_sample.xlsx             # Ad-level details (platform, type, targeting)
│   │   ├── ad_events_sample.xlsx       # Event-level log (impressions, clicks, purchases…)
│   │   └── users_sample.xlsx           # User demographics (age, gender, country, interests)
│   └── data_dictionary.md              # Column definitions for all four source tables
│
├── dax/
│   ├── measures.md                     # All DAX measures with descriptions
│   ├── calculated_columns.md           # Calculated columns (Event Date on ad_events)
│   ├── calendar_table.md               # Calendar Table DAX + all date columns
│   └── dynamic_measure_table.md        # Select Dynamic Measure parameter table DAX
│
├── assets/
│   ├── dashboard_preview.png           # Full dashboard screenshot
│   └── data_model.png                  # Data model / relationship diagram screenshot
│
├── Business Requirements/
│   └── KPIS, Charts_needed             # Extended write-up: goals, methodology, insights
│
└── README.md
```

---

## Data Model

The report is built on **4 imported tables** and **2 DAX-generated tables**:

| Table | Source | Description |
|---|---|---|
| `ad_events` | Excel import | Grain-level event log — one row per user interaction |
| `ads` | Excel import | Ad metadata — platform, type, targeting parameters |
| `campaigns` | Excel import | Campaign names, dates, and budgets |
| `users` | Excel import | User demographics and interest categories |
| `Calender Table` | DAX (`CALENDAR()`) | Date spine driven by ad_events date range |
| `Select Dynamic Measure` | DAX (Field Parameter) | Powers the dynamic measure slicer |

### Relationships

```
users (1) ──────── (*) ad_events
ad_events (*) ──── (1) ads
ads (*) ──────────── (1) campaigns
ad_events (*) ──── (1) Calender Table   [via Event Date]
```

---

## DAX — Measures

| Measure | Formula Summary |
|---|---|
| `Impressions` | `COUNTROWS` of events where `event_type = "Impression"` |
| `Clicks` | `COUNTROWS` of events where `event_type = "Click"` |
| `Comments` | `COUNTROWS` of events where `event_type = "Comment"` |
| `Shares` | `COUNTROWS` of events where `event_type = "Share"` |
| `Purchases` | `COUNTROWS` of events where `event_type = "Purchase"` |
| `CTR` | `DIVIDE([Clicks], [Impressions], 0)` |
| `Conversion Rate` | `DIVIDE([Purchases], [Clicks], 0)` |
| `Engagement Rate` | `DIVIDE([Engagements], [Impressions], 0)` |
| `Purchase Rate` | `DIVIDE([Purchases], [Impressions], 0)` |
| `Average Budget per Campaign` | `AVERAGE(campaigns[total_budget])` |
| `Total Budget` | `SUM(campaigns[total_budget])` |
| `Age Title` | `SWITCH` on Dynamic Measure order → dynamic chart title |
| `Gender Title` | `SWITCH` on Dynamic Measure order → dynamic chart title |

Full DAX code for every measure is in [`dax/measures.md`](dax/measures.md).

---

##  DAX — Calculated Columns

| Table | Column | Formula |
|---|---|---|
| `ad_events` | `Event Date` | `DATEVALUE(ad_events[timestamp])` |

---

## DAX — Calendar Table

Created entirely in DAX (not imported):

```dax
Calender Table = CALENDAR(MIN(ad_events[Event Date]), MAX(ad_events[Event Date]))
```

Additional columns on the Calendar Table:

| Column | Formula |
|---|---|
| `Day name` | `FORMAT('Calender Table'[Date], "ddd")` |
| `Day Number` | `FORMAT('Calender Table'[Date], "d")` |
| `Month` | `FORMAT('Calender Table'[Date], "mmm")` |
| `WEEK Day` | `WEEKDAY('Calender Table'[Date], 2)` |
| `Week Number` | `WEEKNUM('Calender Table'[Date], 2)` |

---

##  DAX — Dynamic Measure (Field Parameter)

The `Select Dynamic Measure` table is a **Field Parameter** that allows users to switch the KPI displayed across age and gender visuals from a single slicer:

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

The `Age Title` and `Gender Title` measures use `SWITCH(SELECTEDVALUE(...))` to update chart titles automatically when the slicer selection changes.

---

## Dashboard Features

- **12 KPI cards** — Impressions, Clicks, Comments, Shares, Purchases, Engagements, Engagement Rate, Conversion Rate, CTR, Purchase Rate, Avg Budget, Total Budget
- **Platform filter** — Toggle between Facebook and Instagram
- **Target Interest slicer** — Filter all visuals by audience interest category
- **Month calendar slicer** — Date navigation at the week/day level
- **Dynamic measure slicer** — Switch the metric driving the age and gender breakdowns
- **Impressions by Gender** — Donut chart
- **Impressions / [Dynamic Metric] by Age** — Bar chart
- **Impressions by Country** — Bing Maps visual
- **AD_TYPE Matrix** — Conversion rate, engagement rate, and purchase rate by ad type (Carousel, Image, Stories, Video)
- **Impressions by Week Number and Ad Type** — Stacked bar chart with ad type color legend

---

## Data Dictionary

See [`data/data_dictionary.md`](data/data_dictionary.md) for full column-level definitions.

**Quick reference:**

| Table | Key Columns |
|---|---|
| `campaigns` | `campaign_id`, `name`, `start_date`, `end_date`, `duration_days`, `total_budget` |
| `ads` | `ad_id`, `campaign_id`, `ad_platform`, `ad_type`, `target_gender`, `target_age_group`, `target_interests` |
| `ad_events` | `event_id`, `ad_id`, `user_id`, `timestamp`, `day_of_week`, `time_of_day`, `event_type` |
| `users` | `user_id`, `user_gender`, `user_age`, `age_group`, `country`, `location`, `interests` |

---

## Tools Used

- **Power BI Desktop** — Report development, DAX authoring, data modelling
- **Microsoft Excel** — Raw data source (4 tables)
- **DAX** — All measures, calculated columns, Calendar Table, Field Parameter
- **Bing Maps** — Geographic visual (built-in Power BI)

---

##  How to Use

1. Clone or download this repository.
2. Open the `.pbix` file in Power BI Desktop.
3. If data refresh is needed, update the file paths for the four Excel sources under **Transform Data → Data Source Settings**.
4. Use the platform toggle (Facebook / Instagram) and the **Select Dynamic Measure** slicer to explore the dashboard interactively.

---

## Sample Data Note

The Excel files in `data/raw/` are **sample extracts** (representative rows only) shared for portfolio documentation purposes. The full dataset used to build the dashboard is synthetic and was generated for this project.

---

*Built by Chitra · Power BI Portfolio Project*

