# JCars Logistics Business Analysis & Power BI Dashboard

## Executive Summary
This repository contains an end-to-end Business Intelligence (BI) and data analytics solution developed for **JCars Logistics**, a motor vehicle dealership and supply chain enterprise operating across Kenya. The initiative transforms an unrefined, raw transactional dataset into an executive-ready, interactive Power BI performance and diagnostic platform. 

The primary business objective is to evaluate end-to-end operational efficiency, profitability, delivery logistics, and customer satisfaction—enabling evidence-based strategic decision-making and business interventions.

---

## Key Performance Indicators (KPIs)
* **Total Vehicles Sold:** 466 units
* **Total Transactions:** 276 sales orders
* **Total Calculated Revenue:** KES 1,814,035,803.45 (~KES 1.81 Billion)
* **Total Gross Profit:** KES 341,018,553.66 (~KES 341.02 Million)
* **Gross Profit Margin:** 18.80%
* **Fulfillment Pipeline:** 146 Delivered | 88 Returned | 27 Cancelled
* **Net Logistics Deficit:** -KES 2,807,354.54 (Delivery subsidization across regions)

---

## Repository Structure
```text
├── data/
│   ├── Jcars_Original_Data.csv        # Unrefined flat file dataset (276 rows × 36 columns)
│   └── Jcars_Cleaned_Data.xlsx         # Processed and transformed dataset output
│   └── JCars_Logistics_Analysis.pbix  # Interactive Power BI Report file
├── Image/
│   └── images                       # Analysis screenshots, dashboard screenshots, and star schema diagrams
└── README.md                          # Project documentation
```

---

## Dataset Architecture & Grain
* **Grain:** One row represents an individual vehicle sales order transaction.
* **Dimensions:** 276 rows × 36 attributes.
* **Primary Domains Covered:**
  * **Customer Profiles:** Name, Customer Type (Individual, Corporate, NGO, Government, Dealer), Age, Region, County, City.
  * **Sales & Channels:** Branch, Sales Representative, Lead Source (Walk-in, Website, Facebook, Instagram, WhatsApp, Referral, Corporate Tender, Phone Call).
  * **Vehicle Attributes:** Make, Model, Body Type (SUV, Sedan, Pickup, Truck, Hatchback, Wagon, Crossover, Van), Model Year, Fuel Type, Transmission, Color.
  * **Financials & Pricing:** Units Sold, Unit Selling Price, Unit Cost, Discount, Delivery Fee, Logistics Cost, Recorded Revenue, Calculated Revenue.
  * **Fulfillment & Data Quality:** Order Date, Delivery Date, Payment Method, Payment Status, Delivery Status, Return Status, Customer Rating, Review Count, Data Quality Flags.

---

## Data Quality Diagnostics & Data Hygiene Rules

### Data Anomalies Discovered
1. **Identifier Anomalies:** 19 records contained placeholder IDs (`UNKNOWN`); 3 duplicate Order IDs spanned entirely different customers (`LCL1020`, `LCL1086`, `LCL1174`).
2. **Date Inconsistencies:** 16 records had delivery dates preceding order dates; 26 records missed order dates; 29 records missed delivery dates.
3. **Pricing & Revenue Discrepancies:** 14 records listed $0.00 selling prices; 17 records listed $0.00 costs; KES 431.58M discrepancy between recorded revenue and expected calculated revenue.
4. **Missing Values:** 20 records contained unstandardized return statuses.

### ETL & Data Transformation Rules (Power Query M)
* **Initial Parsing State:** Converted all columns to standard text (`type text`) prior to custom parsing.
* **Text & Missing Values:** Standardized `Blanks`, `N/A`, `-`, and `Null` to `"Unknown"` in text attributes, and `null` in numeric attributes.
* **Customer Segmentation:** Standardized company entity values into `Corporate`, `Retail` into `Individual`, `County Government` into `Government`, and `Non-profit` into `NGO`.
* **Age & Year Filtering:** Replaced ages outside 18–100 years with `null`. Replaced invalid/future model years (`202A`, `2032`) with `null` and converted the attribute to Whole Number.
* **Geographic Consolidation:** Normalized branch names to include `"Yard"` (e.g., `"Mombasa Port Yard"`). Mapping single-entry locations to parent hubs (e.g., `Embakasi`/`Westlands` $\rightarrow$ `Nairobi`; `Nyali` $\rightarrow$ `Mombasa`; `Mlolongo` $\rightarrow$ `Athi River`).
* **Sales Representatives:** Merged single-name entries with matching full names to preserve attribution accuracy.
* **Fuel & Monetary Normalization:** Standardized `Gasoline` and `PMS` to `Petrol`. Converted negative `Units Sold` and `Review Count` values to standard positive integers. Capped illogical discounts ($\ge 50\%$) to `null`.
* **Currency Exchange Rates (CBK Rates):**
  * $1 \text{ USD} = 129.50 \text{ KES}$
  * $1 \text{ ZAR} = 7.89 \text{ KES}$
