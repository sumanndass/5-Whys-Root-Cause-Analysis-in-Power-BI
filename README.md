# 🔍 5 Whys Root Cause Analysis in Power BI
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
