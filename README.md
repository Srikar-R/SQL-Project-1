
# SQL End to End Project

## Accessing the database
use market;

![schema](https://user-images.githubusercontent.com/83721019/220250821-0a490130-13c8-4028-b47b-d5b6fd133f81.png)

### Inspecting customer dimension dataset
```sql
select *
from cust_dimen;

select count(*)
from cust_dimen;
```
__There are 1832 rows of data__

### Checking for missing values
```sql
select *
from cust_dimen
where Cust_id is null;
```
__No null values__
```sql
select *
from cust_dimen;
```
### Residence of Top 10 customers

```sql
select count(city) as 'No_of_Customers',city,state
from cust_dimen
group by city
order by No_of_Customers desc
limit 10;
```
__Mysore, Chennai and Pune are top 3 cities where most customers are from__

### Customer Segementation 
```sql
select count(Customer_Segment) as 'Segment_Type',Customer_segment
from cust_dimen
group by Customer_Segment
order by Segment_Type;
```
__Most customers are small businessess and corporates__

### Inspecting Market_fact_full
```sql
select *
from market_fact_full;
```
-- Observe that sales is not rounded to 2 decimals. 
```sql
select round(sales,2) as Rounded_Sales
from market_fact_full;
```
_Adding another column called Rounded Sales_
```sql
SET SQL_SAFE_UPDATES = 0;

update market.market_fact_full set sales=cast(round(sales,2) as decimal(10,2));

#select cast(sales as decimal(10,2))
#from market_fact_full;
```sql
select sales
from market_fact_full;
```
_Cast is used to convert data type. Same as convert but Cast can be read_

### Average Price
```sql
select avg(sales)
from market_fact_full;
```

__Rs.1775.879 is the average price of all products__


### Most profitable Customers

```sql
select sum(Profit) as Total_Profit,Customer_Name
from market_fact_full as m
inner join cust_dimen as c
on m.cust_id=c.cust_id
group by Customer_Name
order by Total_Profit desc
limit 20;
```


### Least Profitable Customers

```sql
select sum(Profit) as Total_Profit,Customer_Name,city
from market_fact_full as m
inner join cust_dimen as c
on m.cust_id=c.cust_id
group by Customer_Name
order by Total_Profit asc
limit 20 ;
```
__Clearly, these are customers who are purchasing on a frequent basis__

### Most profitable customer segment

```sql
select sum(Profit) as Total_Profit,Customer_Segment
from market_fact_full as m
inner join cust_dimen as c
on m.cust_id=c.cust_id
group by Customer_Segment
order by Total_Profit desc
limit 20;
```
__Corporate, Home office and small business__

### Least profitable cities

```sql
select sum(Profit) as Total_Profit,City,State
from market_fact_full as m
inner join cust_dimen as c
on m.cust_id=c.cust_id
group by City
order by Total_Profit asc;
```
__Kolkata,Patna,Coimbatore,Trivandrum,Delhi__

### Highest Shipping cost by city

```sql
select sum(Shipping_Cost) as Total_Shipping_Cost,City,State
from market_fact_full as m
inner join cust_dimen as c
on m.cust_id =c.cust_id
group by city
order by Total_Shipping_Cost desc;
```
__Since the inventory is in north of India, the trasporation costs to south of India is the highest__

### Inspecting orders dimension table

```sql
select *
from orders_dimen;
```

### Count of city by Order Priority

```sql
select count(Order_Priority) as No_of_orders,Order_priority,City,State
from orders_dimen as o
inner join market_fact_full as m
on m.ord_id=o.ord_id
inner join 
cust_dimen as c
on c.cust_id =m.cust_id
group by city
order by No_of_orders desc;
```
__We observe that most high and critical orders are from mysore, Delhi, Chennai. There is also a 'Not specified' that we have to look at__

## Inspecting product dimnesion

```sql
select *
from prod_dimen;
```
### Most profitable sub category

```sql
select sum(profit) as total_profit,Product_sub_category
from prod_dimen as p
inner join market_fact_full as m
on p.prod_id=m.prod_id
group by product_sub_category
order by total_profit desc;
```
```sql
select Product_sub_category,sum(order_quantity) as ordered_quantity,sum(profit) as total_profit
from prod_dimen as p
inner join market_fact_full as m
on p.prod_id=m.prod_id
group by product_sub_category
having sum(profit)<0
order by total_profit desc;
```
#### Possible reasons for making a net loss on these items
_Possible reasons: High Shipping Cost, Low Profit Margins (Low price, Hgh Discounts) {Either or both}_

```sql
select Product_sub_category,sum(order_quantity) as ordered_quantity,sum(profit) as total_profit,sum(shipping_cost) as Shipping_cost,city,sum(Discount),ship_mode
from prod_dimen as p
inner join market_fact_full as m
on p.prod_id=m.prod_id
inner join cust_dimen as c
on c.Cust_id=m.Cust_id
inner join shipping_dimen as s
on m.ship_id=s.ship_id
group by product_sub_category,city
having sum(profit)<0
order by total_profit desc;


```

### Most demanded product sub category
```sql
select sum(order_quantity) as ordered_quantity,Product_sub_category
from prod_dimen as p
inner join market_fact_full as m
on p.prod_id=m.prod_id
group by product_sub_category
order by ordered_quantity desc;
```

-- We can observe that there are some items that have only made loss

### Most profitable product category
```sql
select sum(profit) as total_profit,Product_category
from prod_dimen as p
inner join market_fact_full as m
on p.prod_id=m.prod_id
group by product_category
order by total_profit desc;
```


### Inspecting shipping dimenson
```sql
select *
from shipping_dimen;
```
### Costliest modes of transportation
```sql
select sum(shipping_cost) as total_Ship_cost, ship_mode
from shipping_dimen as s
inner join market_fact_Full as m
on m.ship_id=s.ship_id
group by ship_mode
order by total_ship_cost desc;
```
_Delivery truck is the costliest shipping mode_

### Highest demand by day 
```sql
select day(ship_date), count(ship_date)
from shipping_dimen
group by day(ship_date);
```
```sql
select sum(order_quantity) as total_orders,dayname(ship_date) as 'Day'
from market_fact_full as m
inner join shipping_dimen as s
on m.ship_id = s.ship_id
group by dayname(ship_date)
order by total_orders desc;
```
_Weekends are the most prefered day for shipping_

### Checking month name 

```sql
select sum(order_quantity) as total_orders,monthname(ship_date) as 'month'
from market_fact_full as m
inner join shipping_dimen as s
on m.ship_id = s.ship_id
group by monthname(ship_date)
order by total_orders desc;
```
_May, September and July_

```sql
select sum(order_quantity) as total_orders,year(ship_date) as 'year'
from market_fact_full as m
inner join shipping_dimen as s
on m.ship_id = s.ship_id
group by year(ship_date)
or```der by total_orders desc;
```
_2012 was the most demanded year_

# Comparing 2012 vs 2011

### Comparing profits of 2012 vs 2011
```sql
with year_comparison as(
select profit,lag(profit) over (order by (year(ship_date))) as profits,year(ship_date) as 'year'
from shipping_dimen as s
inner join market_fact_full as m
on m.ship_id=s.ship_id
where year(ship_date) between 2010 and 2012
group by year(ship_date))
```
```sql
select year,profit,round((profit-profits)*100/(profit),2) as change_in_profit
from year_comparison;
```
_95% increase in profits_

### Month wise comparison of profit
```sql
with monthwise_comparison as (
select monthname(ship_date) as months,year(ship_date) as years,profit,lag(profit,12) over (order by year(ship_date),month(ship_date)) as profit1
from shipping_dimen as s
inner join market_fact_full as m
on s.ship_id=m.ship_id
where year(ship_date)=2011 or year(ship_date)=2012
group by month(ship_date),year(ship_date)
order by year(ship_date),month(ship_date))
```
```sql
select months,profit as '2011',profit1 as '2012',(profit-profit1)*100/profit as perc_change
from monthwise_comparison
where (profit-profit1)*100/profit is not null
;
```
__We observe that March,april,august and november are the months the business did not do well compared to the previous year__

### Shipping Cost by city and delivery mode
```sql
select city,ship_mode,sum(shipping_cost) as total_shipping_cost
from shipping_dimen as s
inner join market_fact_full as m 
on m.ship_id=s.ship_id 
inner join cust_dimen as c
on m.cust_id=c.cust_id 
group by city
order by sum(shipping_cost) desc;
```


