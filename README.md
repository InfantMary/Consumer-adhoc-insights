This GitHub repository contains a set of SQL queries designed to analyze data related to Atliq Hardware, a computer hardware company. These queries were utilized in an SQL challenge to evaluate candidates' proficiency in data analysis and SQL.
**Overview**
Atliq Hardware manufactures and distributes computer hardware across different regions. The Data Analytics Director organized an SQL challenge to assess candidates' skills, and this repository houses the queries used in that challenge.
**Queries**
Here's a brief summary of the queries included in this repository:
1.	**Market Analysis:** Lists the markets where the customer "Atliq Exclusive" operates in the APAC region.
select *from dim_customer where customer = "AtliQ Exclusive" and  region ="APAC";
2.	**Product Comparison:** Calculates the percentage increase in unique products between 2021 and 2020.
with cte1 as 
(select count(distinct product_code) as unique_product_2020
 from fact_sales_monthly 
 where fiscal_year = 2020),
 cte2 as 
(select count(distinct product_code) as unique_product_2021
 from fact_sales_monthly 
 where fiscal_year = 2021)

select  c1.unique_product_2020,c2.unique_product_2021, 
		round((c2.unique_product_2021 - c1.unique_product_2020)*100 /c1.unique_product_2020,2) as percentage_chg
from cte1 c1
join cte2 c2

3.	**Segment-wise Product Count:** Provides a report of unique product counts for each segment, sorted in descending order.
select segment, count(distinct product_code) as product_count from dim_product 
group by segment
order by product_count desc
4.	**Segment Analysis:** Identifies the segment with the most increase in unique products in 2021 compared to 2020.

with cte1 as
(
Select p.segment,count(distinct s.product_code) as product_count_2020
from fact_sales_monthly s
join dim_product p
using(product_code)
where s.fiscal_year="2020"
group by p.segment
),
cte2 as(
Select p.segment,count(distinct s.product_code) as product_count_2021
from fact_sales_monthly s
join dim_product p
using(product_code)
where s.fiscal_year="2021"
group by p.segment
)

Select c1.segment,product_count_2020,product_count_2021,(product_count_2021-product_count_2020) as Difference
From cte1 c1
JOin cte2 c2
on c1.segment=c2.segment
order by  Difference desc

5.	**Manufacturing Costs:** Lists products with the highest and lowest manufacturing costs.
SELECT
    m.product_code,
    p.product,
    m.manufacturing_cost
FROM dim_product p
join fact_manufacturing_cost m
using(product_code)
WHERE
    manufacturing_cost = (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost)
    or manufacturing_cost = (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost)
order by manufacturing_cost desc
6.	**Customer Discounts:** Generates a report of the top 5 customers with the highest average pre-invoice discount percentage in 2021 in the Indian market.
select
d.customer_code,
d.customer,
round(avg(pre_invoice_discount_pct)*100,2) as average_discount_percentage
from dim_customer d
join fact_pre_invoice_deductions f
on d.customer_code = f.customer_code
where f.fiscal_year=2021 and 
	  d.market='India'
group by d.customer_code,d.customer
order by average_discount_percentage desc
limit 5 
7.	**Gross Sales Analysis:** Provides a complete report of the gross sales amount for the customer "Atliq Exclusive" for each month.
SELECT
    CONCAT(MONTHNAME(f.date)) AS month_name,
    f.fiscal_year AS year,
    CONCAT(round((SUM(f.sold_quantity * g.gross_price) / 1000000),2), 'M') AS gross_sales_amnt
FROM
    fact_sales_monthly f
    JOIN fact_gross_price g USING (product_code, fiscal_year)
    JOIN dim_customer c USING (customer_code)
WHERE
    c.customer = 'AtliQ Exclusive'
GROUP BY
    month_name,
    year
ORDER BY
   year ASC;
    
8.	**Quarterly Analysis:** Determines the quarter of 2020 with the maximum total sold quantity.
Select
case
when month(date) in (9,10,11) then "Q1"
when month(date) in (12,1,2) then "Q2"
when month(date) in (3,4,5) then "Q3"
when month(date) in (6,7,8) then "Q4"
end as Quarters,
    CONCAT(ROUND(SUM(sold_quantity)/1000000, 2), 'M') AS Total_sold_quantity_mln
FROM 
    fact_sales_monthly
WHERE 
    fiscal_year = 2020
GROUP BY 
    quarters
9.	**Channel Contribution:** Identifies the channel that contributed the most to gross sales in the fiscal year 2021.
WITH cte1 AS (
    SELECT
        c.channel,
        Concat(ROUND(SUM((g.gross_price * s.sold_quantity)) / 1000000, 2),'M') AS Gross_Sales_Mln
    FROM
        fact_gross_price g
    JOIN
        fact_sales_monthly s USING (product_code,fiscal_year)
    JOIN
        dim_customer c USING (customer_code)
    GROUP BY
        c.channel
)
SELECT
    channel, Gross_Sales_Mln,
Concat(ROUND((Gross_Sales_Mln / SUM(Gross_Sales_Mln) OVER ()) * 100, 2),'%')  AS Percentage_Contribution
FROM
    cte1

Order by Gross_Sales_Mln
10.	**Top Products:** Lists the top 3 products in each division with the highest total sold quantity in the fiscal year 2021.
WITH RankedProducts AS (
    SELECT
        p.division AS division,
        p.product_code AS product_code,
        p.product AS product,
        SUM(f.sold_quantity) AS total_sold_quantity,
        ROW_NUMBER() OVER (PARTITION BY p.division ORDER BY SUM(f.sold_quantity) DESC) AS rnk
    FROM 
        fact_sales_monthly f
    JOIN 
        dim_product p USING (product_code)
    WHERE
        fiscal_year = 2021
    GROUP BY
        p.division,
        p.product_code,
        p.product
        
)

SELECT
    division,
    product_code,
    product,
    total_sold_quantity,
    rnk
FROM
    RankedProducts
WHERE
    rnk <= 3;

**Usage**
To utilize these SQL queries, you can copy and paste them into your preferred SQL database management tool or environment.
**Contributing**
If you have additional queries or improvements to existing queries, you're welcome to contribute to this repository by creating a pull request.
This README provides clear instructions and insights into the purpose and content of the SQL queries in the repository, making it accessible for users to understand and utilize them effectively

# Consumer-adhoc-insights
