# _Retail Sales Analysis_
## Project Overview
This project looks at the business data to analyze various aspects such as product sales performance, customer behaviour, product demand, and staff performance. I have used SQLite to extract meaningful information from a relational database.

## Data Sources
The primary dataset used for the analysis is the "stores.db" file available in the repository within 'data'.
## Tools
SQLite - Data Analysis
## Data
Here are the different tables with their respective attributes and total number of rows.

| Table Name    | Number of Attributes | Row Count |
|---------------|----------------------|-----------|
| Customers     | 13                   | 122       |
| Products      | 9                    | 110       |
| ProductLines  | 4                    | 7         |
| Orders        | 7                    | 326       |
| OrderDetails  | 5                    | 2996      |
| Payments      | 4                    | 273       |
| Employees     | 8                    | 23        |
| Offices       | 9                    | 7         |

To better understand the relationship take a look at the schema available in the above folders.
## Exploratory Data Analysis
It involves exploring the stored .bd file data to answer questions, such as:<br/>
##### Q1. What are the top 10 products which have the highest demand?<br/>

```
SELECT p.productCode, p.productName, p.productLine
       , round(sum(o.quantityOrdered)/p.quantityInStock,2) as low_stock
  from products p 
  join orderdetails o
    on p. productCode = o.productCode
 group by p.productCode
 order by low_stock desc
 LIMIT 10;
```
NOTE: Here I have taken into account the stock ratio. It is a good indicator to consider the products that are selling fast.<br/>
It is calculated by the sum of each product ordered divided by the quantity of product in stock.<br/>
Low Stock = Sum(quantity ordered) / quantity in stock<br/>
Higher the ratio means we need to restock quickly because the quantity in stock is low hence I ordered it in descending.

#### Q2. How Should We Match the Marketing and Communication Strategies to Customer Behavior?<br/>
```
with 
profit_per_customer as(
select o.customerNumber
       ,sum(od.quantityOrdered* (od.priceEach-p.buyPrice)) as profit
  from products p
  join orderdetails od
    on p.productCode = od.productCode
  join orders o
    on o.orderNumber = od.orderNumber
 group by customerNumber
)
select c.contactLastName||" "||c.contactFirstName as contact_name
       ,city
       ,country
       ,profit
  from profit_per_customer as ppc
 cross join customers as c
    on ppc.customerNumber = c.customerNumber
 order by profit DESC
 limit 5
```
NOTE: The above code shows the top most valuable customers that can be termed the VIP customers.<br/>
They will help to steer the marketing and communication because of their in.<br/>
Profit each item = Selling Price of each item − Buy Price of each item

#### Q3. Show the least valuable customers<br/>
```
with 
profit_per_customer as(
select o.customerNumber
       ,sum(od.quantityOrdered* (od.priceEach-p.buyPrice)) as profit
  from products p
  join orderdetails od
    on p.productCode = od.productCode
  join orders o
    on o.orderNumber = od.orderNumber
 group by customerNumber
)
select c.contactLastName
        ,c.contactFirstName
	    ,city
	    ,country
	    ,profit
  from profit_per_customer as ppc
 cross join customers as c
    on ppc.customerNumber = c.customerNumber
 order by profit
 limit 5
```
NOTE: We do not consider the above 5 customers when making decisions because the sales are low.<br/>
#### Q4. Rank the top sales representatives who have achieved the highest sales figures in all the territories<br/>
```
WITH
employee_sales_table as (
select e.employeeNumber
       ,e.firstName||" "||e.lastName as employee_name
       ,round(sum(od.quantityOrdered*od.priceEach),2) as total_amount
  from customers c
  join orders o
    on o.customerNumber = c.customerNumber
  join orderdetails od
    on od.orderNumber = o.orderNumber
  join employees e
    on c.salesRepEmployeeNumber=e.employeeNumber
 group by c.salesRepEmployeeNumber
)
select *
       , rank() over(order by total_amount desc) as rank
  from employee_sales_table
 order by total_amount DESC
 limit 5
```
The table shows the top 5 sales representatives among all the territories.<br/>
The payment amount has not been considered since many customers have bought items independently.<br/>

## Findings
Findings are in a Word document in (.docx) format under the results subcategory.
## Recommendations 
Recommendations are in a Word document in (.docx) format under the results subcategory.
## Limitations
Limitations are in a Word document in (.docx) format under the results subcategory.
