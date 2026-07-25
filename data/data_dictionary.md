# Data Dictionary — Margin Watch: E-Commerce KPI

Source file: `ecommerce_portfolio_raw_data.xlsx` (synthetic, generated for portfolio use)
Tables: `Orders`, `Customers`, `Products`

---

## Table: Orders
**Grain:** one row per order line item (a single order can span multiple rows if it contains multiple products). ~9,000 rows.

| Column | Type | Description | Notes / Known Issues |
|---|---|---|---|
| `Order ID` | Text | Unique identifier for the order (shared across all lines in the same order). Format: `ORD-XXXXXX`. | One order can have multiple `Order Line ID`s. |
| `Order Line ID` | Text | Unique identifier for this specific line item. Format: `{Order ID}-{line number}`. | Primary key for this table. |
| `Order Date` | Text (should be Date) | Date the order was placed. | Stored in **4 different formats** (`YYYY-MM-DD`, `MM/DD/YYYY`, `DD-MM-YYYY`, `Month DD, YYYY`). Must be standardized before analysis. |
| `Ship Date` | Text (should be Date) | Date the order shipped. | Same multi-format issue as `Order Date`. Missing (`null`) for cancelled orders and some pending/processing orders. |
| `Customer ID` | Text | Foreign key to `Customers.Customer ID`. | ~2% missing values. |
| `Product ID` | Text | Foreign key to `Products.Product ID`. | — |
| `Category` | Text | Product category at time of order (denormalized copy from Products). | Inconsistent casing, stray whitespace, and typos present (e.g. `Electroncis`, `Home and Kitchen`, `ELECTRONICS`). Standardize against `Products.Category` before analysis. |
| `Sub-Category` | Text | Product sub-category (denormalized copy from Products). | — |
| `Quantity` | Number | Units ordered on this line. | Can be **negative** (used to represent returns on some rows — not consistently applied, see Order Status). A few extreme outlier values exist (fat-finger entry, e.g. qty in the hundreds) — investigate before treating as real. |
| `Unit Price` | Number (some stored as Text) | Price per unit at time of sale. | ~1.5% of values stored as text with a `$` prefix (e.g. `"$45.20"`) instead of numeric. ~1.5% missing. |
| `Discount` | Decimal (0-1) | Discount applied to this line, as a fraction (e.g. `0.15` = 15% off). | ~2% missing — treat missing as 0 or investigate case-by-case. |
| `Cost Price` | Number | Unit cost of the product (denormalized copy from Products). | Use with `Unit Price` and `Quantity` to calculate profit. |
| `Payment Method` | Text | How the order was paid for. Values: `Credit Card`, `PayPal`, `Debit Card`, `Cash on Delivery`, `Gift Card`. | — |
| `Ship Mode` | Text | Shipping speed selected. Values: `Standard`, `Express`, `Same Day`, `Economy`. | — |
| `Order Status` | Text | Current status of the order. Values: `Delivered`, `Cancelled`, `Returned`, `Pending`, `Processing`. | Inconsistent casing and leading whitespace present (e.g. `" returned"`, `PENDING`). Standardize before grouping/filtering. |
| `Shipping Cost` | Number | Cost to ship this line item. | `0` for cancelled orders. ~2% missing on other statuses. |

---

## Table: Customers
**Grain:** one row per customer. ~1,815 rows (includes 15 intentional duplicate records — see below).

| Column | Type | Description | Notes / Known Issues |
|---|---|---|---|
| `Customer ID` | Text | Unique identifier. Format: `C-XXXXX`. | 15 IDs appear **twice**, with the name in different casing on the duplicate row (simulates a duplicate CRM entry). Decide how to de-duplicate. |
| `Customer Name` | Text | Full name. | — |
| `Email` | Text | Customer email address. | ~4% missing. |
| `Signup Date` | Date | Date the customer registered. | Stored consistently as a proper date (no format issue on this table). |
| `Segment` | Text | Customer type. Values: `Consumer`, `Corporate`, `Home Office`. | — |
| `Country` | Text | Customer's country. | — |
| `City` | Text | Customer's city. | — |

---

## Table: Products
**Grain:** one row per product (product catalog). ~189 rows.

| Column | Type | Description | Notes / Known Issues |
|---|---|---|---|
| `Product ID` | Text | Unique identifier. Format: `P-XXXX`. | Primary key; joins to `Orders.Product ID`. |
| `Product Name` | Text | Product display name (auto-generated). | Cosmetic only — not meaningful for analysis beyond display. |
| `Category` | Text | One of 8 categories: `Electronics`, `Clothing`, `Home & Kitchen`, `Beauty & Personal Care`, `Sports & Outdoors`, `Toys & Games`, `Books & Stationery`, `Groceries`. | This is the **clean, canonical** version — use it to standardize the messy `Category` column in `Orders`. |
| `Sub-Category` | Text | More specific grouping within category. | — |
| `Cost Price` | Number | What the business pays for one unit. | — |
| `List Price` | Number | Standard selling price before discount. | Should generally match `Orders.Unit Price` before discount is applied — useful cross-check during cleaning. |
| `Supplier` | Text | Supplier/vendor name. | — |

---

## Relationships

```
Customers.Customer ID  ──< Orders.Customer ID
Products.Product ID    ──< Orders.Product ID
```

- One customer can have many order lines.
- One product can appear on many order lines.
- `Orders.Category`, `Orders.Sub-Category`, and `Orders.Cost Price` are denormalized (copied) from `Products` at the time of the order — they exist in `Orders` for convenience but should be validated against `Products` rather than trusted blindly, since that's where the messiness was introduced.

---

## Summary of Data Quality Issues (for your cleaning log)

| # | Issue | Table.Column | Suggested Fix |
|---|---|---|---|
| 1 | Multiple date formats | `Orders.Order Date`, `Orders.Ship Date` | Parse all formats, convert to a single consistent date type |
| 2 | Inconsistent casing / typos | `Orders.Category`, `Orders.Order Status` | Trim whitespace, standardize casing, map typos to canonical values (cross-check against `Products.Category`) |
| 3 | Numbers stored as text | `Orders.Unit Price` | Strip `$`, convert to numeric |
| 4 | Missing values | `Orders.Customer ID`, `Orders.Unit Price`, `Orders.Discount`, `Orders.Ship Date`, `Orders.Shipping Cost`, `Customers.Email` | Decide per-column: impute, flag as unknown, or exclude from relevant KPI |
| 5 | Duplicate rows | `Orders` (~1.5% of rows) | De-duplicate on `Order Line ID` |
| 6 | Duplicate customer records | `Customers.Customer ID` (15 IDs) | De-duplicate, keep one canonical name per ID |
| 7 | Outlier values | `Orders.Quantity` | Investigate extreme values, cap or exclude if confirmed data entry errors |
| 8 | Inconsistent returns convention | `Orders.Quantity` vs `Orders.Order Status` | Standardize how returns are represented (negative qty vs. status flag alone) |
