create database if not exists dannys_diner;

use dannys_diner;

CREATE TABLE sales (
  customer_id VARCHAR(1),
  order_date DATE,
  product_id INTEGER
);
--------------------------------------
INSERT INTO sales
  (customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
--------------------------------------
CREATE TABLE menu (
  product_id INTEGER,
  product_name VARCHAR(5),
  price INTEGER
);
--------------------------------------
INSERT INTO menu
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
--------------------------------------
CREATE TABLE members (
  customer_id VARCHAR(1),
  join_date DATE
);
--------------------------------------
INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
  
-----------------------------------------------------------------------------------
# extracting all the data present into 'sales' table
select * from sales;

# extracting all the data present into 'menu' table
select * from menu;

# extracting all the data present into 'members' table
select * from members;

-----------------------------------------------------------------------------------
/*
 1. What is the total amount each customer spent at the restaurant?
*/
select s.customer_id as "Customer ID", sum(m.price) as "Total Amount"
from sales as s
inner join menu as m on s.product_id = m.product_id
group by s.customer_id
order by sum(m.price) desc;

-----------------------------------------------------------------------------------
/*
 2. How many days has each customer visited the restaurant?
*/
select s.customer_id as "Customer ID", count(distinct(s.order_date)) as "No. of Days Visited"
from sales as s
group by s.customer_id
order by count(distinct(s.order_date)) desc;

-----------------------------------------------------------------------------------
/*
 3. What was the first item from the menu purchased by each customer?
*/
with purchase_rank as (
			select s.customer_id, s.order_date, m.product_name, dense_rank() over (partition by s.customer_id order by s.order_date) as purchase_order
                        from sales as s
                        inner join menu as m on s.product_id = m.product_id
                        )
select pr.customer_id as "Customer ID", pr.product_name as "Item Name"
from purchase_rank as pr
where purchase_order = 1
group by pr.customer_id;

-----------------------------------------------------------------------------------
/*
 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
*/
select m.product_name as "Item Name", count(s.product_id) as "No of Time Purchased"
from menu as m
inner join sales as s on m.product_id = s.product_id
group by m.product_name
order by count(s.product_id) desc limit 1;

-----------------------------------------------------------------------------------
/*
 5. Which item was the most popular for each customer?
*/
with popular_rank as (
			select s.customer_id, m.product_name, count(s.product_id) as no_of_orders, 
                        dense_rank() over (partition by s.customer_id order by count(s.product_id) desc) as "rank"
                        from sales as s
                        inner join menu as m on s.product_id = m.product_id
                        group by s.customer_id, m.product_name
                        )
select pr.customer_id as "Customer ID", pr.product_name as "Item Name", pr.no_of_orders as "No of Times ordered"
from popular_rank as pr
where pr.rank = 1;

-----------------------------------------------------------------------------------
/*
 6. Which item was purchased first by the customer after they became a member?
*/
with became_member as(
			select s.customer_id, m.product_name, s.order_date, me.join_date,
                        dense_rank() over (partition by s.customer_id order by s.order_date) as "rank"
                        from sales as s
                        inner join menu as m on s.product_id = m.product_id
                        inner join members as me on s.customer_id = me.customer_id
                        where s.order_date >= me.join_date
                        )
select bm.customer_id as "Customer ID", bm.product_name as "Item Name"
from became_member as bm
where bm.rank =1;

-----------------------------------------------------------------------------------
/*
 7. Which item was purchased just before the customer became a member?
*/
with became_member as(
			select s.customer_id, m.product_name, s.order_date, me.join_date,
                        dense_rank() over (partition by s.customer_id order by s.order_date desc) as "rank"
                        from sales as s
                        inner join menu as m on s.product_id = m.product_id
                        inner join members as me on s.customer_id = me.customer_id
                        where s.order_date < me.join_date
                        )
select bm.customer_id as "Customer ID", bm.product_name as "Item Name"
from became_member as bm
where bm.rank =1;

-------------------------------------------------------------------------------------
/*
 8. What is the total items and amount spent for each member before they became a member?
*/
select s.customer_id as "Customer ID", count(distinct(s.product_id)) as "Total Items", sum(m.price) as "Total Price"
from sales as s
inner join menu as m on s.product_id = m.product_id
inner join members as me on s.customer_id = me.customer_id
where s.order_date < me.join_date
group by s.customer_id;

--------------------------------------------------------------------------------------
/*
 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
*/
select s.customer_id as "Customer ID", sum(case
						when m.product_name = "sushi" then m.price * 20
						else m.price * 10
					   end) as "Total Points"
from sales as s
inner join menu as m on s.product_id = m.product_id
group by s.customer_id
order by sum(case
		when m.product_name = "sushi" then m.price * 20
		else m.price * 10
	     end) desc;
            
-------------------------------------------------------------------------------------
/*
 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, 
not just sushi - how many points do customer A and B have at the end of January?
*/
select s.customer_id as "Customer ID", sum(case
						when s.order_date >= me.join_date and s.order_date <= date_add(me.join_date, interval 1 week) then m.price * 2
                                             	else m.price
                                            end) as "Points Earned", s.order_date
from sales as s
inner join menu as m on s.product_id = m.product_id
left join members as me on s.customer_id = me.customer_id
where s.order_date >= "2021-01-01" and order_date <= "2021-01-31"
group by s.customer_id
order by sum(case
		when s.order_date >= me.join_date and s.order_date <= date_add(me.join_date, interval 1 week) then m.price * 2
		else m.price
	     end) desc;

---------------------------------------------------------------------------------------
/*
Join All The Things: -
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights 
without needing to join the underlying tables using SQL.
*/
select s.customer_id as "Customer ID", s.order_date as "Order Date", m.product_name as "Product Name", m.price as "Price",
case
    when s.order_date >= me.join_date then "Y"
    else "N"
end as "Member"
from sales as s
inner join menu as m on s.product_id = m.product_id
left join members as me on s.customer_id = me.customer_id;

-----------------------------------------------------------------------------------------
/*
Rank All The Things: -
Danny also requires further information about the ranking of customer products, 
but he purposely does not need the ranking for non-member purchases 
so he expects null ranking values for the records when customers are not yet part of the loyalty program.
*/
with all_together as(
			select s.customer_id, s.order_date, m.product_name, m.price,
			case
    			    when s.order_date >= me.join_date then "Y"
    			    else "N"
			end as member
			from sales as s
			inner join menu as m on s.product_id = m.product_id
			left join members as me on s.customer_id = me.customer_id
			)
select at.customer_id as "Customer ID", at.order_date "Order Date", at.product_name "Product Name", at.price as "Price",
case
    when member = "N" then null
    else rank() over (partition by at.customer_id, at.member order by at.order_date)
end as "Ranking"
from all_together as at;
