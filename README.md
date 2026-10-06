# Giant-Mart-Sales-Leakage-Case-Study
End-to-end retail diagnostic case study analyzing GH₵4.92M in net revenue across 3 regional branches in Ghana. Features a Power BI Sales Leakage Diagnosis Dashboard to expose unoptimized discounting and protect net profit margins


---

## Project Setup & Framework

### 1.1 Business Case & Problem Statement
**Client Identity:** Giant-Mart Ghana (Simulated FMCG Regional Supermarket Chain)  
**Operational Scope:** 3 Flagship Regional Branches (*Accra, Kumasi, Tamale*)[cite: 1, 2]  

Giant-Mart Ghana’s executive board was celebrating record-breaking gross revenue figures. However, financial audits revealed an operational paradox: **the store generating the highest raw sales volume (Kumasi) was delivering the lowest net profit margins**[cite: 1, 2, 4]. 

Branch managers possessed full autonomy over discretionary point-of-sale markdowns (0% to 40%). Leadership suspected managers were trapped in a **"Volume Bias"**—slashing prices on high-value inventory to hit monthly revenue targets while unconsciously eroding store profitability and selling items below supplier wholesale cost[cite: 1, 4].

### 1.2 Core Business Questions
1. **Discount Leakage Quantification:** What is the total monetary value lost to markdowns over the 15-month operational window, and how much of it resulted in net financial losses[cite: 4]?
2. **Territory Benchmarking:** How do branch rankings change when evaluated by *Retained Net Profit Margin* rather than *Gross Revenue Volume*[cite: 1, 4]?
3. **Threshold Analysis:** Which product categories suffer from deep discounting, and at what exact markdown percentage does a transaction become loss-making?
4. **Governance Guardrails:** What system policy controls are required to eliminate margin bleed while preserving store manager competitiveness?

### 1.3 Key Financial Metrics & Business Logic
To evaluate store health, the analysis uses five core calculated financial metrics:

* **Gross Revenue:** {Units Sold} x {Unit Shelf Price}
* **Discount Leakage:** {Gross Revenue} x {Discount Rate}
* **Net Revenue:** {Gross Revenue} - {Discount Leakage}
* **Net Profit:** {Net Revenue} - ({Units Sold} x {Unit Wholesale Cost})
* **Retained Profit Margin (%):** ( {Net Profit} / {Net Revenue}) x 100$

### 1.4 Repository Structure
```text
giant-mart-sales-leakage-case-study/
│
├── README.md                          <-- Project documentation & case study write-up
├── data/
│   └── Giant_Mart_Cleaned_Dataset.csv  <-- Cleaned operational dataset
├── dashboards/
│   └── Giant_Mart_Sales_Leakage.pbix   <-- Interactive Power BI dashboard workbook
└── visuals/
    └── Giant_Mart_Dashboard.png    <-- High-resolution dashboard screenshot
```

## Data Preparation & DAX Modeling

### 2.1 Dataset Architecture
The case study utilizes a consolidated relational dataset (`data/Giant_Mart_Cleaned_Dataset.csv`) representing 15 months of multi-branch supermarket operations. The schema standardizes operational metrics across three primary dimensions:

* **Temporal Attributes:** `Date`, `Year` (2025–2026), `Month`
* **Spatial & Administrative Attributes:** `Store_Location` (Accra, Kumasi, Tamale), `Store_Manager`
* **Product Catalog Attributes:** `Product_Category` (Electronics, Groceries, Home Decor, Fresh Produce), `Product_Name`, `Unit_Cost` (COGS), `Unit_Price` (Shelf Price)
* **Transactional Attributes:** `Qty_Sold`, `Discount_Applied` (0.00 to 0.40)

