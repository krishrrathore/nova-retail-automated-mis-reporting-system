# Nova Retail Profitability & Performance MIS

An Excel-based MIS analysis + Power BI executive dashboard, built on Nova Retail's ERP data, to
help management see where profitability is unstable and act on it.

---

## The problem

Nova Retail's leadership lacked a single reporting view to consistently track profitability
across regions, stores, categories, and promotions. The main challenge was identifying where
performance was weak and where margin volatility, returns, or discounting needed attention.

When I first looked at this project, the assumption was that margin had declined from 15.4% to
10.2% over two years. I checked that against the actual Orders, Order_Items, and Products data,
and it didn't hold up. Margin didn't decline in a straight line — it went 14.09% → 13.37% →
12.24% → 15.92% year by year, and month to month it swings between 5.9% and 19.9% (overall
average is 13.99%). So the real problem isn't a decline. It's volatility — nobody can tell if a
given month is normal or a warning sign, and nobody has a ranked view of which regions, stores,
or promotions are actually causing the swings.

## What I built

I worked with the 15-table ERP export (Orders, Order_Items, Products, Returns, Shipments,
Promotions, Sales_Targets, and so on — Jan 2021 to Dec 2024). I cleaned it, joined it, and built
a proper data model in Power Query, then wrote DAX measures on top so every KPI is calculated
the same way everywhere, instead of each sheet doing its own math.

On top of that model, I built:
- An Excel workbook with the KPI calculations and a recommendations sheet
- A Power BI dashboard on the same data — this is the main executive dashboard for the project

Revenue only counts non-cancelled orders (611,653 of 659,913 order lines). Gross margin is
calculated at the line level: (price charged − unit cost) × quantity.

## How it's built

ERP CSVs → Power Query (cleaning + joins) → data model (fact table for order lines, dimension
tables for region/store/category/promotion/date) → DAX measures → Excel KPI sheets (SUMIFS,
INDEX-MATCH, with calculations linked to the underlying data) → recommendations → Power BI
dashboard on the same source.

I checked every formula in the final workbook — 5,623 formulas across 20 sheets — and there
are zero calculation errors.

### DAX measures I wrote (18 total)

Total Revenue, Total Cost, Gross Margin, Gross Margin %, Order Count, Return Count, Line Count,
Return Rate %, Refund Leakage, Discount Leakage, Catalog Value, Discount Leakage %, Target
Amount, Target Achievement %, Shipment Count, Late Shipment Count, Late Delivery Rate %,
Revenue Growth %.

Full M-code and DAX in [`PowerQuery_DAX_Specification.md`](./PowerQuery_DAX_Specification.md).

---

## Key numbers

| KPI | Value |
|---|---|
| Total Revenue (FY21–24) | ₹1,887.9 Cr |
| Total Gross Margin | ₹264.2 Cr |
| Overall Gross Margin % | 13.99% |
| Monthly margin swing (std dev) | 2.6% |
| Margin range across 48 months | 5.9% – 19.9% |
| Margin change, last 6 months vs prior 6 months (2024) | +3.85 pp (13.71% → 17.56%) |
| Regions below FY24 target | 3 of 12 |
| Categories with above-average return rate | 8 of 20 |
| Categories with above-average discount leakage | 8 of 20 |
| Total discount leakage | ₹206.4 Cr (10.9% of revenue) |
| Total return/refund leakage | ₹66.5 Cr (3.5% of revenue) |
| Promotions worth continuing | 86 of 200 |
| Promotions to review | 66 of 200 |
| Promotions to discontinue | 48 of 200 |
| Stores flagged for priority attention | 30 of 120 |
| Region-courier pairs with above-average late deliveries | 27 of 60 |

---

## How I scored store performance

I built a Store Performance Score comparing each store's last 6 months (Jul–Dec 2024) against
the previous 6 months (Jan–Jun 2024), using 5 metrics:

| Metric | Weight | Why |
|---|---|---|
| Gross Margin % | 30% | This is a margin problem, so margin gets the most weight |
| Margin trend (change vs prior period) | 25% | A store improving or declining matters more than a single snapshot |
| Return rate % | 20% | Returns are a direct, provable ₹ loss |
| Discount leakage % | 15% | Real, but it overlaps with margin %, so I weighted it lower |
| Revenue growth % | 10% | Lowest on purpose — this is about margin, not sales |

The weights are editable cells in the workbook, so the whole score recalculates if you change
them. I didn't use a store-level sales target for this, because the ERP data only has targets at
the region level — I didn't want to make up a store-level number that isn't really there.

---

## What's in the Excel workbook (20 sheets)

- **Cover sheet** — scope and how to use the file
- **8 model sheets** — the cleaned, joined data (region/store/category/promotion breakdowns)
- **8 KPI sheets** — margin, target, category, discount, returns, SLA, promotions, store score
- **2 report sheets** — trend and regional contribution charts
- **Insights sheet** — key findings written as formulas, so they update if the data changes
- **Recommendations sheet** — 352 rows (all regions, categories, stores, and promotions), each
  with an action generated by a formula based on that item's numbers, not typed by hand
