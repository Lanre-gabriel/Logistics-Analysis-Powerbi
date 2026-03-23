# Logistics-Analysis-Powerbi
---
This project analyzes logistics performance across different shipping modes (Jets, Bus, Lorry, and Motorbike), focusing on shipment completion rate, in-progress shipments, shipment value, and delivery time to uncover insights and improve operational efficiency.

## Table of Content 
- [Introduction](#Introduction)
- [Project Objectives](#Project-Objectives)
- [Data Transformation](#Data-Transformation)
- [Data Normalization and Modelling](#Data-Normalization-and-Modelling)
- [Analysis and Calculations using Dax](#Analysis-and-Calculation-Using-Dax)
- [Data Visualization](#Data-Visualization)
- [Insight](#Insight)
- [Conclusion](#Conclusion)
- [Recommendations](#Recommendations)
- [About Me](#About-Me)
---

## Introduction 

Logistics is the backbone of many businesses, yet delays, inefficiencies, and poor customer satisfaction can silently impact overall performance.

In this project, I analyzed a logistics dataset to evaluate shipment performance, delivery efficiency, and customer satisfaction across different shipping modes and cities.
The goal of this analysis is to provide insights into logistics operations, identify inefficiencies, and deliver actionable recommendations to optimize shipment completion rates and reduce in-progress shipments.
Additionally, this project examines shipment value, delivery timelines, and customer satisfaction levels to help stakeholders understand performance trends and assess the impact of different delivery modes on timely deliveries.

---

## Project Objectives 

The objectives of this project are to;

- Measure completed shipments across various transport modes (Jets, Bus, Lorry, and Motorbike) and track their trends over time (monthly and yearly)
  
- Analyze shipment value, identifying patterns and fluctuations over time and across different transport modes.
  
- Break down shipment performance by region, with a focus on tracking expected shipments in each city by transport mode.

-Assess customer satisfaction based on shipment performance, delivery timelines, and feedback across different shipment types.

- Evaluate in-progress shipments to identify bottlenecks and areas for operational improvement.

---

## Data Cleaning & Transformation (Power Query)

The dataset was cleaned and transformed using **Power Query** to ensure data consistency, accuracy, and readiness for analysis. Several transformation steps were applied to structure the data and create meaningful categories for better insights.

---
**1. Splitting Delivery Location**

- The **Delivery Location** column was splitted using a delimiter.
- This allowed extraction of structured fields such as **City** and **Region**.
- It improves location-based analysis and dashboard filtering.

**2. Customer Satisfaction
Categorization**

To make customer feedback more interpretable, the **Customer Satisfaction Score (CSS)** was grouped into a Likert-scale format.

A custom column named **Customer Satisfaction Category** was created using conditional logic.

**Categorization:**

- 1 – 2 → Very Dissatisfied  
- 3 – 4 → Dissatisfied  
- 5 – 6 → Neutral  
- 7 – 8 → Satisfied  
- 9 – 10 → Very Satisfied  

**Power Query Formula:**

```powerquery
if [CSS] >= 1 and [CSS] <= 2 then "Very Dissatisfied"
else if [CSS] >= 3 and [CSS] <= 4 then "Dissatisfied"
else if [CSS] >= 5 and [CSS] <= 6 then "Neutral"
else if [CSS] >= 7 and [CSS] <= 8 then "Satisfied"
else "Very Satisfied"
```
**3. Carrier Performance Rating Categorization**

To evaluate logistics efficiency, the **Carrier Performance Rating** was transformed into grouped categories.

A new column called **Carrier Performance Category** was also created using conditional logic.

**Categorization:**

- ≤ 2 → Low Rating  
- = 3 → Medium Rating  
- > 3 → High Rating  

**Power Query Formula:**

```powerquery
if [Carrier Performance Rating] <= 2 then "Low Rating"
else if [Carrier Performance Rating] = 3 then "Medium Rating"
else "High Rating"
```

---

## Data Normalization & Modelling

To improve data structure, reduce redundancy, and enable efficient analysis, the dataset was normalized into multiple dimension tables and a central fact table using a **star ⭐ schema approach**.

**Data Normalization**

The raw dataset was decomposed into 8 smaller, well-structured tables to eliminate duplication and ensure consistency across the model and 1 Centralized facts table 

Each table was designed to represent a specific business entity.

**Dimension Tables**

The following dimension tables were created:

**📦 DimShipments**
Contains core shipment details:
- Shipment ID  
- Shipment Status  
- Shipment Type  
- Shipment Priority  

 **🚚 DimShippingMode**
Stores shipping method details:
- Shipping Mode (Bus, Jets, Lorry, Motorbike)

**👥 DimCustomer**
Contains customer segmentation:
- Customer Type (Business, Retail, International)  
- Customer ID (generated using index column)


**📍 DimLocation**
Stores geographical data:
- City  
- City ID  

**💳 DimPayment**
Captures payment information:
- Payment Status (Paid, Pending, Overdue)  
- Payment Status ID  

**🚛 DimCarrier**
Stores carrier details:
- Carrier Name  
- Carrier ID  

**⭐ DimCarrierSatisfaction**
Captures customer satisfaction categories:
- Customer Satisfaction Category  
- Satisfaction ID  

**📊 DimCarrierPerformance**
Captures carrier performance categories:
- Carrier Performance Category  
- Performance ID  

**Data Modelling**
After creating the dimension tables, relationships were established between them and the central fact table.

- A **star schema** was implemented  
- The **fact table** contains transactional shipment data  
- Dimension tables are linked using unique IDs (keys)  
- Relationships were created as **one-to-many (1:M)**  

This data model:

- Reduced data redundancy  
- Improved query performance  
- Enabled efficient filtering and slicing in Power BI  
- Provided a scalable structure for future data expansion

---

## Analysis & Calculations (DAX)

To support the analysis, key DAX measures were created to track shipment performance, delivery efficiency, and financial metrics.

**Base Measures**

These foundational measures were created and reused across multiple calculations:

**📦 Total Shipments**
```DAX
Total Shipments = COUNTROWS('Fact Logistics')
```
**Shipment Status KPIs**
***Completed Shipment***
```DAX
Completed Shipments = 
CALCULATE(
    COUNTROWS('Fact Logistics'),
    'DimShipment'[Shipment Status] = "Completed"
)
```
**🚚 In-Progress Shipments**
***(All non-completed shipments grouped together)***
```DAX
In Progress Shipments = 
CALCULATE(
    COUNTROWS('Fact Logistics'),
    'DimShipment'[Shipment Status] IN {
        "In Transit",
        "Not Completed",
        "Ready for Pickup"
    }
)
```
**Shipment Value**
```DAX
Shipment Value = SUM('Fact Logistics'[Shipment Value ($)])
```
**Performance Ratios**
```DAX
% Completed = 
DIVIDE([Completed Shipments], [Total Shipments], 0)
```
***%Early Deliveries***
```DAX
% Early Shipments =
VAR Early =
    CALCULATE(
        COUNTROWS('Fact Logistics'),
        'DimShipment'[Shipment Status] = "Completed",
        'Fact Logistics'[Delivery Date] <= 'Fact Logistics'[Estimated Delivery Date]
    )
RETURN
DIVIDE(Early, [Completed Shipments], 0)
```
***%Late Deliveries***
```DAX
% Late Shipments =
VAR Late =
    CALCULATE(
        COUNTROWS('Fact Logistics'),
        'DimShipment'[Shipment Status] = "Completed",
        'Fact Logistics'[Delivery Date] > 'Fact Logistics'[Estimated Delivery Date]
    )
RETURN
DIVIDE(Late, [Completed Shipments], 0)
```
**Time-Based Analysis**
***Year-Over-Year(YoY)- Completed Shipments***
```DAX
YoY Completed Shipments =
VAR SPLY =
    CALCULATE(
        [Completed Shipments],
        SAMEPERIODLASTYEAR('DateTable'[Date])
    )
VAR YoY =
    DIVIDE([Completed Shipments], SPLY) - 1
RETURN
FORMAT(YoY, "0%")
```
***Month-over-Month - Shipment Value***
```DAX
MoM Shipment Value =
VAR PM =
    CALCULATE(
        [Shipment Value],
        PREVIOUSMONTH('DateTable'[Date])
    )
VAR MoM =
    DIVIDE([Shipment Value], PM) - 1
RETURN
FORMAT(MoM, "0%")
```
**Payments performance**
***% Paid Shipments***
```Dax
% Paid =
VAR Paid =
    CALCULATE(
        [Shipment Value],
        'DimPayment'[Payment Status] = "Paid"
    )
RETURN
DIVIDE(Paid, [Shipment Value], 0)
```
---

## Data Visualization (Dashboard Preview)

---

## Insight 

---

## Conclusion 

---
## Recommendation

---

## About Me
