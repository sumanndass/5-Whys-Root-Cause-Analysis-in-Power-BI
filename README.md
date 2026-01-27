# 🔍 5 Whys RCA Framework PowerBI
**5 Whys is a root cause analysis technique where you repeatedly ask “Why?” (typically 5–7 times) until you move from a visible problem to its true underlying cause.**
- Incident Count & Financial Impact Driven RCA Framework

## 📌 Overview
- This repository demonstrates a production-grade 5 Whys Root Cause Analysis (RCA) framework built in Power BI, designed to answer leadership-level questions such as:
  - Why did SLA breaches happen?
  - Which root causes are driving financial impact?
  - Is the trend driven by incident volume or by impact severity?
- The model supports dynamic metric switching, cumulative trend analysis, and hierarchical root-cause drill-downs across multiple “Why” levels.

## 🧠 Business Problem
- Traditional RCA dashboards fail because they:
  - Focus only on incident count
  - Ignore financial exposure
  - Do not show cause-to-impact flow
  - Cannot explain why the number keeps increasing
- This model solves that by:
  - Allowing users to toggle between Incident Count and Total Impact
  - Showing day-wise cumulative exposure
  - Mapping incidents across 5 levels of causation
  - Maintaining one clean DAX logic across all visuals

## 🏗️ Data Model Design
- [Fact Table](https://github.com/sumanndass/5-Whys-Root-Cause-Analysis-in-Power-BI/blob/main/Incident%20File.xlsx)
  - Incident File
    - Incident_ID
    - Impact_Value
    - Date
    - Problem Type
    - Why1 → Why5
- Dimension Tables
  - CalendarTbl – Date intelligence
  - What to See – Metric selector (Field Parameters)

## 🎛️ Field Parameter: Metric Switcher
```dax
What to See = {
    ("Incident Count", NAMEOF('Incident File'[_Incident Count]), 0),
    ("Total Impact", NAMEOF('Incident File'[_Total Impact]), 1)
}
```
- 🔎 Why this exists
  - This allows one slicer to control:
    - KPI cards
    - Line charts
    - Decomposition tree
    - Cumulative logic
- 🧩 Key Columns Created
  | Column            | Purpose               |
  | ----------------- | --------------------- |
  | Label             | Display name          |
  | Measure Reference | DAX measure pointer   |
  | Order             | Numeric control logic |
  - ⚠️ Order is critical — it drives all SWITCH logic downstream.

## 📅 Calendar Table
```dax
CalenderTbl = 
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date])
)
```
- 🔎 Why CALENDARAUTO?
  - Automatically detects min/max dates from the model
  - Safe for incident-based datasets
  - Eliminates manual date maintenance

## 📊 Core Measures
- 1️⃣ Incident Count
  ```dax
  _Incident Count = 
  COUNTA('Incident File'[Incident_ID])
  ```
  - Design choice
    - COUNTA ensures robustness if Incident_ID is text
    - Avoids blank issues from deleted records
- 2️⃣ Total Impact
  ```dax
  _Total Impact = 
  SUM('Incident File'[Impact_Value])
  ```
  - Design choice
    - Financial exposure measure
    - Used consistently across all RCA layers

## 📈 Cumulative Trend Measure (Most Important)
```dax
_Cum = 
VAR Ord = SELECTEDVALUE('What to See'[What to See Order])
RETURN
IF(
    SWITCH(
        Ord,
        0, CALCULATE([_Incident Count]),
        1, CALCULATE([_Total Impact])
    ) <> BLANK(),
    SUMX(
        WINDOW(
            1, ABS,
            0, REL,
            ALLSELECTED(CalenderTbl[Date]),
            ORDERBY(CalenderTbl[Date])
        ),
        SWITCH(
            Ord,
            0, CALCULATE([_Incident Count]),
            1, CALCULATE([_Total Impact])
        )
    )
)
```

## 🔬 Deep Dive: How ```_Cum``` Works
- Step 1: Detect selected metric
  ```dax
  VAR Ord = SELECTEDVALUE('What to See'[What to See Order])
  ```
  - 0 → Incident Count
  - 1 → Total Impact
- Step 2: Prevent blank rows
  ```dax
  IF( SelectedMeasure <> BLANK(), ... )
  ```
  - Stops cumulative lines from:
    - Extending into future dates
    - Showing flat tails
- Step 3: WINDOW() for cumulative logic\
  ```dax
  WINDOW(
      1, ABS,
      0, REL,
      ALLSELECTED(CalenderTbl[Date]),
      ORDERBY(CalenderTbl[Date])
  )
  ```
  | Parameter     | Meaning                       |
  | ------------- | ----------------------------- |
  | `1, ABS`      | Start from first visible date |
  | `0, REL`      | End at current row            |
  | `ALLSELECTED` | Respect slicers               |
  | `ORDERBY`     | Chronological accumulation    |
- Step 4: Dynamic aggregation
  ```dax
  SWITCH(
      Ord,
      0, [_Incident Count],
      1, [_Total Impact]
  )
  ```
- Same cumulative logic → two different business metrics
- No duplicate DAX. No maintenance nightmare.

## 🧾 Dynamic Line Chart Title
```dax
_Line Title = 
VAR Ord = SELECTEDVALUE('What to See'[What to See Order])
RETURN
    SWITCH(
        TRUE(),
        Ord = 0, "Daily Incident Volume Trend",
        Ord = 1, "Daily Impact Exposure Trend"
    )
```
- 🎯 Why this matters
  - Avoids misleading visuals
  - Titles adapt instantly with slicer
  - Improves executive trust

## 🌳 5 Whys Decomposition Tree
```dax
Problem Type
 → Why1
   → Why2
     → Why3
       → Why4
         → Why5
```
- Key Strengths
  - Single measure feeds entire tree
  - Can pivot between volume-driven RCA and impact-driven RCA
  - Reveals governance & process failures, not just symptoms

## 📊 Dashboard Highlights
- ✔ KPI Cards – Incident Count & Total Impact
- ✔ Metric Toggle (What to See)
- ✔ Cumulative Exposure Trend
- ✔ Day-wise progression
- ✔ Root Cause Drill-down
- ✔ Financial attribution at every Why level

## 🚀 Why This Model Is Enterprise-Grade
- Uses modern DAX (WINDOW, Field Parameters)
- No duplicated measures
- Fully slicer-aware
- Scales across years & datasets
- Business-explainable logic

## 🏁 Final Thoughts
- This framework transforms RCA from storytelling to evidence-based decision making.
- Instead of:
  - ```“Incidents increased”```
- You now answer:
  - ```“Weak Vendor SLA governance caused ₹1.34M exposure across 13 incidents.”```

## 💪 Power BI File
[Download Incident Report.pbix file](https://github.com/sumanndass/5-Whys-Root-Cause-Analysis-in-Power-BI/blob/main/Incident%20Report.pbix)
