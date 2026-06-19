# Statistical Process Control (SPC) in Power BI

This repository demonstrates how native Power BI capabilities can be used to build Statistical Process Control (SPC) reports without requiring custom visuals from Microsoft AppSource.

The solution includes interactive reports for both Process Capability Analysis and Individuals & Moving Range (I-MR) Control Charts using DAX, Power Query, and a simple star schema data model.

---

## Features

### Process Capability Analysis

The Process Capability report calculates long-term process capability statistics including:

- Pp
- Ppk
- Ppl
- Ppu

These metrics measure process variation relative to Upper and Lower Specification Limits (USL/LSL) and help evaluate whether a manufacturing process is capable of consistently producing within specification.

### Individuals & Moving Range (I-MR) Control Charts

The I-MR report provides:

- Individual value control charts
- Moving Range charts
- Dynamically calculated control limits
- Automatic highlighting of Nelson Rule 1 violations
- Interactive filtering across manufacturing sites and process parameters

---

## Dataset

The project uses a mock manufacturing dataset consisting of:

- 100 production batches
- 3 manufacturing sites
- Multiple Key Performance Indicators (KPIs)

The source data is stored in a wide-format CSV file to simulate a typical manufacturing data export.

---

## Power Query Transformations

Only minimal data transformations were required. The primary transformation was an **Unpivot Columns** operation that converted the wide dataset into a normalized fact table.

The resulting structure includes:

- **Parameter** — KPI name
- **Value** — KPI measurement
- Batch metadata and manufacturing attributes

This format enables a scalable data model capable of supporting any number of process parameters without modifying report logic.

---

## Data Model

The solution uses a simple star schema consisting of:

- **Fact table**
  - Individual KPI measurements

- **Parameter dimension**
  - Parameter name
  - Units
  - Lower Specification Limit (LSL)
  - Upper Specification Limit (USL)

The specification limits are consumed by DAX measures to calculate process capability statistics and dynamically render specification and control limits throughout the reports.

---

## Technologies

- Power BI
- DAX
- Power Query (M)
- Statistical Process Control (SPC)
- Process Capability Analysis
- Star Schema Data Modeling