- **Data limitations sheet** — what this dataset can't tell us

### Power BI executive dashboard

This is the main dashboard for the project. It has 5 visuals and 6 KPI cards:

**Visuals:**
- Revenue vs Target Trend — monthly Total Revenue against Target Amount, Jan 2021 to Dec 2024
- Target Achievement by Region — bar chart by region
- Category Revenue vs Gross Margin % — bubble chart, categories color-coded
- Return Rate by Category — bar chart
- Top 10 Promotions by Discount Leakage % — bar chart

**KPI cards:** Total Revenue, Gross Margin %, Return Rate %, On-Time Delivery %, Average Order
Value, Repeat Customer Rate %

I checked the KPI cards against the source data directly:

| Card | Power BI shows | Calculated from source | Match? |
|---|---|---|---|
| Total Revenue | 19bn | ₹18.88bn | Matches |
| Average Order Value | 67.88K | ₹67,881 | Matches |
| Return Rate % | 3.71% | 3.68% | Matches |
| On-Time Delivery % | 90.74% | 90.69% | Matches |
| Repeat Customer Rate % | 65.54% | 65.54% | Matches, exact |
| Gross Margin % | 7.21% | 13.99% (from the Excel workbook) | **Does not match** |

The Gross Margin % card in Power BI does not agree with the Excel workbook's verified figure —
it's off by close to half. I haven't been able to check the DAX behind this measure yet, so I'm
not fixing or explaining the number here. This needs to be corrected in the Power BI file
before this project is presented anywhere, and the Excel figure (13.99%) should be treated as
the correct one until the Power BI measure is fixed.

---

## Questions this project answers

1. Is margin actually declining, or just volatile?
2. Which regions are missing their target, and by how much?
3. Which categories bring in revenue but don't make much profit, and why?
4. Which promotions are worth keeping, and which ones are losing money?
5. Which stores need attention, based on a score anyone can check?
6. How much money is being lost to discounts and returns?
7. Where are deliveries running late? (This is an operational flag only — there's no cost data
   to say it's hurting profit, so I didn't claim that.)

## Recommendations

The recommendations sheet doesn't have anything typed in by hand. Each row is a formula that
checks that item's numbers and writes the action:

- **Region below target** → flags for a pricing/staffing review, states how far below target
- **Category with high returns or high discounts** → flags for investigation, with the actual
  percentage
- **Store flagged for attention** → names the store, its score, and its rank out of 120
- **Promotion to discontinue or continue** → states the exact margin impact and the lift in
  average order value

If the data changes and you refresh, the recommendations update on their own.

---

## Business Impact

**Reporting:** one set of numbers everyone can trust, instead of every department calculating
its own version. Refresh the file and it's up to date.

**Decision-making:** instead of a vague "revenue looks fine," management gets a specific list —
3 regions, 8 categories, 48 promotions, 30 stores — worth looking at first.

**Financial visibility:** the workbook shows ₹206.4 Cr in discount leakage and ₹66.5 Cr in
return/refund leakage. Both are historical leakage — money already given up through discounts
or lost to refunds over FY21–24, based on real transaction data. These are not projected or
recoverable savings. I haven't modeled how much of this could actually be recovered — that
would need assumptions this data can't support.

---

## What this project does NOT do

Being upfront about the limits:

- No inventory or warehouse data exists in the ERP export, so I can't say anything about
  stock levels or holding costs.
- No shipping cost data exists — only delivery dates and status — so I can measure late
  deliveries but not what they cost.
- Suppliers only connect to products, not to shipments, so I can't measure supplier delays.
- No store-level sales target exists, so the store score doesn't use one.
- No employee cost data exists, so I couldn't tie staff performance to profit.
- This only covers gross margin, not full P&L — there's no rent, overhead, or fixed cost data.
- There's no margin decline to fix — the real issue is volatility, and that's what this
  project is built around.
- I haven't calculated any ₹ savings from the recommendations — only how much is currently
  being lost to discounts and returns.
- This isn't real-time. You refresh it when you want updated numbers.
- No AI or machine learning is used anywhere. It's Excel formulas, Power Query, and DAX.
- The Gross Margin % card on the Power BI dashboard currently shows 7.21%, which does not match
  the Excel-verified 13.99%. This is a known, unresolved issue — see the Power BI section above.

---

## Tools used

- **Excel:** Power Query, Excel Tables, SUMIFS, INDEX-MATCH, PivotTables, conditional
  formatting, named ranges
- **Power BI:** data modeling, DAX, dashboard
- **Python:** used for the data preparation steps behind the scenes

## Files

- `Nova_Retail_MIS_Reporting_System.xlsx` — the full Excel workbook
- `PowerQuery_DAX_Specification.md` — the M-code and DAX to rebuild the data model in Excel
- Power BI dashboard file (`.pbix`) — same data, cross-checked against Excel