* **Default Imputation Rules:**
  * `Customer Age`: 35 | `Vehicle Year`: 2022 | `Units Sold`: 1 | `Discount`: 0 | `Customer Rating`: 3 | `Delivery Fee`/`Logistics Cost`: 0 | `Order ID`: `"UNKNOWN"`.

---

## Data Modeling (Star Schema)

The analytical model uses a dimensional Star Schema structure centered on `Fact_Sales`, connected via integer surrogate keys. Role-playing dimensions (`Dim_Date`) are implemented for time-series modeling across order and delivery dates.

```
                     ┌───────────────────────┐
                     │     Dim_Customer      │
                     ├───────────────────────┤
                     │ PK  CustomerKey       │
                     │     Customer Name     │
                     │     Customer Type     │
                     │     Customer Age      │
                     └───────────┬───────────┘
                                 │ 1
                                 │
                                 │ *
 ┌───────────────────────┐   ┌───┴───────────────────┐   ┌───────────────────────┐
 │     Dim_Location      │   │      Fact_Sales       │   │  Dim_Sales_Channel    │
 ├───────────────────────┤   ├───────────────────────┤   ├───────────────────────┤
 │ PK  LocationKey       │   │ PK  SalesKey          │   │ PK  SalesChannelKey   │
 │     Region            │   │ FK  CustomerKey       │   │     Sales Rep         │
 │     County            │ 1 │ FK  LocationKey       │ 1 │     Lead Source       │
 │     City              ├───┤ FK  SalesChannelKey   ├───┤                       │
 │     Branch            │ * │ FK  VehicleKey        │ * └───────────────────────┘
 └───────────────────────┘   │ FK  DateKey           │
                             │ FK  DeliveryDateKey   │
                             │     Units Sold        │
                             │     Unit Selling Price│
                             │     Unit Cost         │
                             │     Discount          │
                             │     Delivery Fee      │
                             │     Logistics Cost    │
                             │     Revenue           │
                             │     Customer Rating   │
                             │     Review Count      │
                             │     [Degenerates]     │
                             └───┬───────────────┬───┘
                               * │               │ *
                                 │               │
                                 │ 1             │ 1
                     ┌───────────┴───────────┐ ┌─┴─────────────────────────┐
                     │      Dim_Vehicle      │ │        Dim_Date         │
                     ├───────────────────────┤ ├─────────────────────────┤
                     │ PK  VehicleKey        │ │ PK  DateKey             │
                     │     Car Make          │ │     Date                │
                     │     Car Model         │ │     Year / Quarter      │
                     │     Vehicle Type      │ │     Month / Day_of_Week │
                     │     Vehicle Year      │ └─────────────────────────┘
                     │     Fuel Type         │
                     │     Transmission      │
                     │     Color             │
                     └───────────────────────┘
```

---

## Key DAX Measures