### 2.2 Data Integrity & Transformation Audit
Before constructing Power BI report pages, data validation checks were applied during the ETL/cleaning process:
1. **Missing Values & Nulls:** Verified zero blank records across transactional IDs, store keys, or product prices[.
2. **Boundary Validation:** Verified that `Discount_Applied` values sit between $0.00$ ($0\%$) and $0.40$ ($40\%$), and `Qty_Sold` remains strictly positive.
3. **Price Logic Verification:** Verified `Unit_Price` $\ge$ `Unit_Cost` at base catalog level to ensure baseline product profitability.

### 2.3 Core DAX Measures (Power BI Metric Engine)
To dynamically calculate financial metrics across slicers (Year, Month, Location, Product Category), the following DAX measures were implemented in Power BI:

```dax
// 1. Gross Revenue (Total potential income at full retail shelf price)
Gross Revenue = 
SUMX(
    GiantMart_Cleaned_Dataset,
    GiantMart_Cleaned_Dataset[Qty_Sold] * GiantMart_Cleaned_Dataset[Unit_Price]
)

// 2. Discount Leakage (Total value sacrificed to markdowns)
Discount Leakage = 
SUMX(
    GiantMart_Cleaned_Dataset,
    (GiantMart_Cleaned_Dataset[Qty_Sold] * GiantMart_Cleaned_Dataset[Unit_Price]) * GiantMart_Cleaned_Dataset[Discount_Applied]
)

// 3. Net Revenue (Actual cash collected at the checkout register)
Net Revenue = [Gross Revenue] - [Discount Leakage]

// 4. Cost of Goods Sold (Total wholesale supplier cost)
COGS = 
SUMX(
    GiantMart_Cleaned_Dataset,
    GiantMart_Cleaned_Dataset[Qty_Sold] * GiantMart_Cleaned_Dataset[Unit_Cost]
)

// 5. Net Profit (Actual cash retained by the business)
Net Profit = [Net Revenue] - [COGS]

// 6. Net Profit Margin % (Percentage of collected cash retained as profit)
Net Profit Margin % = 
DIVIDE([Net Profit], [Net Revenue], 0) * 100
```

## Analysis & Visual Architecture

### 3.1 Executive Dashboard Overview
The **Sales Leakage Diagnosis Dashboard** was constructed in Power BI to give executive stakeholders an immediate visual breakdown of top-line revenue versus profit retention.

<img width="987" height="552" alt="Image" src="https://github.com/user-attachments/assets/4fef4752-1b88-4b5c-9722-67a25d3214bb" />

### 3.2 Visual Component Mapping
The dashboard layout is structured into three diagnostic zones:

1. **Executive KPI Header (Top Cards):**
   * **Discount Leakage:** **GH₵454.19K** lost to price markdowns across all transactions.
   * **Units Sold:** **7K** total units moved across 15 months.
   * **Net Revenue:** **GH₵4.92M** in actual cash collected at checkout registers.
   * **Net Profit Margin:** **18.72%** overall business profit retention rate.
   * **Net Profit:** **GH₵920.02K** retained after wholesale COGS and discount deductions.

2. **Temporal & Regional Breakdown (Middle Visuals):**
   * **Net Revenue & Discount Leakage by Month (Combo Bar Chart):** Tracks monthly sales trends against discount volume, showing peak leakage during promotional periods (e.g., January/February).
   * **Net Revenue & Net Profit by Store Location (Stacked Bar Chart):** Visually compares revenue height against profit retention for Kumasi, Accra, and Tamale.

3. **Product & Territory Concentration (Bottom Visuals):**
   * **Discount Leakage by Product & Net Revenue by Product (Donut Charts):** Isolates product line contributions, proving that high-ticket items like **Smart TVs** and **Area Rugs** drive the bulk of both revenue and discount leakage.
   * **Discount Leakage by Store Location (Pie Chart):** Ranks regional contribution to total profit loss:
     * **Kumasi:** **GH₵194.42K (42.81%)** of total leakage.
     * **Accra:** **GH₵139.07K (30.62%)** of total leakage.
     * **Tamale:** **GH₵120.70K (26.58%)** of total leakage.

### 3.3 Diagnostic Findings

#### 1. The "Volume Bias" Illusion (Territory Misalignment)
* **Kumasi** generated the highest raw sales volume and top-line Net Revenue (~GH₵1.93M). However, because it ran aggressive markdowns, it suffered the worst leakage (**GH₵194.42K** / **42.81%** of national loss) and dropped to the lowest retained net profit margin in the chain (**17.72%**).
* **Tamale** generated less raw volume (~GH₵1.37M) but maintained a disciplined discounting approach, yielding the highest retained profit margin in the business (**19.63%**).

#### 2. The Electronics Discount Trap
* Deep discounting on high-cost inventory (specifically **Smart TV 43"**) creates severe negative-margin transactions.
* While discounts between 0% and 15% maintain healthy margins, applying **25% to 40% markdowns** pushes unit prices below wholesale supplier costs (COGS), causing Giant-Mart to actively lose up to **GH₵580 per unit sold**.

---

## Recommendations & Strategic Impact (ACT Phase)

### 4.1 Operational Policy Guardrails
To prevent further profit margin erosion while maintaining sales velocity, Giant-Mart should implement three core operational policy controls:

1. **Automated POS Discount Caps:**
   * Configure point-of-sale checkout software to hard-lock maximum discount allowances by product category.
   * Cap **Electronics** markdowns at **15%** maximum. Any discount request above 15% must require CFO or Commercial Director authorization in the system.

2. **Profit-First Manager Incentive Structure:**
   * Restructure store manager performance evaluations and quarterly bonus pools.
   * Shift primary evaluation criteria from **Gross Sales Volume** to **Retained Net Profit Margin (%)**, aligning branch leadership incentives with bottom-line business health.

3. **Margin-Floor Checkout Safeguards:**
   * Program POS registers to calculate real-time unit margins prior to printing receipts.
   * Automatically block any transaction where $\text{Net Price} < \text{COGS}$ to completely eliminate negative-margin sales on high-ticket inventory like Smart TVs.

### 4.2 Projected Commercial Impact
* **Direct Loss Recovery:** Immediate mitigation of negative-margin sales, preserving estimated tens of thousands of cedis annually in lost inventory value.
* **Margin Alignment:** Elevation of Kumasi’s retained net profit margin from **17.72%** toward the corporate target of **20%+**.
* **Overall Profitability Expansion:** Projected expansion of global retained net profit margin from **18.72%** to above **20.5%** across all regional operations within 6 months.

---

## 🛠 Tech Stack & Tools Used
* **Power BI:** Data modeling, DAX measure creation, dynamic KPI card construction, and interactive dashboard layout.
* **Excel / Data Cleaning:** Dataset normalization, baseline cost validation, and schema structuring.
* **GitHub:** Documentation, version control, and case study publishing.
