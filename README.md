# PrabhatKumar_bitsom_ba_2511037_part1_data_cleaning
## Problem Summary

A retail company exported order-level sales data from multiple internal systems. The raw dataset (`raw_orders.xlsx`) contained 932 records with numerous data quality issues including mixed date formats, duplicate records, missing values, invalid discounts, case inconsistencies, and sales calculation mismatches. The objective was to clean, validate, and produce analysis-ready outputs with supporting reports.

---

## Dataset Description

| Attribute | Value |
|---|---|
| File | raw_orders.xlsx |
| Sheet | raw_orders |
| Raw Records | 932 |
| Columns | 21 (order_id, dates, customer info, product info, financials, statuses) |
| Date Range | 2024 – 2025 |
| Regions | North, South, East, West |
| Categories | Furniture, Technology, Office Supplies |

---

## Tools Used

Excel 

---

## Cleaning Steps Performed

1. **Preserved raw data** – `raw_orders.xlsx` kept unchanged; all work done in `cleaned_orders.xlsx`
2. **Cleaned text fields** – stripped whitespace, normalised to Title Case, collapsed extra spaces for 10 text columns
3. **Standardised dates** – parsed 7+ date formats; both dates exported as `YYYY-MM-DD`
4. **Removed 20 exact duplicates** – kept first occurrence
5. **Flagged and removed 12 conflicting order IDs** – duplicate order_ids with different values
6. **Filled missing region/ship_mode** with `Unknown` per business rules
7. **Cleaned discounts** – converted % strings, handled negatives, set missing to 0.0 when valid
8. **Added 7 calculated columns** – see below
9. **Applied data_quality_flag** to every record

---

## Business Rules Applied

| Rule | Action |
|---|---|
| Missing region | Fill `Unknown`, flag Warning |
| Missing ship_mode | Fill `Unknown`, flag Warning |
| Missing discount | Set 0.0 if quantity/unit_price valid |
| Negative discount | Flag Invalid |
| Discount > 50% | Flag Invalid |
| Ship date before order date | Flag Invalid |
| Cancelled/Failed orders | Excluded from completed-sales pivot summaries |
| Refunded orders | Summarised separately |

---

## Summary of Data Quality Issues Found

| Issue | Count |
|---|---|
| Exact duplicate rows | 20 |
| Conflicting duplicate order IDs | 12 |
| Missing region (filled Unknown) | 25 |
| Missing ship mode (filled Unknown) | 20 |
| Negative discounts | 15 |
| Discounts > 50% | 15 |
| Ship date before order date | 20 |
| Sales calculation mismatches | ~30 |
| Invalid/missing dates | <10 |
| **Final Clean records** | **786** |
| **Warning records** | **64** |
| **Invalid records** | **50** |

---

## Summary of Final Pivot Reports

| Sheet | Key Insight |
|---|---|
| Sales & Profit by Region | West leads in sales; South has strong profit margins |
| Sales by Category & Sub-Cat | Technology and Furniture drive highest revenue |
| Orders by Ship Mode | Standard Class and First Class are most used |
| Profit Margin by Segment | Corporate segment shows highest average margin |
| Issue Orders by Region | Returns and cancellations spread across all regions |
| Monthly Sales Trend | Peaks in Q3 and Q4 of both 2024 and 2025 |
| Refunded Orders | Refunds concentrated in South and West regions |

---

## Key Business Insights

1. **West and South regions** together account for the majority of completed sales revenue
2. **furniture *e highest sales value per order on average
3. **Standard Class** shipping is most commonly used, suggesting cost-conscious buyers

4.*Cancelled and failed orders** represent approximately 15% of raw records – impacting revenue reporting if not excluded
