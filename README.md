# Zepto Inventory Analysis

## Overview
SQL-driven analysis of Zepto's product inventory — cleaning the raw catalog,
then answering pricing, discounting, and stock-availability questions across
3,732 SKUs and 14 product categories using PostgreSQL.

## Dataset
[Zepto Inventory Dataset](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset) (Kaggle)

**Columns:** `category`, `name`, `mrp`, `discountPercent`, `availableQuantity`,
`discountedSellingPrice`, `weightInGms`, `outOfStock`, `quantity`

## Tools Used
PostgreSQL, SQL

## Data Cleaning
- Verified no null values across any column.
- Found and removed 1 row with `mrp = 0` (invalid product entry).
- Converted `mrp` and `discountedSellingPrice` from paise to rupees
  (raw values were stored ×100).

## Data Quality Note
While exploring category-level aggregates, every product in the dataset
turned out to be **duplicated under a second category label** with otherwise
identical data (same name, price, stock, weight) — for example, every
"Munchies" row has an exact duplicate under "Cooking Essentials," every
"Packaged Food" row is duplicated under "Ice Cream & Desserts" and
"Chocolates & Candies," and "Personal Care" duplicates "Paan Corner."
This roughly halves the true number of unique products (~1,866 rather than
3,731) and means any category-level SUM (like revenue or total weight) is
double-counted for these paired categories. Worth flagging to anyone
using this dataset for downstream analysis.

## Key Findings

**1. Fruits & Vegetables has the highest average discount (15.46%)**,
more than 4 points ahead of the next category (Meats, Fish & Eggs at 11.03%).
Most other categories average 6-8% discount.

**2. 453 of 3,732 SKUs (12.1%) are out of stock.**

**3. Several premium products are out of stock despite high MRP** — e.g.
Patanjali Cow's Ghee (₹565) and MamyPoko Pants Extra Large diapers (₹399)
were both unavailable at the time of the data snapshot.

**4. 39 products have MRP above ₹500 but less than 10% discount** — mostly
cooking oils, ghee, and personal care items (hair color, shampoo), suggesting
these categories see less aggressive discounting even at higher price points.

**5. Best value-per-gram products are dominated by fresh produce and staples**
— Onion, Potato, Beetroot, and Tata Salt/Aashirvaad Salt all rank in the top
10 cheapest per gram, as expected for everyday household basics.

**6. Most inventory (3,392 of 3,731 SKUs) falls in the "Low" weight tier**
(under 1kg), with only 46 products classified as "Bulk" (5kg+).

## Sample Query Results

**Top 5 categories by average discount %:**
| Category | Avg Discount |
|---|---|
| Fruits & Vegetables | 15.46% |
| Meats, Fish & Eggs | 11.03% |
| Packaged Food | 8.32% |
| Ice Cream & Desserts | 8.32% |
| Chocolates & Candies | 8.32% |

**Products with high MRP but out of stock (MRP > ₹300):**
| Product | MRP |
|---|---|
| Patanjali Cow's Ghee | ₹565 |
| MamyPoko Pants Extra Large Diapers | ₹399 |
| Aashirvaad Atta With Multigrains | ₹315 |
| Everest Kashmiri Lal Chilli Powder | ₹310 |

## Recommendations
1. **De-duplicate paired categories** before running any revenue or weight
   aggregation — as noted above, several categories are exact duplicates and
   will double-count totals if not handled.
2. **Investigate stock-outs on high-MRP staples** (ghee, atta, diapers) —
   these are high-frequency repurchase items where availability gaps likely
   cost more in lost sales than low-MRP impulse items.
3. **Review discounting on premium personal care and cooking oil products**
   — 39 SKUs over ₹500 carry under 10% discount, which may be intentional
   margin protection but is worth confirming against category strategy.

## Project Structure
```
zepto-inventory-analysis/
│
├── data/
│   └── zepto_v2.csv
├── sql/
│   └── queries.sql
├── category_analysis.png
├── README.md
```
