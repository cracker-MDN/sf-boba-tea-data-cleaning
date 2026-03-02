# SF Boba Tea Shop Data Cleaning

## Project Overview

This project is part of the **Google Data Analytics Professional Certificate** curriculum. The goal was to clean a real-world style dataset of boba tea shops in the San Francisco Bay Area, making it ready for mapping and rating analysis.

The dataset was sourced from Yelp and contains shop names, ratings, addresses, cities, and coordinates. It required several cleaning steps before it could be reliably used for analysis.

---

## Business Context

A marketing agency based in San Francisco wants to contact local boba tea shops for a potential collaboration. They plan to visit **top-rated shops within a 10-mile radius** of their target area. The dataset needed to be cleaned before it could support that decision.

---

## Dataset

| File | Description |
|---|---|
| `data/sf_boba_tea_raw.csv` | Original uncleaned dataset (606 rows) |
| `data/sf_boba_tea_cleaned.csv` | Final cleaned dataset (595 rows) |

**Columns:** `id`, `name`, `rating`, `address`, `city`, `lat`, `long`

---

## Tools Used

- **Microsoft Excel** — data cleaning, filtering, Text to Columns, COUNTIF

---

## Problems Identified & Fixed

### 1. Duplicate Rows
**Problem:** 3 fully identical rows existed in the dataset, inflating shop counts.  
**Fix:** Used Excel's Remove Duplicates across all columns to ensure only truly identical rows were removed.  
**Rows removed:** 3

### 2. Invalid Ratings
**Problem:** 9 shops had Yelp ratings greater than 5 (Yelp's maximum), ranging from 5.2 to 9.2. These would skew average rating calculations and mislead analysis.  
**Decision:** Deleted the entire rows rather than replacing with an assumed value.  
**Reasoning:** Replacing invalid ratings with an arbitrary value (e.g., 5) fabricates data without evidence. Since we cannot verify the true ratings without returning to the source, deletion is the more honest and defensible approach. See [cleaning log](docs/cleaning_log.md) for full reasoning.  
**Rows removed:** 9

### 3. Combined Lat-Long Column
**Problem:** Latitude and longitude were stored in a single column as a combined string (e.g., `37.56295-122.010039999999`), making the data unusable for mapping tools that require separate numeric fields.  
**Fix:** Used Text to Columns with `-` as delimiter to split into two columns, then multiplied longitude values by -1 to restore negative signs (California longitudes are negative).  
**New columns:** `lat`, `long`

### 4. Empty Rows
**Problem:** After cleaning, several rows had no location data (lat/long = 0 or blank).  
**Fix:** Deleted these rows — a shop with no coordinates cannot be mapped or visited.  
**Rows removed:** ~1

---

## Key Decisions & Reasoning

**Why delete invalid ratings instead of replacing them?**  
The course suggested replacing invalid ratings with 5 as a practical workaround. However, this approach fabricates data — we have no evidence that the true rating is 5. A shop with a data entry error of 9.2 could have a true rating of 3.5 or 4.0. Replacing it with 5 could mislead the agency into visiting a lower-quality shop. Deleting preserves data integrity at the cost of a small reduction in dataset size (9 rows out of 606).

**Why use all columns for duplicate detection?**  
Using only the `id` or `name` column risks removing rows that share an ID but have different data — which could be a legitimate second location or corrected entry. Matching on all columns ensures only truly identical records are removed.

---

## Results

| Metric | Before | After |
|---|---|---|
| Total rows | 606 | 595 |
| Duplicate rows | 3 | 0 |
| Invalid ratings (>5) | 9 | 0 |
| Lat-long format | Combined string | Separate numeric columns |

---

## Key Takeaways & Analytical Decisions

- Always inspect data before deleting — filtering first reveals patterns that inform better decisions
- COUNTIF is a powerful verification tool: establish a baseline count before cleaning, then verify after
- The same 3-step pattern applies to all formula-based cleaning: apply in helper column → paste as values → delete helper
- Data cleaning decisions should be documented and defensible, not just technically correct

---

## Certificate Context

This project is part of the **Google Data Analytics Professional Certificate** on Coursera — specifically the *Data Cleaning* module. The techniques practiced include filtering, Remove Duplicates, COUNTIF, and Text to Columns in Microsoft Excel.
