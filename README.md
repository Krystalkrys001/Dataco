# DataCo Smart Supply Chain: Sales, Shipping & Customer Analytics

A three-page Power BI dashboard built on the DataCo Smart Supply Chain dataset (180,519 real order line items), analyzing sales and profitability, shipping and delivery performance, and customer and order health.

## Table of Contents

- [Overview]
- [Business Questions]
- [Dataset]
- [Tools Used]
- [Data Cleaning and Preparation]
- [Dashboard Pages]
- [Key Insights]
- [Sample DAX Measures]
- [Data Quality Notes and Limitations]
- [Repository Structure]
- [Links]
- [Author]

## Overview

This project analyzes a real, publicly available supply chain dataset to answer three separate business questions: how is the business performing financially, is it actually delivering what it sells on time, and who is it really serving. Each question got its own dashboard page rather than one combined view, so every visual on a page had to justify its place by answering that page's specific question.

## Business Questions

1. **Sales and Profit** - Where does the business actually make its money, and does that match where it sells the most?
2. **Shipping and Delivery Performance** - Is the business delivering what it sells, on time, and can that be trusted at face value?
3. **Customer and Order Status** - Who is the business really serving, and how healthy is the order pipeline?

## Dataset

- **Source:** DataCo Smart Supply Chain Dataset (public)
- **Size:** 180,519 rows, 53 columns (as received)
- **Grain:** one row per order line item, not one row per order
- **Coverage:** 5 global markets (Europe, LATAM, Pacific Asia, USCA, Africa), over 20,000 unique customers, roughly 65,000 distinct orders
- **Key fields used:** Sales, Order Item Total, Order Profit Per Order, Order Item Discount, Days for shipping (real), Days for shipment (scheduled), Delivery Status, Late_delivery_risk, Order Status, Customer Id, Order Id, Category Name, Product Name, Market, Order Region/State/City

## Tools Used

- Power BI Desktop (Power Query, Data Model, DAX)

## Data Cleaning and Preparation

Every column was checked before being trusted, not assumed clean because it came from a "real" dataset. The following actions were taken in Power Query:

**Removed, zero analytical value:**
- Customer Password and Customer Email, both held a single repeated placeholder value across all 180,519 rows
- Product Description, entirely blank across every row
- Product Status, a single unique value across the entire dataset

**Removed, exact duplicates of a retained column:**
- Benefit per order (identical to Order Profit Per Order)
- Category Id (identical to Product Category Id)
- Product Card Id (identical to Order Item Cardprod Id)

**Removed, insufficient usable data:**
- Order Zipcode (86% of rows missing)
- Customer Street (over 7,400 unique values, too granular for meaningful analysis)
- Product Image (file path field, not used in this analysis)

**Retained despite appearing redundant at a glance:**
- Order City/Country/Region/State and Customer City/Country/State were kept separately, they represent different things (where a customer is based versus where an order shipped to), not duplicate data.

## Dashboard Pages

### Page 1: Sales and Profit Analysis
KPI cards: Total Sales, Total Profit, Average Order Value
Visuals: Sales by Market and Customer Segment, Order Status by Profit, Top 10 Products by Sales, Top 10 Categories by Profit

### Page 2: Shipping and Delivery Performance
KPI cards: Late Delivery Rate, On-Time Delivery Rate, Average Delivery Delay, Average Scheduled Shipment Days
Visuals: Real vs Scheduled Days by Shipping Mode, Delivery status breakdown

### Page 3: Customer and Order Status
KPI cards: Total Customers, Cancellation Rate, Fraud Rate
Visuals: Sales by Customer Segment, Top 10 and Bottom 10 Customers by Sales, Top 10 States by Sales

## Key Insights

