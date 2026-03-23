# Transportation & Logistics Analytics

![Power BI Dashboard](Logistics_challenge_dashboard.pdf)

## Overview

An end-to-end logistics analytics project analyzing **4,000+ shipment bookings** across 184 suppliers and 36 customers in India. I performed comprehensive data cleaning, null value imputation, and feature engineering in Python, then built a 3-page interactive Power BI dashboard covering shipment performance, supplier/customer analysis, and delay root cause investigation.

**Tools:** Python (pandas, NumPy, matplotlib, seaborn, geopy) · Power BI  
**Dataset:** FP20 Analytics Challenge — Transportation & Logistics Tracking Dataset

---

## Problem Statement

The logistics operation lacked a unified view of delivery performance. With 61.4% of shipments delayed and an average delay of 11.49 days, the business needed visibility into:
- Which routes, suppliers, and vehicle types were driving the most delays
- How transportation distance correlated with delivery performance
- Which materials and customers were most impacted by late deliveries
- Booking and delivery volume trends over time

---

## Dataset

Single primary table: `Transportation & Logistics Tracking Dataset.xlsx`

Key fields: Booking ID, Origin/Destination Location, Supplier Name, Customer Name, Material Shipped, Vehicle Type, Trip Start/End Date, Planned ETA, Actual ETA, Transportation Distance (KM), Minimum KMs To Be Covered In A Day, GPS Provider, Delivery Status (On Time / Delay)

---

## Data Cleaning & Feature Engineering (Python)

### Step 1 — Data Quality Assessment
- Verified shape, dtypes, null counts, and duplicate rows across the dataset
- Converted 5 date columns (`Planned ETA`, `Trip Start Date`, `Trip End Date`, `Actual ETA`, `Data Ping time`) from object to datetime using `pd.to_datetime`
- Identified and replaced string `"Null"` / `"NULL"` values in `Current Location` and `Gps Provider` columns with proper `np.nan`

### Step 2 — Null Value Imputation

**Transportation Distance (KM):**
- Filled nulls using route-level median (grouped by `Origin Location` × `Destination Location`)
- For remaining nulls, used state-level mean (grouped by `Origin State` × `Destination State`)
- Filled any residual nulls with overall column median

**Minimum KMs To Be Covered In A Day:**
- Applied business logic based on transportation distance:
  - Distance > 650 KM → 275 KM/day
  - 0 < Distance ≤ 650 KM → 250 KM/day
  - Distance = 0 → 0

**Actual ETA:**
- Cross-referenced `Data Ping time` — found only minor day differences in most rows
- Filled null `Actual ETA` values using date from `Data Ping time` + most common hour for that Origin-Destination State pair (computed via `.mode()`)

**Current Location:**
- Used `geopy` (Nominatim geocoder) with reverse geocoding from latitude/longitude coordinates to fill null location strings

**Vehicle Type / Driver Name:**
- Filled remaining nulls with `"Other"`

### Step 3 — Feature Engineering
- Extracted `Origin City`, `Origin State`, `Destination City`, `Destination State` from location strings using `lambda` + `str.split`
- Standardized city/state names (title case, stripped whitespace, corrected encoding issues)
- Corrected unofficial place names to official designations (e.g., `"Pondicherry"` → `"Puducherry"`)
- Added `Destination Country` flag (India vs. Nepal) based on state name
- Created `Delivery Status` column (`"Ontime"` / `"Delay"`) from binary `Ontime` field

### Step 4 — Anomalous Date Correction
- Identified two rows (index 3583, 3584) with erroneous date values using route-level delivery day analysis
- Used median delivery days for matching routes to derive corrected timestamps for `Planned ETA`, `Actual ETA`, `Trip Start Date`, and `Trip End Date`

### Step 5 — Final Cleanup
- Dropped unnecessary columns: `Actual ETA` (original), `Ontime`, `Driver Mobile No`, `Current Location` (original), `ETA Date`, `ETA_Hour`
- Renamed imputed columns to production names
- Exported cleaned data to a new sheet (`Cleaned_Data2`) in the original Excel file

---

## Power BI Dashboard (3 Pages)

