# _Model Cars Database Analysis_
<img src="https://github.com/srishupadhyay/Retail_Insights/assets/171460110/8b8f066c-8827-4e4f-b064-077ee1bf79d8" alt="car" width="300"/>

## Project Overview
This project looks at the sales data to analyze aspects such as product sales performance, customer behaviour, product demand, and staff performance. I have used SQLite to extract meaningful information from a relational database.

## Data Sources
The primary dataset used for the analysis is the "stores.db" file available in the repository within 'Data'.
## Tools
[SQLite](https://sqlitebrowser.org/dl/)
## Data
Here are the different tables with a total number of  attributes and rows.

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

## Results
1. The highest-selling and in-demand products are
```
| Product Code | Product Name                               | Product Line     | Quantity Sold |
|--------------|--------------------------------------------|------------------|---------------|
| S24_2000     | 1960 BSA Gold Star DBD34                   | Motorcycles      | 67.0          |
| S12_1099     | 1968 Ford Mustang                          | Classic Cars     | 13.0          |
| S32_4289     | 1928 Ford Phaeton Deluxe                   | Vintage Cars     | 7.0           |
| S32_1374     | 1997 BMW F650 ST                           | Motorcycles      | 5.0           |
| S72_3212     | Pont Yacht                                 | Ships            | 2.0           |
| S700_3167    | F/A 18 Hornet 1/72                         | Planes           | 1.0           |
| S700_1938    | The Mayflower                              | Ships            | 1.0           |
| S50_4713     | 2002 Yamaha YZR M1                         | Motorcycles      | 1.0           |
| S32_3522     | 1996 Peterbilt 379 Stake Bed with Outrigger| Trucks and Buses | 1.0           |
| S18_2795     | 1928 Mercedes-Benz SSK                     | Vintage Cars     | 1.0           |
```
The higher the stock rate, the higher the demand is.<br/>
**We should focus on restocking the above products otherwise timely deliveries won't be possible and can result in bad customer experience.**<br/>
2. Strategies should be according to these VIP customers with the highest profit
```
| Name            | City        | Country     | Sales (in USD) |
|-----------------|-------------|-------------|----------------|
| Freyre Diego    | Madrid      | Spain       | 326,519.66     |
| Nelson Susan    | San Rafael  | USA         | 236,769.39     |
| Young Jeff      | NYC         | USA         | 72,370.09      |
| Ferguson Peter  | Melbourne   | Australia   | 70,311.07      |
| Labrune Janine  | Nantes      | France      | 60,875.30      |
```
**The above customers should be constantly attended to since they bring maximum sales for us.**
3. Least engaged customers are<br/>
```
| Name           | City       | Country   | Sales (in USD) |
|----------------|------------|-----------|----------------|
| Young Mary     | Glendale   | USA       | 2,610.87       |
| Taylor Leslie  | Brickhaven | USA       | 6,586.02       |
| Ricotti Franco | Milan      | Italy     | 9,532.93       |
| Schmitt Carine | Nantes     | France    | 10,063.80      |
| Smith Thomas   | London     | UK        | 10,868.04      |
```
With VIP and least engaged customers, we can proceed with strategizing who's input to consider.<br/>
**We should collaborate more with the above customers to improve relations.**
4. Top 5 sales representatives among all the territories.

```
| ID   | Name             | Sales (in USD) | Rank |
|------|------------------|----------------|------|
| 1370 | Gerard Hernandez | 1,258,577.81   | 1    |
| 1165 | Leslie Jennings  | 1,081,530.54   | 2    |
| 1401 | Pamela Castillo  | 868,220.55     | 3    |
| 1501 | Larry Bott       | 732,096.79     | 4    |
| 1504 | Barry Jones      | 704,853.91     | 5    |
```
They are from all the locations and territories.


