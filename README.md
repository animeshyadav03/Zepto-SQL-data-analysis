# 🛒 Zepto Inventory SQL Data Analysis Project

A SQL-based data analysis project exploring Zepto's product inventory — a
real e-commerce/quick-commerce catalog dataset — from raw data cleaning
through business-focused analysis.

This project covers the full analyst workflow: setting up a raw dataset,
exploring it for quality issues, cleaning it, and writing SQL queries to
answer real pricing, stock, and revenue questions.

## 📌 Project Overview

The goal was to work through this dataset the way an analyst would on the
job:

- ✅ Set up the raw inventory dataset in a SQL table
- ✅ Explore the data for structure, nulls, and inconsistencies
- ✅ Clean the data — remove invalid rows, fix unit formatting
- ✅ Write business-driven SQL queries covering pricing, discounting, stock
  availability, and revenue

## 📁 Dataset Overview

Sourced from Kaggle: [Zepto Inventory Dataset](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset)

Each row represents a product SKU. 3,732 rows across 14 categories.

**Columns:**
| Column | Description |
|---|---|
| `sku_id` | Unique identifier per row (added as primary key) |
| `name` | Product name |
| `category` | Product category (Fruits & Vegetables, Snacks, Beverages, etc.) |
| `mrp` | Maximum Retail Price (converted from paise to ₹) |
| `discountPercent` | Discount applied on MRP |
| `discountedSellingPrice` | Final price after discount (converted to ₹) |
| `availableQuantity` | Units available in inventory |
| `weightInGms` | Product weight in grams |
| `outOfStock` | Boolean flag for stock availability |
| `quantity` | Units per package |

## 🔧 Project Workflow

### 1. Table Creation
```sql
CREATE TABLE zepto (
  sku_id INT AUTO_INCREMENT PRIMARY KEY,
  category VARCHAR(120),
  name VARCHAR(150) NOT NULL,
  mrp DECIMAL(8,2),
  discountPercent DECIMAL(5,2),
  availableQuantity INT,
  discountedSellingPrice DECIMAL(8,2),
  weightInGms INT,
  outOfStock BOOLEAN,
  quantity INT
);
```
> Note: MySQL's `BOOLEAN` is an alias for `TINYINT(1)`, so `outOfStock` is
> stored as `0`/`1` under the hood — this works transparently in queries.

### 2. 🔍 Data Exploration
- Counted total records (3,732 rows)
- Checked sample rows to understand structure
- Checked for null values across all columns — **none found**
- Listed all distinct product categories (14 total)
- Compared in-stock vs. out-of-stock counts
- Checked for product names appearing multiple times (multiple SKUs per
  product — different sizes/discounts, as expected in a real catalog)

### 3. 🧹 Data Cleaning
- Found and removed 1 row with `mrp = 0` (invalid entry)
- Converted `mrp` and `discountedSellingPrice` from paise to rupees

### 4. ⚠️ Data Quality Finding
While aggregating by category, discovered that every product in the dataset
is **duplicated under a second category label** with otherwise identical
data — e.g. every "Munchies" row has an exact duplicate under "Cooking
Essentials," and "Personal Care" duplicates "Paan Corner." This roughly
halves the true unique product count and means naive category-level SUMs
(revenue, weight) double-count these pairs unless de-duplicated first.

### 5. 📊 Business Insights
- Top 10 best-value products by discount percentage
- High-MRP products currently out of stock
- Estimated revenue per category
- Products with MRP > ₹500 but minimal discount (<10%)
- Top 5 categories by average discount
- Price-per-gram ranking to find best value-for-money items
- Products grouped into Low / Medium / Bulk weight tiers
- Total inventory weight per category

## 🔑 Key Findings

- **Fruits & Vegetables has the highest average discount (15.46%)** — more
  than 4 points ahead of the next category, Meats, Fish & Eggs (11.03%).
- **12.1% of SKUs (453 of 3,732) are out of stock**, including several
  high-MRP staples like Patanjali Cow's Ghee (₹565) and MamyPoko Pants
  Extra Large diapers (₹399).
- **39 products priced above ₹500 carry under 10% discount** — mostly
  cooking oils, ghee, and personal care items, suggesting these categories
  see less aggressive promotion even at higher price points.
- **Everyday staples dominate the best value-per-gram ranking** — Onion,
  Potato, Beetroot, and salt brands all rank in the top 10.
- **Most SKUs (3,392 of 3,731) fall in the "Low" weight tier** (under 1kg);
  only 46 are classified "Bulk" (5kg+).

## 📈 Visualization
![Category discount and stock-out comparison](category_analysis.png)

Left: average discount % by category. Right: out-of-stock rate % by category.

## 💡 Recommendations
1. De-duplicate paired categories before running any revenue or weight
   aggregation, since several categories are exact duplicates of each other.
2. Investigate stock-outs on high-MRP, high-frequency staples (ghee, atta,
   diapers) — availability gaps here likely cost more in lost sales than
   low-MRP impulse items.
3. Review discounting strategy on premium personal care and cooking oil
   products, where discount levels are notably lower than other categories.

## 🛠️ How to Use This Project
```bash
git clone https://github.com/animeshyadav03/Zepto-SQL-data-analysis.git
cd Zepto-SQL-data-analysis
```
1. Open `queries.sql` — contains table creation, data exploration, cleaning,
   and business analysis queries in order.
2. Load `zepto_v2.csv` into a MySQL database. Options:
   - **MySQL Workbench**: use the Table Data Import Wizard.
   - **Command line**, after creating the `zepto` table:
     ```sql
     LOAD DATA LOCAL INFILE 'zepto_v2.csv'
     INTO TABLE zepto
     FIELDS TERMINATED BY ','
     ENCLOSED BY '"'
     LINES TERMINATED BY '\n'
     IGNORE 1 ROWS
     (category, name, mrp, discountPercent, availableQuantity,
      discountedSellingPrice, weightInGms, outOfStock, quantity);
     ```
     If you hit a "local infile" error, enable it first:
     `SET GLOBAL local_infile = 1;` (server-side) and connect with
     `--local-infile=1` (client-side).
3. Run `queries.sql` against the loaded table to reproduce the analysis.

## 📁 Project Structure
```
Zepto-SQL-data-analysis/
│
├── zepto_v2.csv
├── queries.sql
├── category_analysis.png
├── README.md
├── LICENSE
```