### Page 1 — Overview
**KPI Cards:** Total Bookings (4K) · Items Available (712) · Avg Delivery Time (9 days) · Avg Distance (860.50 KM) · Customers (36) · Suppliers (184)

- **Material Shipment Statistics table:** Bookings, Avg Delivery Time, Avg Delay Time, Delayed %, Suppliers, and Customers per material type
- **Bookings by Delivery Status:** 61.42% Delay vs. 38.58% Before Time
- **Bookings by Shipment Type:** 98.46% Regular vs. 1.54% Market
- **Top 5 Most Common Shipment Routes** by total bookings and avg distance
- **Top 5 Routes by Highest Avg Delivery Time**

### Page 2 — Supplier & Customer Performance
- **Shipment Handled by Suppliers:** Before Time vs. Delay breakdown per supplier (Ekta Transport Company leading with 238 total)
- **Shipments Received by Customers:** Before Time vs. Delay per customer (Larsen & Toubro highest at 977 total)
- **Booking Trend:** Quarter-over-quarter volume from Q2 2019 to Q4 2020
- **Delivery Trend:** Quarter-over-quarter deliveries over same period

### Page 3 — Delay Analysis
**KPI Cards:** Total Bookings (4K) · Delayed % (61.4%) · Items Available (712) · Avg Delay Time (11.49 days)

- **Impact of Transportation Distance on Delivery:** Scatter plot — Before Time vs. Delay by distance
- **Top 5 Routes by Delayed Deliveries**
- **Top 5 Items by Delayed Deliveries:** Auto Parts (816), Empty Trays (153), Grs Starter (59)
- **Top 5 Vehicles by Delayed Deliveries:** 32 FT Multi-Axle 14MT HCV (682), 40 FT 3XL Trailer 35MT (528)
- **Top 5 Customers by Delayed Deliveries:** Larsen & Toubro (777), Ford India (476)
- **Top 5 Suppliers by Delayed Deliveries:** Ekta Transport Company (186), Trans Cargo India (172)
- **Delayed Deliveries Trend:** Q2 2019 → Q3 2020 (peak: 1,352 in Q3 2020)

---

## Key Findings

- **61.4% of all shipments were delayed** with an average delay of 11.49 days — a systemic issue, not an outlier problem
- **Auto Parts** was the most delayed material type (816 delayed bookings), representing a critical bottleneck in the supply chain
- **Ekta Transport Company** handled the highest volume but also had the most delayed deliveries (186), signaling a carrier performance issue
- **Larsen & Toubro** was the most impacted customer with 777 delayed shipments
- **32 FT Multi-Axle 14MT HCV** vehicles had the most delays (682) — may indicate overloading or route-capacity mismatches
- Delay volume grew significantly through 2020, peaking at 1,352 in Q3 2020 — likely COVID-19 supply chain disruption
- Long-distance routes (2,000+ KM) showed higher delay concentration, confirming distance as a delay risk factor

---

## Recommendations

1. **Audit Ekta Transport Company's operations** — highest volume carrier but disproportionate delay rate; review SLA compliance and route assignments
2. **Prioritize Auto Parts supply chain** — most bookings, highest delay count; route optimization and dedicated carrier allocation warranted
3. **Investigate 32 FT Multi-Axle HCV utilization** — top delayed vehicle type; assess whether vehicle-route matching is optimized
4. **Reduce long-haul single-handoff routes** — routes above 1,900 KM (e.g., Gurgaon→Tamil Nadu) consistently appear in delay hotspots
5. **Build quarterly delay monitoring into ops reviews** — trend data shows delay accumulation is visible early enough to intervene

---

## How to Run

```bash
# Install dependencies
pip install pandas numpy matplotlib seaborn scipy geopy openpyxl

# Run EDA & cleaning notebook
jupyter notebook Logistics_Challenge_EDA___Data_Cleaning.ipynb
```

Cleaned data will be exported to a new sheet `Cleaned_Data2` in the original Excel file for Power BI ingestion.

---

## Repository Structure

```
├── Logistics_Challenge_EDA___Data_Cleaning.ipynb   # Full EDA & data cleaning pipeline
├── Transportation___Logistics_Tracking_Dataset.xlsx # Raw + cleaned data
├── Logistics_challenge_dashboard.pdf               # Power BI dashboard export
├── README.md
```

