# Data Cleaning Log
**Project:** Part 1 – Business Data Cleaning, Validation & Excel Reporting  
**Dataset:** raw_orders.xlsx (retail order-level sales data)  
**Analyst:** Data Cleaning Pipeline  

---

## 1. Issues Found

| # | Issue Type | Description | Count |
|---|---|---|---|
| 1 | Exact Duplicates | Identical rows across all columns | 20 |
| 2 | Conflicting Duplicate IDs | Same order_id with different field values | 12 |
| 3 | Mixed Date Formats | Dates in 7+ different formats (dd MMM YYYY, MM/dd/YYYY, dd-MM-YYYY, YYYY-MM-DD etc.) | ~200+ |
| 4 | Ship Before Order | ship_date is earlier than order_date | 20 |
| 5 | Missing Region | Region field blank | 25 |
| 6 | Missing Ship Mode | Ship mode field blank | 20 |
| 7 | Missing Discount | Discount cell empty | ~30 |
| 8 | Negative Discount | Discount value is negative (e.g., -0.19, -0.23) | 15 |
| 9 | High Discount | Discount above 50% (e.g., 0.65, 70%) | 15 |
| 10 | Discount as Percentage String | Value stored as "70%" instead of 0.70 | 1 |
| 11 | Case Inconsistencies | ALL CAPS, lowercase, Title Case mixed across text fields | ~50+ |
| 12 | Extra/Leading/Trailing Spaces | Whitespace around values (e.g., "  North ", "  Small Business ") | ~30+ |
| 13 | Sales Calculation Mismatch | Reported sales doesn't match quantity × unit_price × (1 – discount) | Several |
| 14 | Cancelled Orders in Data | Cancelled/Returned orders present in dataset | ~100+ |
| 15 | Failed Payment Records | Failed payment status records present | ~50+ |

---

## 2. Cleaning Actions Performed

### Text Fields Cleaned
- Applied `Trim()` to remove leading/trailing whitespace
- Applied `.Proper()` for consistent Title Case
- 
- Fields cleaned: `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`

### Date Standardisation
Applied "Text to Columns" parsing tool to resolve mixed date strings and set the explicit workbook format to `YYYY-MM-DD`.
*

### Duplicate Handling
- **Exact duplicates (20):** Removed; kept first occurrence
- **Conflicting order_id duplicates (12):** Removed duplicated rows; first occurrence kept; logged separately

### Missing Values
- **Missing region (25):** Filled with `Unknown` as per business rule; flagged in quality report
- **Missing ship_mode (20):** Filled with `Unknown`; flagged in quality report
- **Missing discount (~30):** Set to `0.0` only when `quantity` and `unit_price` are both valid; otherwise left as NaN

---

## 3. Business Rules Applied

| Rule | Applied Action |
|---|---|
| Missing region | Filled as `Unknown`; flagged in data_quality_flag |
| Missing ship_mode | Filled as `Unknown`; flagged in data_quality_flag |
| Missing discount | Treat as 0.0 only if quantity and unit_price are valid |
| Negative discount | Flagged as `Invalid` in data_quality_flag |
| Discount > 50% | Flagged as `Invalid` in data_quality_flag |
| Discount as % string (e.g., "70%") | Converted to decimal (0.70) |
| Cancelled orders | Excluded from completed-sales pivot summaries |
| Failed payments | Excluded from completed-sales pivot summaries |
| Refunded orders | Summarised separately in pivot report |
| Ship date before order date | Flagged as `Invalid` in data_quality_flag |

---

## 4. Calculated Columns Added

| Column | Formula/Logic |
|---|---|
| `cleaned_discount` | Standardised discount (handles %, negatives, missing) |
| `calculated_sales` | `quantity × unit_price × (1 – cleaned_discount)` |
| `calculated_profit` | `calculated_sales − cost` |
| `profit_margin` | `calculated_profit / calculated_sales` |
| `shipping_delay_days` | `ship_date − order_date` in days |
| `order_month` | Extracted month from `order_date` |
| `order_year` | Extracted year from `order_date` |
| `data_quality_flag` | `Clean` / `Warning` / `Invalid` based on rule violations |

---

## 5. Records Removed / Flagged

| Action | Count | Reason |
|---|---|---|
| Exact duplicates removed | 20 | Identical rows |
| Conflicting IDs removed | 12 | Duplicate order_id with different values |
| Flagged as Invalid | 50 | Negative/high discount, ship before order, invalid date |
| Flagged as Warning | 64 | Missing region/ship_mode, sales mismatch |
| Total removed from analysis | 32 | (exact dups + conflicting) |

---

## 6. Assumptions Made


1. **Discount >50% treated as invalid:** Business context suggests max legitimate discount is 50%. Values above this are flagged but not removed.
2. **Negative discounts:** Treated as data entry errors; flagged as Invalid.
3. **"70%" string discount:** Converted to 0.70 (decimal) for calculation.
4. **Cancelled + Paid orders:** Retained in cleaned data but excluded from completed-sales summaries. Sales should not be credited where orders were cancelled.
5. **Refunded orders:** Retained; separately summarised.
6. **Sales mismatch:** When reported `sales` differs from `calculated_sales` by more than ₹1, flagged as Warning. The `calculated_sales` column is used for all analysis.
7. **Missing cost:** If cost is missing, `calculated_profit` will be NaN. No imputation performed.

---

## 7. Limitations

- Date parsing may misinterpret ambiguous formats (e.g., months vs. days)
- Some sales mismatches may be due to rounding in the source system, not errors
- Product name field not cleaned beyond title-casing (no deduplication of similar names)
- Customer name variants (e.g., "PRIYA MENON" vs "Priya Menon") are normalised but not de-duplicated across customer_id
- Outlier detection on unit_price/quantity was not performed
- The cleaning_log reflects automated decisions; manual review is recommended for the 50 Invalid-flagged records
