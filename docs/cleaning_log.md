# Data Cleaning Log — SF Boba Tea Shop Dataset

**Analyst:** MD Noornabi

**Date:** March 2026  
**Tool:** Microsoft Excel  
**Dataset:** San Francisco Boba Tea Shop Location Info  
**Source:** Google Data Analytics Certificate

---

## Initial Assessment

**Raw dataset:** 603 rows × 6 columns  
**Columns:** `id`, `name`, `rating`, `address`, `city`, `lat-long`

### Problems Identified Before Cleaning

| # | Issue | Location | Severity |
|---|---|---|---|
| 1 | Duplicate rows | Entire dataset | Medium — inflates counts |
| 2 | Invalid ratings (>5) | Column C (rating) | High — skews analysis |
| 3 | Combined lat-long values | Column F (lat-long) | High — unusable for mapping |
| 4 | Empty rows with no location data | Various rows | Medium — misleading nulls |

---

## Step 1: Remove Duplicate Rows

**Action:** Data → Remove Duplicates → All columns selected  

**Why all columns?**  
Selecting only `id` or `name` risks false positives — two rows sharing an ID but with different data could represent a corrected entry or a second location. Requiring all columns to match ensures only truly identical records are flagged.

**Result:** 3 duplicate rows removed  
**Rows remaining:** 600  

**Verification:** Manually confirmed removed rows were exact copies with no unique data.

---

## Step 2: Identify Invalid Ratings

**Action:** Applied COUNTIF formula in helper cell: `=COUNTIF(C:C,">5")`  

**Baseline count:** 9 rows with ratings > 5  

**Invalid entries found:**

| Shop Name | Invalid Rating |
|---|---|
| gong-cha-fremont | 6.7 |
| boba-queen-fremont | 5.2 |
| infinitea-san-francisco | 5.6 |
| super-cue-cafe-san-francisco-2 | 8.9 |
| t4-san-leandro | 7.4 |
| amor-cafe-and-tea-san-jose | 5.4 |
| ohana-hawaiian-bbq-of-pleasanton | 5.7 |
| che-lo-union-city-2 | 9.2 |
| happy-lemon-sunnyvale-2 | 6.2 |

---

## Step 3: Handle Invalid Ratings

**Decision: Delete entire rows**

**Reasoning:**  
The course suggested replacing invalid values with 5 as a practical workaround. After evaluating this approach, I identified a data integrity concern:

- Yelp ratings are assigned by real users — we cannot assume the "correct" value without checking the source
- Replacing with 5 fabricates data that could actively mislead the marketing agency's shop selection
- A shop with a corrupted rating of 9.2 could have a true rating anywhere from 1.0 to 5.0
- Silent data fabrication is harder to detect and audit than a missing row

**Alternative considered:** Replace with NULL/blank to retain location data while flagging the rating as unknown. This is valid but adds complexity for downstream users who need to handle nulls.

**Final decision:** Delete rows. The dataset has 600 rows — losing 9 (1.5%) is an acceptable tradeoff for data integrity. The agency's goal is to find top-rated shops, so shops with unverifiable ratings should not be included.

**Action:** Filtered rating column for values > 5, selected all 9 rows, deleted entire rows  
**Verification:** Re-ran COUNTIF → returned 0 ✅  
**Rows remaining:** 591  

---

## Step 4: Split Combined Lat-Long Column

**Problem:** Column F contained combined strings like `37.56295-122.010039999999`  
Mapping tools (Tableau, Power BI, Google Maps API) require latitude and longitude as **separate numeric fields**.

**Challenge identified:**  
The `-` character serves dual purpose — as a separator between lat and long, AND as the negative sign for longitude (California longitudes are negative, ~-121 to -122). Using `-` as a delimiter would consume the negative sign.

**Action:** Data → Text to Columns → Delimited → Other: `-`  
**Result:** Two columns produced:
- Column F: `lat` (positive values, e.g., 37.56295) ✅
- Column G: `long` (positive values, e.g., 122.010) ⚠️ — missing negative sign

**Restoring negative longitude:**  
Applied formula in helper column H: `=-G2`, dragged down for all rows  
Pasted as values back into column G (to avoid circular reference)  
Deleted helper column H  

**Why paste as values before deleting?**  
If column H contained `=-G2` and we deleted column G first, all formulas in H would return `#REF!` errors. Pasting as values converts formulas to static numbers, removing the dependency.

**Renamed columns:** F → `lat`, G → `long`  
**Rows remaining:** 591  

---

## Step 5: Remove Empty/Zero Location Rows

**Problem:** After splitting, several rows showed 0 or blank values in lat and long columns — artifacts from previously deleted rows or missing source data.

**Decision:** Delete rows with no location data  
**Reasoning:** The agency's goal is to physically visit shops. A shop with no coordinates cannot be mapped or located. Keeping these rows would add noise without any analytical value.

**Rows removed:** ~4  
**Rows remaining:** 595  

---

## Final Verification

| Check | Result |
|---|---|
| `COUNTIF(C:C,">5")` | 0 ✅ |
| Duplicate rows | 0 ✅ |
| Lat column — all negative? | No (latitude should be positive ~37-38) ✅ |
| Long column — all negative? | Yes (longitude ~-121 to -122) ✅ |
| Empty rows | 0 ✅ |

---

## Summary

| Metric | Before | After |
|---|---|---|
| Total rows | 603 | 595 |
| Duplicate rows | 3 | 0 |
| Invalid ratings | 9 | 0 |
| Lat-long format | Combined string | Separate numeric columns |
| Empty/zero location rows | ~4 | 0 |

---

## Key Takeaways & Analytical Decisions

1. **Filter before delete** — always inspect what you're about to remove before acting
2. **COUNTIF as a QA tool** — establish baseline counts before cleaning, verify after
3. **Question course recommendations** — replacing invalid data with assumed values (e.g., replacing >5 ratings with 5) is a data integrity risk worth flagging
4. **Document decisions, not just actions** — future analysts need to understand *why* a row was deleted, not just that it was
5. **Helper column pattern** — apply formula in helper → paste as values → delete helper. This pattern applies universally to formula-based cleaning in Excel