```dax
// Total Revenue
Total Revenue = 
SUM(Jcars_Cleaned_data[Revenue])

// Total Transactions
Total Transactions = 
COUNTA(Jcars_Cleaned_data[Order ID])

// Average Order Value
Average Order Value = 
DIVIDE([Total Revenue], [Total Transactions], 0)

// Total Gross Profit
Total Gross Profit = 
SUM(Jcars_Cleaned_data[Revenue]) - SUMX(
    Jcars_Cleaned_data,
    (Jcars_Cleaned_data[Units Sold] * Jcars_Cleaned_data[Unit Cost]) + 
    COALESCE(Jcars_Cleaned_data[Logistics Cost], 0)
)

// Gross Profit Margin (%)
Gross Profit Margin (%) = 
DIVIDE([Total Gross Profit], SUM(Jcars_Cleaned_data[Revenue]), 0)

// Logistics Cost Ratio (%)
Logistics Cost Ratio (%) = 
DIVIDE(SUM(Jcars_Cleaned_data[Logistics Cost]), [Total Revenue], 0)

// Total Cancelled Volume
Total Cancelled Volume = 
CALCULATE(
    COUNTROWS(Jcars_Cleaned_data),
    Jcars_Cleaned_data[Payment Status] = "Cancelled"
)

// Total Returned Volume
Total Returned Volume = 
CALCULATE(
    COUNTROWS(Jcars_Cleaned_data),
    Jcars_Cleaned_data[Returned] = "Yes"
)

// Data Quality Tracker
Data Quality = 
COUNT(Fact_Sales[Data Quality Flag])
```

---

## Executive Dashboard & Key Business Insights

### 1. Revenue & Brand Performance
* **Top Performing Brand:** **Toyota** dominates total volume and financial contribution, generating **KES 720.90M revenue** and **KES 146.26M gross profit** across 142 units sold.
* **Highest Profitability Margin:** **Volkswagen** achieves the highest profit margin in the portfolio at **75% Gross Profit Margin**.
* **Underperforming Brands:** **Nissan (-8% margin)** and **BMW (-1% margin)** operate at negative net profit margins, indicating over-discounting or high procurement acquisition costs.
* **Body Type Leaders:** **SUVs** represent the primary revenue driver (>KES 1.05B revenue), followed by **Sedans**.

### 2. Regional & Branch Efficiency
* **Top Profitability Benchmark (Nairobi):** Yields a **62% Gross Profit Margin**, delivering KES 128.70M gross profit on 28 high-value units sold.
* **Volume Driver (Central / Thika):** Thika leads volume with **KES 373.88M revenue** across 71 units at a **29% margin**.
* **Margin Compression Risks:** **Nyanza (Kisumu)** achieves only a **2% Gross Profit Margin** (KES 3.50M profit on KES 217.53M revenue) due to delivery subsidies and heavy discounting.

### 3. Sales Representative Performance
* **Top Performers:** **Grace Njeri** leads the team in total revenue (>KES 340M) and gross profit (~KES 145M). **Faith Achieng** leads in unit sales volume (~62 units).
* **Volume vs. Profit Discrepancies:** **Samuel Mutua** moved high unit volumes (~48 units) but delivered low profit margins due to high average discounts. **Kevin Mwangi** generated the lowest revenue (~KES 105M) with near-zero profit contribution.

### 4. Lead Sources & Payment Channels
* **High-Value Channels:** **Website** and **Walk-in** channels lead in overall profitability, generating over KES 300M in revenue and ~KES 130M in gross profit each.
* **Loss-Making Acquisition Channels:** **Phone Call** and **WhatsApp** channels operate at negative gross profit margins.
* **Payment Friction:** Fast, low-friction methods (**M-Pesa**, **Bank Transfer**) drive core cash inflows. **Partially Paid** orders hold the highest Average Order Value at **KES 9.84M**.

---

## Management Action Items & Strategic Recommendations

1. **Restructure Procurement Capital:** Freeze or reduce inventory acquisition for loss-making makes (Nissan, BMW, Mercedes-Benz). Reallocate working capital toward high-margin brands (Toyota, Volkswagen, Subaru).
2. **Standardize Delivery Pricing:** Address the delivery subsidization defect (present in ~69.6% of orders) by replacing discretionary regional freight fees with standardized, distance-based delivery tariffs.
3. **Institute Pre-Delivery Inspections (PDI):** Implement mandatory PDI sign-offs at branch yards prior to customer handover to mitigate the **31.9% return rate** (concentrated in SUVs and Hatchbacks).
4. **Enforce Credit Governance:** Halt physical vehicle releases on pending or partially paid orders, locking up KES 316M in uncollected funds—specifically across NGO and Corporate accounts.
5. **Implement Discount Authority Caps:** Restrict sales agent discretionary discount authority to a maximum of **5%**, requiring manager sign-offs for higher rates.