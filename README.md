# AtliQ Mart’s Diwali & Sankranti Promotions!

This repository contains the SQL scripts used to analyze the performance of promotional campaigns run by AtliQ Mart during Diwali 2023 and Sankranti 2024. The project addresses various business requests related to identifying high-value discounted products, store distribution, campaign effectiveness, and product performance in terms of incremental sales and revenue.

## Introduction

Promotional campaigns play a crucial role in the retail industry, driving sales and attracting customers during festive seasons. This project aims to analyze the performance of promotional campaigns conducted by AtliQ Mart during Diwali 2023 and Sankranti 2024. By leveraging data analytics, we seek to gain insights into the effectiveness of these campaigns and provide recommendations for optimizing future marketing strategies.

## Data Sources

The analysis is based on data obtained from AtliQ Mart's internal databases. The main datasets used include fact_events, dim_products, dim_stores, and sales_summary. These datasets contain information about product sales, store locations, promotional events, and campaign revenues.

## Project Overview:

1. Analyzed data from AtliQ Mart's internal databases.
2. Performed SQL queries to fulfill five business requests.
3. Insights are intended to inform future promotional strategies and resource allocation.

## Business Requests

### 1. High-Value Products in 'BOGOF' Promotion

**Objective:** Identify high-value products featured in the 'BOGOF' (Buy One Get One Free) promotion.

```sql
SELECT
    DISTINCT p.product_name,
    f.base_price
FROM
    fact_events f
JOIN
    dim_products p ON f.product_code = p.product_code
WHERE
    f.promo_type = 'BOGOF' AND f.base_price > 500;
```

### 2. Store Presence Overview

**Objective:** Provide an overview of the number of stores in each city.

```sql
SELECT
    City,
    COUNT(store_id) as Total_Stores
FROM
    dim_stores
GROUP BY
    City
ORDER BY
    Total_Stores DESC;
```

### 3. Promotional Campaign Revenue Analysis
**Objective:** Display total revenue generated before and after each promotional campaign.

```sql
SELECT 
     campaign_name,
     ROUND(SUM(base_price * `quantity_sold(before_promo)`) / 1000000,2) AS total_revenue_before_promotion,
     ROUND(SUM(CASE
            WHEN promo_type = 'BOGOF' THEN base_price * 0.5 * (`quantity_sold(after_promo)` * 2)
            WHEN promo_type = '500 Cashback' THEN (base_price - 500) * `quantity_sold(after_promo)`
            WHEN promo_type = '50% OFF' THEN base_price * 0.5 * `quantity_sold(after_promo)`
            WHEN promo_type = '33% OFF' THEN base_price * 0.67 * `quantity_sold(after_promo)`
            WHEN promo_type = '25% OFF' THEN base_price * 0.75 * `quantity_sold(after_promo)` END) / 1000000,2) AS total_revenue_after_promotion
     FROM
     fact_events
     JOIN
     dim_campaigns USING (campaign_id)
     GROUP BY campaign_name;
```

### 4. Incremental Sold Quantity Analysis during Diwali Campaign
**Objective:** Calculate Incremental Sold Quantity (ISU%) for each category during the Diwali campaign.

```sql
With  Diwali_campaign_sale as ( Select category , 
     Round(Sum((
          Case 
          When promo_type = "BOGOF" Then `quantity_sold(after_promo)`*2
          Else `quantity_sold(after_promo)`
          End
      - `quantity_sold(before_promo)`) * 100)
      / Sum(`quantity_sold(before_promo)`),2) as `ISU%` 
      From fact_events 
      Join 
      dim_products using(product_code)
      Join 
      dim_campaigns using(campaign_id)
      Where campaign_name = "Diwali"
      Group by category)

      Select 
      Category , 
      `ISU%`, 
      row_number() Over(order by `ISU%` desc) as rank_order 
      From Diwali_campaign_sale ;
```

### 5. Top 5 Products by Incremental Revenue Percentage
**Objective:** Identify the top 5 products ranked by Incremental Revenue Percentage (IR%) across all campaigns.

```sql
with cte1 as(
SELECT category,product_name,sum(base_price * `quantity_sold(before_promo)`) as Total_Revenue_BP,
sum(
case
when promo_type = "BOGOF" then base_price * 0.5 * 2*(`quantity_sold(after_promo)`)
when promo_type = "50% OFF" then base_price * 0.5 * `quantity_sold(after_promo)`
when promo_type = "25% OFF" then base_price * 0.75* `quantity_sold(after_promo)`
when promo_type = "33% OFF" then base_price * 0.67 * `quantity_sold(after_promo)`
when promo_type = "500 cashback" then (base_price-500)*  `quantity_sold(after_promo)`
end) as Total_Revenue_AP FROM retail_events_db.fact_events 
join dim_products using (product_code) 
join dim_campaigns using(campaign_id)
group by product_name,category),

cte2 as(
select *,(total_revenue_AP - total_revenue_BP) as IR,  
((total_revenue_AP - total_revenue_BP)/total_revenue_BP) * 100 as `IR%`
from cte1)

select product_name,category,`IR`,`IR%`, rank() over(order by`IR%` DESC ) as Rank_IR from cte2 limit 5

 # CTE1 - used Common_Table_Expression to determine the revenue before promotion and after promotion
 # CTE2 - to calculate the Incremental Revenue, Incremental Revenue %
 # RANK() - used window function to obtain the ranks of the categories based on their IR%
```

## Limitations and Challenges

One significant limitation encountered during the analysis is related to the handling of promotions with the 'BOGOF' (Buy One Get One Free) promotion type. The dataset does not accurately account for the quantity of the free item provided as part of the promotion. This limitation may lead to some discrepancies or misunderstandings in the analysis, particularly when evaluating the effectiveness of 'BOGOF' promotions and comparing them with other promotion types.

## Results and Insights

The analysis revealed several key insights:

- High-value products featured in 'BOGOF' promotions.
- Distribution of stores across different cities.
- Total revenue generated before and after each promotional campaign.
- Incremental sold quantity and revenue percentage during the Diwali campaign.
- Top 5 products ranked by incremental revenue percentage.

These insights can help AtliQ Mart make informed decisions for future promotional activities, optimize resource allocation, and improve overall sales performance.

## Conclusion

Overall, the analysis provides valuable insights into the performance of promotional campaigns conducted by AtliQ Mart during Diwali 2023 and Sankranti 2024. By leveraging data analytics, AtliQ Mart can enhance its marketing strategies, attract more customers, and drive higher sales during festive seasons.

## Visualizations

Explore the live Power BI dashboard for interactive visualizations:

[View Power BI Dashboard](https://app.powerbi.com/view?r=eyJrIjoiOGZmYjUxOTAtYWViYS00MjAxLTg4YzQtYjE3MDQyNmUxYmU1IiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)

## Future Work

Future work could include:
- Exploring additional datasets to gain deeper insights into customer behavior and preferences.
- Conducting more granular analysis on specific product categories or regions.
- Implementing machine learning models for predictive analytics to forecast sales and optimize promotional strategies.



