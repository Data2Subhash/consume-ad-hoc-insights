# consume-ad-hoc-insights

# Atliq Hardware SQL Challenge

Welcome to the Atliq Hardware SQL Challenge! This repository contains a set of SQL queries, each designed to answer specific questions related to Atliq Hardware, a computer hardware company. These queries were used in an SQL challenge to assess candidates' skills in data analysis and SQL.

## Table of Contents

1. [Overview](#overview)
2. [Queries](#queries)
3. [Contributing](#contributing)

## Overview

Atliq Hardware is a computer hardware company that manufactures and distributes hardware across four main regions. To make data-informed decisions, the Data Analytics Director organized an SQL challenge to assess candidates' skills in both technical and soft skills. This repository contains the SQL queries used in that challenge.

## Queries

### Query 1: Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

**Answer:** select distinct(market) from dim_customer where customer='Atliq Exclusive' and region = 'APAC'

### Query 2: What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields,unique_products_2020, unique_products_2021 and percentage_chg

**Answer:** select A.X as unique_product_2020, B.Y as unique_product_2021, ((Y-X)/X*100) as percentage_chg from
(
(select count(distinct(product_code)) as X
from fact_sales_monthly where fiscal_year=2020) A,
(select count(distinct(product_code)) as Y
from fact_sales_monthly where fiscal_year=2021) B
)

### Query 3: Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields, segment, product_count

**Answer:** select segment,count(distinct(product_code)) as product_count from dim_product
group by segment order by product_count desc

### Query 4: Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields,segment, product_count_2020, product_count_2021 and difference

**Answer:** WITH CTE1 AS 
	(SELECT P.segment AS A , COUNT(DISTINCT(FS.product_code)) AS B 
    FROM dim_product P, fact_sales_monthly FS
    WHERE P.product_code = FS.product_code
    GROUP BY FS.fiscal_year, P.segment
    HAVING FS.fiscal_year = "2020"),
CTE2 AS
    (
	SELECT P.segment AS C , COUNT(DISTINCT(FS.product_code)) AS D 
    FROM dim_product P, fact_sales_monthly FS
    WHERE P.product_code = FS.product_code
    GROUP BY FS.fiscal_year, P.segment
    HAVING FS.fiscal_year = "2021"
    )  SELECT CTE1.A AS segment, CTE1.B AS product_count_2020, CTE2.D AS product_count_2021, (CTE2.D-CTE1.B) AS difference  
FROM CTE1, CTE2
WHERE CTE1.A = CTE2.C 
order by difference desc;

### Query 5: Get the products that have the highest and lowest manufacturing costs.The final output should contain these fields,product_codeproduct and manufacturing_cost

**Answer:** select p.product,p.product_code,f.manufacturing_cost from fact_manufacturing_cost f 
JOIN dim_product p ON f.product_code=p.product_code
where f.manufacturing_cost in (
  select MAX(manufacturing_cost) from fact_manufacturing_cost
  union
  select MIN(manufacturing_cost) from fact_manufacturing_cost
)
order by f.manufacturing_cost desc;

### Query 6: Generate a report which contains the top 5 customers who received anaverage high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields, customer_code, customer, average_discount_percentage

**Answer:** select c.customer,c.customer_code,(f.pre_invoice_discount_pct) as average_discount_percentage
from fact_pre_invoice_deductions f 
JOIN dim_customer c ON f.customer_code=c.customer_code 
where f.fiscal_year=2021 and c.market='india'
order by average_discount_percentage desc
limit 5

### Query 7: Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns:Month, Year and Gross sales Amount

**Answer:** set sql_mode="";
select month(f.date) as month,monthname(date) as monthname, f.fiscal_year,
concat(round(sum(g.gross_price*f.sold_quantity)/1000000,2), 'M') as gross_sales_amount from fact_gross_price g
right JOIN fact_sales_monthly f ON 
f.product_code=g.product_code
JOIN dim_customer c ON 
f.customer_code=c.customer_code
where c.customer='Atliq Exclusive' 
Group by month,f.fiscal_year
order by f.fiscal_year


### Query 8: In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity, Quarter and total_sold_quantity

**Answer:** select concat('Q',ceiling(a.month/3)) as quarters ,sum(sold_quantity) as sold_quantity from 
(select month(DATE_ADD(date, INTERVAL 4 MONTH)) as month,fiscal_year,sold_quantity 
from fact_sales_monthly where fiscal_year = 2020) a
group by quarters
order by sold_quantity desc

### Query 9: Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields, channel, gross_sales_mln and percentage

**Answer:** set sql_mode="";

with cte1 as(select c.channel,sum(f.sold_quantity*g.gross_price) as gross_sales from dim_customer c
JOIN fact_sales_monthly f ON 
f.customer_code=c.customer_code
JOIN fact_gross_price g ON 
g.product_code=f.product_code
where f.fiscal_year=2021
group by channel
order by gross_sales desc) 

select  channel,round(gross_sales/1000000,2) as gross_sales,round((gross_sales*100)/sum(gross_sales) over(),2) as pct_chg from cte1 

### Query 10: Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields, division and product_code

**Answer:** with cte1 as(select p.division,f.product_code,p.product,sum(f.sold_quantity) as sold_quantity,rank() 
over(partition by division order by sum(sold_quantity) desc) as rank_order from fact_sales_monthly f
JOIN dim_product p ON 
f.product_code=p.product_code
where fiscal_year=2021 
group by p.product_code
)
select * from cte1 where rank_order<=3


## Usage

To use these SQL queries, you can copy and paste them into your preferred SQL database management tool or environment.

## Contributing

If you have additional queries or improvements to existing queries, feel free to contribute to this repository by creating a pull request.


This README provides an organized and informative structure for your GitHub repository, making it easy for others to understand and use your SQL queries.
