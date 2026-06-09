# Data Dictionary — Meta Ad Performance Dashboard

All four source tables are imported from Excel. Column types reflect how Power BI interprets each field after load.

---

## Table: `campaigns`

Stores metadata for each advertising campaign.

| Column | Data Type | Description |
|---|---|---|
| `campaign_id` | Whole Number | Unique identifier for each campaign (primary key) |
| `name` | Text | Descriptive campaign name (e.g. `Campaign_1_Launch`) |
| `start_date` | Date | Campaign start date |
| `end_date` | Date | Campaign end date |
| `duration_days` | Whole Number | Total length of the campaign in days |
| `total_budget` | Decimal Number | Total allocated budget for the campaign (currency) |

---

## Table: `ads`

One row per individual ad creative. Each ad belongs to a campaign and has targeting attributes.

| Column | Data Type | Description |
|---|---|---|
| `ad_id` | Whole Number | Unique identifier for each ad (primary key) |
| `campaign_id` | Whole Number | Foreign key linking to `campaigns[campaign_id]` |
| `ad_platform` | Text | Publishing platform — `Facebook` or `Instagram` |
| `ad_type` | Text | Ad format — `Video`, `Image`, `Carousel`, or `Stories` |
| `target_gender` | Text | Intended gender audience — `Male`, `Female`, or `All` |
| `target_age_group` | Text | Age bracket targeted (e.g. `25-34`, `35-44`) |
| `target_interests` | Text | Comma-separated interest categories (e.g. `travel, photography`) |

---

## Table: `ad_events`

Grain-level event log. One row per user interaction with an ad. This is the central fact table.

| Column | Data Type | Description |
|---|---|---|
| `event_id` | Whole Number | Unique identifier for each event (primary key) |
| `ad_id` | Whole Number | Foreign key linking to `ads[ad_id]` |
| `user_id` | Text | Identifier of the user who triggered the event |
| `timestamp` | Date/Time | Full datetime of the event |
| `day_of_week` | Text | Day name derived from timestamp (e.g. `Saturday`) |
| `time_of_day` | Text | Time bucket — `Morning`, `Afternoon`, `Evening`, or `Night` |
| `event_type` | Text | Type of interaction — `Impression`, `Click`, `Share`, `Comment`, `Purchase`, `Like` |

### Calculated Column (added in Power BI)

| Column | Formula | Description |
|---|---|---|
| `Event Date` | `DATEVALUE(ad_events[timestamp])` | Date-only version of timestamp; used to join to the Calendar Table |

---

## Table: `users`

User-level demographic profile. One row per unique user.

| Column | Data Type | Description |
|---|---|---|
| `user_id` | Whole Number | Unique identifier for each user (primary key) |
| `user_gender` | Text | User's gender — `Male`, `Female`, or `Other` |
| `user_age` | Whole Number | User's age in years |
| `age_group` | Text | Age bracket the user falls into (e.g. `25-34`, `55-65`) |
| `country` | Text | User's country of residence |
| `location` | Text | City or town name |
| `interests` | Text | Comma-separated interest tags (e.g. `food, finance`) |

---

## DAX-Generated Tables (not imported)

### `Calender Table`

Built using `CALENDAR()` spanning the full date range of `ad_events`.

| Column | Formula | Description |
|---|---|---|
| `Date` | *(base column from CALENDAR)* | Every calendar date in the event range |
| `Day name` | `FORMAT([Date], "ddd")` | Abbreviated day name (e.g. `Mon`) |
| `Day Number` | `FORMAT([Date], "d")` | Numeric day of month |
| `Month` | `FORMAT([Date], "mmm")` | Abbreviated month name (e.g. `Jun`) |
| `WEEK Day` | `WEEKDAY([Date], 2)` | ISO weekday number (1=Monday … 7=Sunday) |
| `Week Number` | `WEEKNUM([Date], 2)` | ISO week number of the year |

### `Select Dynamic Measure`

Field Parameter table. Used by the dynamic measure slicer to switch the metric shown in the Age and Gender visuals.

| Column | Description |
|---|---|
| `Select Dynamic Measure` | Display label shown in the slicer |
| `Select Dynamic Measure Fields` | `NAMEOF()` reference to the corresponding measure |
| `Select Dynamic Measure Order` | Integer (0–5) controlling slicer sort order |
