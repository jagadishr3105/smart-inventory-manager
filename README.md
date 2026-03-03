# Smart Inventory Excel Workbook README

**Document purpose**
This README provides the authoritative reference for the Smart Inventory Excel workbook. It explains the workbook’s purpose, business context, data structure, and how to operate, refresh, and extend the model. The document is written for analysts, planners, operations managers, and IT support working in an enterprise environment.

## 1) Workbook purpose and business outcomes
The workbook consolidates sales, inventory, and procurement data to provide a single operational view across regions, warehouses, SKUs, and suppliers. It enables:
* Daily and monthly demand tracking versus forecast to spot exceptions early.
* Revenue, COGS, and margin analysis at multiple levels (time, region, warehouse, SKU).
* Inventory health monitoring with turnover, stock value, and items below reorder point.
* Procurement readiness including supplier lead-time monitoring and suggested order quantities.
* Promotion effectiveness benchmarking against baseline performance.

The intended outcomes are faster decisions, reduced stockouts, optimized working capital, and aligned execution across Sales, Operations, and Procurement.

## 2) Audience and roles
* **Demand & Supply Planners:** Review demand vs. forecast, drive replenishment proposals.
* **Warehouse & Operations Managers:** Monitor inventory levels, turnover, urgent SKUs, and throughput.
* **Procurement:** Track supplier performance and lead times; act on the replenishment queue.
* **Finance/FP&A:** Validate revenue, COGS, and margin at month‑end and for rolling forecasts.
* **IT/Data Operations:** Maintain data pipelines, access controls, and workbook versioning.

## 3) Data structure and sources

### 3.1 Input layer
`Raw_Data` — transactional grain, typically one row per `Date` × `SKU_ID` × `Warehouse_ID` × `Supplier_ID` with the following key fields:

| Field | Description |
| :--- | :--- |
| **Date** | Transaction date for sales and inventory snapshot. |
| **SKU_ID** | Stock keeping unit identifier. |
| **Warehouse_ID** | Logical or physical warehouse code. |
| **Supplier_ID** | Vendor providing the SKU. |
| **Region** | Commercial/operational region (e.g., North, South, East, West). |
| **Units_Sold** | Units shipped/sold for the date. |
| **Inventory_Level** | On-hand units (end-of-day or as-of snapshot). |
| **Supplier_Lead_Time_Days** | Historical or master-data lead time in days. |
| **Reorder_Point** | Policy threshold triggering replenishment. |
| **Order_Quantity** | Usual replenishment lot size or last purchase qty (used for suggestions). |
| **Unit_Cost** | Item cost used for COGS. |
| **Unit_Price** | Selling price used for revenue. |
| **Promotion_Flag** | 1 if promotion applies; 0 otherwise. |
| **Stockout_Flag** | 1 when stockout occurred; 0 otherwise. |
| **Demand_Forecast** | Optional forecast for the date/SKU/warehouse. |

> *Add to new sheet* (Source artifact)
> **Grain & integration note:** The model assumes additive daily facts at the SKU‑Warehouse level. If your system supplies weekly or monthly facts, normalize dates before loading.

### 3.2 Curated layer
The workbook includes helper sheets that are the *only* sources you should use for PivotTables and charts:
* **Monthly_Summary:** Month-level totals: Units Sold, Demand Forecast, Revenue, COGS, Gross Profit, Avg Inventory, and Gross Margin %.
* **Region_Split:** Region-level Units Sold and Revenue.
* **Top_SKUs:** SKU-level Units Sold and Revenue (sorted by volume).
* **Inventory_Turnover:** SKU-level COGS, average inventory, and calculated `Inv_Turnover = COGS / AvgInventory`.
* **Region_by_Month:** Month × Region Units Sold for stacked/area visuals.
* **Promo_Impact:** Aggregations split by Promotion (Promo vs. No Promo) including Total Units, Revenue, and Avg Lead Time.
* **Warehouse_KPIs:** Warehouse-level Units, Revenue, COGS, Avg Lead Time, Avg Inventory, Gross Profit/Margin, and Stock Value.
* **Supplier_Metrics:** Supplier-level Units, Revenue, and Avg Lead Time.
* **SKU_WH_Stock:** Latest Current_Stock, Reorder_Point, Avg_Lead_Time, 30‑day demand, shortfall, and Suggested_Order_Qty per SKU × Warehouse.
* **Reorder_List:** Filtered view of SKU_WH_Stock where Current_Stock < Reorder_Point; used for the replenishment table.
* **Date_Table:** Continuous calendar (Date, Year, Quarter, Month, YearMonth) to support robust time slicing.

## 4) Business definitions and methodology

**Revenue / COGS / Gross Profit**
* `Revenue = Units_Sold × Unit_Price`
* `COGS = Units_Sold × Unit_Cost`
* `Gross_Profit = Revenue − COGS`
* `Gross_Margin_% = Gross_Profit / Revenue` *(blank if Revenue=0)*

**Inventory Turnover**
* `Inv_Turnover = COGS / AvgInventory` at the SKU level, then aggregated for displays.
* Higher values indicate faster movement; extremely high values with frequent stockouts may signal understocking.

**Promotion Impact**
* KPIs compare Promo vs. No Promo populations for Units, Revenue, and average lead time to assess effect on demand and supply strain.

**Replenishment logic (Suggested Order Qty)**
* `Shortfall = max(0, Reorder_Point − Current_Stock)`
* `Demand_30D` approximates near‑term pull.
* `Suggested_Order_Qty = max(round(Shortfall), round(Demand_30D), median(Order_Quantity))`
* *Note:* Business users may replace the rule with policy‑based EOQ, safety stock, or service‑level targets.

