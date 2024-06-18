# Pizzahut-sales-analysis

The provided data appears to be from a pizza restaurant’s database, detailing orders, pizzas, and their prices. Here’s a brief description suitable for a PowerPoint presentation:

Order Details: Lists individual pizza orders with their unique IDs, the specific pizza ordered, and the quantity.
Order Log: Records the date and time for each order, providing a timeline of sales.
Pizza Catalog: Describes different types of pizzas offered, categorized by ingredients like chicken, and details their specific toppings.
Pricing Information: Shows the size variants (small, medium, large) for each pizza type and their respective prices.
This structured data can be used to analyze sales trends, customer preferences, and pricing strategies. It’s essential for inventory management and business insights.


create database pizzahut;
use pizzahut;

create table orders (
order_id int not null,
order_date datetime not null,
order_time time not null,
primary key (order_id));

select * from pizzahut.orders;
create table order_details(
order_detail int not null,
order_id int not null,
pizza_id text not null,
quantity int not null,
primary key (order_detail)
);

select * from pizzahut.order_details;

-- Retrieve the total number of orders placed.
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;
-- Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS total_sales
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id;
    
-- Identify the highest-priced pizza.
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;

-- Identify the most common pizza size ordered.
select pizzas.size, count(order_details.order_detail) as order_count
from pizzas join order_details on pizzas.pizza_id = order_details.pizza_id
group by pizzas.size order by order_count desc;

-- List the top 5 most ordered pizza types along with their quantities.
SELECT 
    pizza_types.name, SUM(order_details.quantity) AS Quantities
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY Quantities DESC
LIMIT 5;

-- Intermediate: Join the necessary tables to find the total category of each pizza category ordered.
select pizza_types.category, sum(order_details.quantity) as Quantities
from pizza_types join pizzas on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category
order by Quantities desc;

-- Determine the distribution of orders by hour of the day.
select hour(order_time), count(order_id) from orders
group by hour(order_time);

-- Join relevant tables to find the category-wise distribution of pizzas.

select category, count(name) from pizza_types
group by category;

-- Group the orders by date and calculate the average number 
-- of pizzas ordered per day.
select round(avg(Quantities), 0) as average_pizzas_order_per_day from (select orders.order_date, sum(order_details.quantity) as Quantities
from orders join order_details on orders.order_id = order_details.order_id
group by orders.order_date) as order_quantity;

-- Determine the top 3 most ordered pizza types 
-- based on revenue.
SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;

-- Calculate the percentage contribution of each pizza category to 
-- total revenue.
SELECT 
    pizza_types.category,
    ROUND(SUM(order_details.quantity * pizzas.price) / (SELECT 
                    ROUND(SUM(order_details.quantity * pizzas.price),
                                2) AS total_sales
                FROM
                    order_details
                        JOIN
                    pizzas ON order_details.pizza_id = Pizzas.pizza_id) * 100,
            2) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;

-- Analyze the cumulative revenue generated over time.

select order_date, sum(revenue) over(order by order_date)as cum_revenue 
from (select orders.order_date, sum(order_details.quantity * pizzas.price) as revenue
from order_details join pizzas on order_details.pizza_id = pizzas.pizza_id
join orders on orders.order_id = order_details.order_id
group by orders.order_date order by revenue) as sales;

-- Determine the top 3 most ordered pizza types based 
-- on revenue for each pizza category.

select category, revenue
from (select category, name, revenue, 
rank() over(partition by category order by revenue desc) as rn
from (select pizza_types.category, pizza_types.name, 
sum(order_details.quantity * pizzas.price) as revenue
from pizza_types join pizzas on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details on pizzas.pizza_id = order_details.pizza_id
group by pizza_types.category, pizza_types.name
order by revenue) as a) as b
where rn<=3;
