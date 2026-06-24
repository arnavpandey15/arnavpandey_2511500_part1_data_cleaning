# Retail Sales Data Cleaning & Business Reporting Project

## 1. Problem Summary
The retail company exported order-level sales data from multiple disconnected internal systems. This raw dataset was heavily polluted with data entry errors, inconsistent text formatting, invalid date structures, duplicate records, missing fields, invalid discount entries, and mathematical mismatches. The business needed a clean, validated, and analysis-ready version of this data, alongside summarized business reports, to facilitate accurate management review and decision-making.

## 2. Dataset Description
The dataset contains order-level transactional data, structured around customer purchases and logistics. Key data categories include:
* **Customer & Geography:** `customer_name`, `segment`, `region`, `state`, `city`.
* **Product Details:** `category`, `sub_category`.
* **Logistics & Status:** `ship_mode`, `order_date`, `ship_date`, `order_status`, `payment_status`.
* **Financial Metrics:** `quantity`, `unit_price`, `discount`, `sales`, `cost`, `profit`.

## 3. Tools Used
* **Python (Pandas, NumPy, OpenPyXL):** Used for automated, rule-based data cleaning, date parsing, mathematical validation, and heavy-lifting transformations.
* **Microsoft Excel:** Used for final data presentation, conditional formatting, and PivotTable summary reporting.

## 4. Cleaning Steps Performed
* **Raw Data Preservation:** Created a strict separation between the `Raw_Data` and `Cleaned_Data` tabs to ensure a non-destructive workflow.
* **Text Standardization:** Applied `TRIM` and `PROPER` text casing equivalent logic to 10 text fields to remove leading/trailing/multiple spaces and standardizing capitalization.
* **Typo Correction:** Merged known category misspellings (e.g., `Off. Supplies` into `Office Supplies`) and ship modes using a dictionary mapping.
* **Date Parsing:** Standardized `order_date` and `ship_date` into a uniform `YYYY-MM-DD` format, converting invalid strings into `NaT`.
* **Duplicate Handling:** Removed exact row duplicates and flagged conflicting `order_id` duplicates to prevent data loss.
* **Calculated Fields:** Re-engineered financial columns by creating `cleaned_discount`, calculating a true `calculated_sales`, `calculated_profit`, `profit_margin`, and an automated `shipping_delay_days`.

## 5. Business Rules Applied
* **Missing Geographies:** Any missing `region` or `ship_mode` was filled with `Unknown` and flagged.
* **Discount Logic:** Missing discounts were safely imputed as `0.0` if other sales logic held true. Negative discounts and discounts > 100% were overridden as `0.0` for mathematical integrity and flagged as Invalid.
* **Date Sequencing:** Any order where the `ship_date` occurred chronologically before the `order_date` was flagged as an `Invalid shipping record`.
* **Completed Sales Filtering:** Orders with a status of Cancelled, Failed, Refunded, or Returned were excluded from the completed sales summaries via a boolean helper column (`is_valid_completed_sale`). Refunded orders were extracted to their own dedicated summary sheet.

## 6. Summary of Data Quality Issues Found
* **Inconsistent Formatting:** Rampant extra spaces and capitalization errors across string fields [cite: 3].
* **Missing Data:** Null values detected in critical logistical and financial columns [cite: 3].
* **Date Formatting & Chronology:** Mixed date formats, unrecognizable text strings, and impossible ship dates [cite: 3].
* **Duplicates:** Pure identical system glitches alongside conflicting records sharing the same Order ID [cite: 3].
* **Financial Errors:** Impossible negative discounts, discounts over 100%, and mismatches between the raw system sales/profit and the actual calculated formulas [cite: 3].

## 7. Summary of Final Pivot Reports
The `outputs/pivot_summary.xlsx` file includes 6 distinct reports:
1. **Region Summary:** Total valid sales and profit categorized by Region.
2. **Category Summary:** Sales and profit sorted descending by Category and Sub-Category.
3. **Ship Mode Summary:** Valid order counts sorted by shipping method.
4. **Segment Margin:** True aggregate profit margin percentages broken down by Customer Segment.
5. **Failed Orders Region:** Counts of only Cancelled, Failed, Refunded, or Returned orders grouped by region.
6. **Monthly Trend:** A chronological timeseries of sales and profit by Year and Month.

## 8. Key Business Insights
Based on the cleaned pivot summaries:
* **Top Performing Region:** The **South** region leads overall sales volume at ~$1.60M, closely followed by the **West** (~$1.50M).
* **Category Dominance:** **Technology** (driven heavily by Copiers and Phones) and **Furniture** (Chairs) are the highest revenue-generating sub-categories.
* **Sales Volatility:** Early 2024 showed noticeable fluctuation, with a strong peak in May (~$336K) following a significant dip in March (~$207K).

## 9. Assumptions and Limitations
* **Rounding Tolerance:** A $0.05 tolerance was assumed when flagging calculation mismatches to account for floating-point/rounding quirks [cite: 3].
* **Discount Interpretations:** Discounts > 1.0 (100%) were assumed to be typographical or system errors and nullified for calculation purposes [cite: 3].
* **Static Error Mapping:** The typo correction mapping is hardcoded; entirely new misspellings in future exports will bypass the current cleaning logic [cite: 3].
* **No External Validation:** Customer IDs and Names were not validated against an external "Customer Master" database [cite: 3].

## 10. Screenshots Included
*(Note: Replace these placeholders with actual screenshots from the final output directories prior to final submission)*

* ![Data Quality Audit](assets/dq_report_screenshot.png)
* ![Pivot Summary Report](assets/pivot_summary_screenshot.png)
* ![Cleaned Data Excel Snippet](assets/cleaned_data_screenshot.png)