- Total Sales reached 33.05M against Total Profit of 3.97M, a blended margin of roughly 12%.
- The Consumer segment leads sales in every single market with no exceptions, a structural pattern rather than a regional one.
- The single highest selling product (Field & Stream Sports, 0.76M) is not in the category generating the most profit. Fishing leads all categories in profit at 0.61M.
- Late Delivery Rate stands at 54.83%, calculated from the dataset's own Late_delivery_risk flag. On-Time Delivery Rate, calculated strictly from Delivery Status = "Shipping on time" only, stands at 17.84%. A looser, complement-based definition would inflate that figure to roughly 45%, and was deliberately not used here.
- Average delivery delay sits near zero (0.57 days) despite the majority of orders technically qualifying as late, indicating the issue is distributional rather than reflected in the average.
- One customer accounts for 11.7M in total sales, an order of magnitude above the next highest customer. This is flagged as requiring source-system verification rather than reported as a confirmed insight.
- Cancellation Rate stands at 2.08% and Suspected Fraud rate at 2.26% of all orders.

## Sample DAX Measures

```
Total Sales = SUM('Table[Order Item Total])

Total Profit = SUM('Table[Order Profit Per Order])

Profit Margin % =
DIVIDE(
    SUM('Table[Order Profit Per Order]),
    SUM('Table[Order Item Total])
)

Total Orders = DISTINCTCOUNT('Table'[Order Id])

Total Customers = DISTINCTCOUNT('Table[Customer Id])

Average Order Value =
DIVIDE(
    SUM(Table'[Order Item Total]),
    DISTINCTCOUNT('Table'[Order Id])
)

Late Delivery Rate =
DIVIDE(
    CALCULATE(COUNTROWS('Table), Table'[Late_delivery_risk] = 1),
    COUNTROWS('Table')
)

On-Time Delivery Rate (strict) =
DIVIDE(
    CALCULATE(COUNTROWS('Table), 'Table'[Delivery Status] = "Shipping on time"),
    COUNTROWS('Table')
)

Average Delivery Delay =
AVERAGE('Table'[Days for shipping (real)]) -
AVERAGE('Table'[Days for shipment (scheduled)])

Fraud Rate =
DIVIDE(
    CALCULATE(DISTINCTCOUNT('Table'[Order Id]), 'Table'[Order Status] = "SUSPECTED_FRAUD"),
    DISTINCTCOUNT('Table'[Order Id])
)

Cancellation Rate =
DIVIDE(
    CALCULATE(DISTINCTCOUNT('Table'[Order Id]), 'Table'[Order Status] = "CANCELED"),
    DISTINCTCOUNT('Table'[Order Id])
)
```

## Data Quality Notes and Limitations

Documenting these openly rather than hiding them is part of the analysis, not a separate afterthought.

- The dataset is one row per order line item, not per order. Every measure involving a count of orders uses `DISTINCTCOUNT` on Order Id specifically to avoid overcounting.
- On-Time Delivery Rate can be legitimately calculated two different ways with very different results (17.84% strict versus roughly 45% using a looser complement definition). This project uses the strict definition throughout, and that choice is documented rather than left implicit.
- The relationship between the near-zero average delivery delay and the 54.83% late delivery rate has not yet been fully explained by this analysis. A closer look at the distribution of delivery delay (not just the average) is the identified next step.
- The top customer by sales (11.7M) and the leading state by sales (Puerto Rico, 14.2M) both show a magnitude of concentration large enough to warrant verification against source records before being treated as fully confirmed findings.
- The Shipping Mode volume chart reflects order volume, not delivery speed, and has intentionally not been used to make a speed comparison claim between shipping modes until that measure is rebuilt using averages rather than totals.

## Repository Structure

```
├── data/
│   └── DataCoSupplyChainDataset.csv
├── powerbi/
│   └── DataCo_Supply_Chain_Analysis.pbix
├── docs/
│   └── screenshots/
├── README.md
```

## Links

- Medium article: https://medium.com/@ulasistanley1/from-180-000-orders-to-one-question-where-does-this-business-actually-bleed-money-f56c97b05ef3]
- LinkedIn post: https://www.linkedin.com/posts/chukwuma-obinna-ulasi-082198413_techyjaunt-techyjaunt-dataanalytics-ugcPost-7492612876445315072-ZcRk/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAGk2S2IBonLi7enVgV_4EWmKqN3SMKtPPDg

## Author

**Ulasi Chukwuma Obinna**
Data Analyst
Email: ulasistanley1@gmail.com
GitHub: https://github.com/Krystalkrys001
Medium: https://medium.com/@ulasistanley1
LinkedIn: https://www.linkedin.com/in/chukwuma-obinna-ulasi-082198413/
