# Data Cleaning & Validation Log

## 1. List of Issues Found in Raw Data
During the initial assessment of the `raw_orders.xlsx` dataset, the following issues were identified:
* **Inconsistent Text Formatting:** Leading, trailing, and multiple spaces within strings, along with inconsistent capitalization (e.g., `FIRST CLASS` vs `First Class`).
* **Unwanted Special Characters:** Underscores used instead of spaces in some text fields.
* **Category Misspellings:** Similar categories written differently (e.g., `Off. Supplies`, `Office Supply`, `Off Supplies`).
* **Missing Values:** Blank or null values detected in critical logistical fields (`region`, `ship_mode`), dates, and financial fields (`discount`).
* **Date Format Issues:** Dates formatted inconsistently (e.g., `21 Jul 2024`, `07/27/2024`, `2024-02-20`) and text strings that could not be parsed as dates.
* **Chronological Errors:** Instances where the `ship_date` occurred *before* the `order_date`.
* **Duplicates:** Both exact row duplicates and conflicting duplicate `order_id` records (same ID, different details).
* **Invalid Financial Rules:** Discounts entered as negative numbers or excessively high values (>100%).
* **Calculation Mismatches:** Discrepancies between the raw `sales`/`profit` figures and the actual mathematical calculation of `(Quantity * Unit Price * (1 - Discount))` and `(Sales - Cost)`.
* **Order Status Mixes:** Records representing Cancelled, Failed, Refunded, or Returned orders mixed in with completed sales.

---

## 2. Cleaning Actions Performed
* **Preservation:** The original dataset was preserved completely untouched in a `Raw_Data` tab.
* **Text Standardization:** Applied a multi-step string cleaning function to 10 text columns using Python/Excel equivalents (Strip, Regex space removal, Title casing, and character substitution).
* **Typo Correction:** Standardized known misspellings in `category` and `ship_mode` using a targeted dictionary mapping.
* **Date Parsing:** Converted all date fields into a standardized `YYYY-MM-DD` format using datetime coercion. Invalid strings were converted to empty/NaT and heavily flagged.
* **Duplicate Handling:** Scanned the dataset to drop pure identical duplicates, while retaining and flagging conflicting duplicate IDs.
* **Calculated Fields Engineering:** Appended mathematically sound columns: `cleaned_discount`, `calculated_sales`, `calculated_profit`, `profit_margin`, `shipping_delay_days`, `order_month`, and `order_year`.
* **Quality Flagging:** Consolidated all row-level issues into a single, highly readable `data_quality_flag` column, categorized as `Clean`, `Warning`, or `Invalid`.

---

## 3. Business Rules Applied
* **Missing Region/Ship Mode:** Filled with the string `"Unknown"` and flagged as a Warning.
* **Missing Discount:** Treated as `0.0` *only* if `quantity`, `unit_price`, and `sales` were valid. Otherwise, flagged as Invalid.
* **Invalid Discounts:** Negative discounts and discounts >1.0 were overridden as `0.0` in the `cleaned_discount` column for calculation purposes, and heavily flagged as Invalid.
* **Date Sequencing:** Ship dates occurring before order dates were explicitly flagged as `"Invalid shipping record"`.
* **Excluded Statuses:** Any order or payment marked as Cancelled, Failed, Refunded, or Returned was marked `is_valid_completed_sale = FALSE` to strictly exclude them from final revenue PivotTables.
* **Refund Separations:** Refunded and Returned records were successfully extracted into a standalone `Refunded_Orders_Summary` tab.

---

## 4. Assumptions Made
* **Tolerance for Calculation Mismatches:** When comparing raw system sales/profit against our calculated formulas, I assumed a `$0.05` rounding tolerance. Any mismatch below 5 cents was considered a floating-point quirk rather than a system error.
* **Discount Interpretations:** I assumed any discount greater than `1.0` (100%) was a typographical error by the user (e.g., typing "20" instead of "0.20") or a system glitch, making the record invalid.
* **Conflicting Duplicates:** I assumed that automatically deleting rows with identical `order_id`s but conflicting details could result in the loss of valid line-item data. Thus, they were retained but heavily flagged.

---

## 5. Records Removed
* **Exact Duplicate Rows:** Any record that matched another record across *every single column* perfectly was assumed to be a system export glitch and was permanently **removed** from the `Cleaned_Data` table (keeping only the first instance).

---

## 6. Records Flagged
Instead of deleting problematic rows, they were retained and explicitly tagged in the `data_quality_flag` column. Flags included:
* `Warning: Missing region filled as Unknown`
* `Warning: Missing ship_mode filled as Unknown`
* `Warning: Missing discount treated as 0`
* `Warning: Conflicting duplicate order_id`
* `Invalid: Negative discount`
* `Invalid: Discount above allowed range`
* `Invalid: Missing order_date` / `Missing ship_date`
* `Invalid: Invalid ship_date text`
* `Invalid: ship_date is earlier than order_date`

---

## 7. Limitations of the Cleaning Process
* **Static Typo Mapping:** The mapping used to fix category spellings (e.g., `Off. Supplies` -> `Office Supplies`) is hardcoded. If a new, unknown typo appears in next month's export (e.g., `Ofc Suplies`), the script will not automatically catch it without an update.
* **No Customer Master Validation:** The cleaning process assumes `customer_name` and `customer_id` align correctly. There is no external database lookup to verify if "Karan Malhotra" actually corresponds to "CUST-4655".
* **Aggressive Coercion:** Coercing highly corrupted date or numeric text directly into `NaT` or `0` cleans the data for reporting but destroys the original corrupted string in those specific mathematical columns. (However, this is mitigated by preserving the `Raw_Data` tab).