**Time handling**
* Dates are normalized to a proper Excel date and rolled up to Month and Quarter for analysis.
* The `Date_Table` provides a continuous calendar so Timelines and year‑over‑year views do not break when some days have no transactions.

## 5) How to use the dashboard

### 5.1 Opening and navigation
* The recommended tabs are **Overview**, **Warehouse Ops**, and **Procurement**, populated with PivotCharts sourced from the curated sheets.
* Use the Slicers for Region, Warehouse, Supplier, Promotion, and the Timeline for Month to filter the entire page.

### 5.2 Typical workflows
**Daily health check (Overview)**
* Review Monthly Demand vs. Forecast for variance trends.
* Scan the Regional Demand Split and Revenue vs. COGS to confirm mix and profitability.
* Use the Inventory Turnover Distribution to spot slow‑moving risk.

**Warehouse management (Warehouse Ops)**
* Read the KPI cards for each warehouse (Units, Revenue, COGS, Profit, Margin).
* Check Stock Value and Units in Stock charts.
* Work the SKUs Below Reorder Point list in priority order (Shortfall or 30‑day demand).

**Procurement planning (Procurement)**
* Monitor Avg Lead Time, Items to Reorder, and Active Suppliers.
* Use Supplier Volume & Lead Time to balance allocations.
* Export the Replenishment Orders Required table for PO creation.

### 5.3 Interactivity
* Connect all relevant PivotTables to the same Slicers/Timeline via Report Connections, so one click filters all visuals.
* To drill, right‑click a Pivot cell and choose Show Details to see underlying rows.

## 6) Data refresh and maintenance

### 6.1 Refresh cadence
* Load the newest `Raw_Data` rows (append only) then use **Data ▸ Refresh All**.
* The curated sheets recompute automatically; PivotTables/Charts then update.

### 6.2 Data quality requirements
* **Mandatory fields:** Date, SKU_ID, Warehouse_ID, Units_Sold, Inventory_Level, Unit_Cost, Unit_Price.
* **Consistent keys:** All IDs must be stable across periods.
* **Numeric types:** Costs/prices/quantities must be numeric; blanks are allowed but degrade certain KPIs.
* **Lead times:** If missing, procurement charts may understate risk; populate from master data.

### 6.3 Versioning and change control
* Use a semantic version in the file name (e.g., `SupplyChain_CC_v1.4.xlsx`).
* Maintain a Change Log tab with date, owner, and summary of modifications.
* Store the workbook in a controlled SharePoint/OneDrive location with appropriate DLP, retention, and access policies.

## 7) Extending the model
* **Additional KPIs:** * `Service Level = 1 − Stockout_Rate`; 
    * Forecast Accuracy (MAPE, WAPE) if forecast history is available; 
    * `Working Capital = Stock Value × Unit Cost basis`.
* **Advanced replenishment:** Replace “median order quantity” with EOQ or service‑level driven reorder point and safety stock per SKU‑Warehouse.
* **Scenario analysis:** Create duplicates of the curated sheets with alternative assumptions (e.g., shorter lead times, different promo cadence) and switch sources via named ranges.

## 8) Security, privacy, and controls
* The workbook may contain cost and margin data considered sensitive.
* Apply least‑privilege access; distribute consumer‑only copies with Value‑Only exports when needed.
* If the workbook is integrated with Power Query or external connections, validate credentials and avoid embedding secrets.

## 9) Troubleshooting

| Symptom | Likely cause | Resolution |
| :--- | :--- | :--- |
| **Charts show blanks or errors** | Non‑date values or empty Date cells | Normalize date column; confirm it formats as a date and refresh. |
| **Turnover is zero or extremely high** | Missing or tiny AvgInventory | Check Inventory snapshots; ensure stable, periodic inventory inputs. |
| **Reorder list appears empty** | Reorder_Point not populated or equals zero | Populate policy values; refresh curated sheets. |
| **Timeline doesn’t filter some visuals** | Pivot not connected to Timeline/Slicers | Use Report Connections to link all Pivots to the same controls. |
| **GETPIVOTDATA returns #REF!** | Pivot moved or field renamed | Repoint the formula using Insert Function from the Pivot cell. |

> *Add to new sheet* (Source artifact)

## 10) Governance and responsibilities
* **Data owner:** Supply Chain Operations (source quality and definitions).
* **Model owner:** Analytics/BI team (curated sheets, calculations, visuals).
* **IT support:** Data platform, access, and backup.
* **Users:** Regional ops, warehouse teams, procurement, and finance stakeholders.

## 11) Appendix — sheet catalog
* **Raw_Data:** Transactional inputs *(do not pivot directly on this sheet in production dashboards)*.
* **Monthly_Summary:** Time‑series basis for demand vs. forecast and financial trends.
* **Region_Split:** Region share for donut charts.
* **Top_SKUs:** Supports Top‑N ranking visuals and SKU leaderboards.
* **Inventory_Turnover:** Source for turnover histogram and SKU risk sorting.
* **Region_by_Month:** Stacked/area charts for regional mix over time.
* **Promo_Impact:** KPI cards for promo vs. non‑promo performance.
* **Warehouse_KPIs:** KPI cards, stock value, and unit stock charts by warehouse.
* **Supplier_Metrics:** Procurement tab supplier visuals.
* **SKU_WH_Stock:** Basis for stock health, shortfall, and suggestions by SKU‑Warehouse.
* **Reorder_List:** Actionable procurement queue (export to CSV if needed).
* **Date_Table:** Slicer/Timeline calendar.

## 12) Contact and support
* **For data issues:** Supply Chain Ops Data Steward
* **For model/visual changes:** Analytics Center of Excellence
* **For access and platform:** IT Service Desk *(ticket category: Analytics/Excel/Operational Dashboards)*
