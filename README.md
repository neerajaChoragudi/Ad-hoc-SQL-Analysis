# Ad-hoc SQL Analysis


###### Challenge : Provide Insights to Management in Consumer Goods Domain

#### Domain:  Consumer Goods | Function: Executive Management

Atliq Hardwares (imaginary company) is one of the leading computer hardware producers in India and well expanded in other countries too.

However, the management noticed that they do not get enough insights to make quick and smart data-informed decisions. They want to expand their data analytics team by adding several junior data analysts. Tony Sharma, their data analytics director wanted to hire someone who is good at both tech and soft skills. Hence, he decided to conduct a SQL challenge which will help him understand both the skills.

#### Task:  

Imagine yourself as the applicant for this role and perform the following task

1.    Check ‘ad-hoc-requests.pdf’ - there are 10 ad hoc requests for which the business needs insights.
2.    You need to run a SQL query to answer these requests. 
3.    The target audience of this dashboard is top-level management - hence you need to create a presentation to show the insights.
4.    Be creative with your presentation, audio/video presentation will have more weightage.



###  1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.  

      select  distinct market,customer,region from dim_customer
      where customer ="Atliq Exclusive" and region = "APAC";

<img width="747" alt="MYSQLresult-1" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/79abb1e0-d900-4c5e-b5c0-f12104195459">


###  2. What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields, unique_products_2020 unique_products_2021      percentage_chg  

     select count(distinct case when fiscal_year = 2020 then product_code end) 
     as unique_products_2020,count(distinct case when fiscal_year = 2021 then product_code end) as unique_produts_2021,
     ((COUNT(DISTINCT CASE WHEN fiscal_year = 2021 THEN product_code END) - COUNT(DISTINCT CASE WHEN fiscal_year = 2020 THEN product_code END))
     /COUNT(DISTINCT CASE WHEN fiscal_year = 2020 THEN product_code END)) * 100 as percentage_change
     from fact_sales_monthly  where fiscal_year in (2020,2021);

<img width="745" alt="MYSQLresult-2" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/dc30d3ad-115d-4a0f-842e-e023e93229f5">


###  3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields, segment product_count  

     select segment,count(distinct product_code) as product_count from dim_product
     group by segment
     order by product_count desc;

<img width="753" alt="MYSQLresult-3" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/0472c644-f2b7-40f7-9034-141ea2e8cd9b">
 
###  4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields, segment, product_count_2020, product_count_2021 ,difference
 
     with cte1 as (select p.segment,
       count(distinct case when s.fiscal_year = 2020 then p.product_code end) as product_count_2020,
       count(distinct case when s.fiscal_year = 2021 then p.product_code end) as product_count_2021,
      (count(distinct case when s.fiscal_year = 2021 then p.product_code end)-
      (count(distinct case when s.fiscal_year = 2020 then p.product_code end)))/
      count(distinct case when s.fiscal_year = 2020 then p.product_code end) * 100 as pct_difference
      from  dim_product p 
      join fact_sales_monthly s
      on s.product_code = p.product_code
      group by p.segment)
        select * from cte1 
        order by pct_difference desc
        limit 1;

<img width="754" alt="MYSQLresult-4" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/4ee043b9-df04-4f6b-8083-2091d9428129">

###  5. Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields product_code, product ,manufacturing_cost*/ 

      select m.product_code,p.product,m.manufacturing_cost
      from fact_manufacturing_cost m 
      join dim_product p 
      on p.product_code = m.product_code
      where manufacturing_cost=
      (select max(manufacturing_cost) from fact_manufacturing_cost)or 
      manufacturing_cost= (select min(manufacturing_cost) from fact_manufacturing_cost)
      order by manufacturing_cost desc;

 <img width="742" alt="MYSQLresult-5" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/125f6904-0804-486b-aecc-82dea25b057a">


### 6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields,customer_code,customer ,average_discount_percentage (Let us consider it is asking the value  greater than the average value of pre_invoice_discount_pct) 

      select pre.customer_code,c.customer,
      round(avg(pre.pre_invoice_discount_pct),2) as highest_avg_pre_deduction_pct 
      from fact_pre_invoice_deductions pre
      join dim_customer c 
      on c.customer_code = pre.customer_code
      where c.market = "india" and fiscal_year = 2021
      group by c.customer,pre.customer_code
      order by highest_avg_pre_deduction_pct desc
      limit 5;

<img width="734" alt="MYSQLresult-6" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/c4cf2dcd-8b88-4d5e-8c3e-67364dcc8dad">


###  7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month .This analysis helps to get an idea of low and high-performing months and take strategic decisions.The final report contains these columns: Month ,Year ,Gross sales Amount

      with AtliqGrossSales as (select c.customer,monthname(s.date) as month,s.fiscal_year, 
      round(sum(s.sold_quantity*g.gross_price),2) as gross_sales_amount 
      from fact_sales_monthly s 
      join dim_customer c 
      on c.customer_code = s.customer_code 
      join fact_gross_price g 
      on g.product_code = s.product_code  and g.fiscal_year = s.fiscal_year
      group by month,s.fiscal_year,c.customer
      order by s.fiscal_year desc)
 
        select * from AtliqGrossSales 
        where customer = "Atliq Exclusive";

 <img width="731" alt="MYSQLresult-7" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/004e84a1-6b82-4d7b-8d00-c9e612ebd942">

###  8. In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity,Quarter ,total_sold_quantity 

        select 
     CASE 
          WHEN MONTH(date) IN(9 ,10, 11) THEN 'Q1'
          WHEN MONTH(date) IN(12 ,1, 2) THEN 'Q2'
          WHEN MONTH(date) IN(3 ,4, 5)THEN 'Q3'
          ELSE 'Q4'
     END AS fiscal_quarter,
      sum(sold_quantity) as total_sold_quantity
      from fact_sales_monthly
      where fiscal_year = 2020
      group by  fiscal_quarter
      order by fiscal_quarter,total_sold_quantity desc ;

<img width="738" alt="MYSQLresult-8" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/1efa93e9-3d30-4ada-9479-5b0a332456c7">

 
 ###  9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields channel ,gross_sales_mln ,percentage
 
      with cte9 as (select c.channel,
          round(sum(g.gross_price*s.sold_quantity)/100000,2) as gross_sales_mln
          from fact_sales_monthly s 
          join dim_customer c 
          on c.customer_code = s.customer_code 
          join fact_gross_price g 
          on g.product_code = s.product_code 
          where s.fiscal_year = 2021
          group by channel
         order by gross_sales_mln desc)
     select *,
     round(gross_sales_mln*100/sum(gross_sales_mln) over(),2) as percentage_contribution
     from cte9;

 <img width="739" alt="MYSQLresult-9" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/953679b1-050a-4885-93b4-7530cd7ab202">
 
 
 ###  10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields, division, product_code product, total_sold_quantity ,rank_order*/

      with cte10 as (select p.division,p.product_code,p.product,sum(s.sold_quantity) as total_sold_quantity 
         from fact_sales_monthly s 
         join dim_product p 
         on p.product_code = s.product_code
         where fiscal_year = 2021
         group by p.division,p.product_code),
     Acte11 as (select *,dense_rank() over(partition by division order by total_sold_quantity desc) as rnk
    from cte10)
        select * from cte11 
        where rnk <4;

<img width="723" alt="MYSQLresult-10" src="https://github.com/neerajaChoragudi/Ad-hoc-SQL-Analysis/assets/141207588/39b8db20-c7c9-45fc-8890-9e2ed12560a8">
  
  # Ad-hoc-SQL-Analysis
