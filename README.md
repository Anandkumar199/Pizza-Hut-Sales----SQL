# Pizza-Hut-Sales----SQL

I Have Used 4 tables called Order_details , Orders , Pizza_Types , Pizzas.
It has 20K+ orders and more than 30 types of pizza types.
I have analyzed all the tables and performed queries.
I joined 2 tables and 3 tables using joins to combine the data.
Performed window functions for calculations across rows and for running totals , rankings , moving averages etc.
Used CTEs for Breaking down large queries into smaller , logical parts to read and maintain.


          -- Retrieve the total number of orders placed.
 
  select count(order_id) as Total_Orders from orders;


          -- Calculate the total revenue generated from pizza sales

SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS Total_Sales
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
ORDER BY price DESC
LIMIT 1;


            -- Identify the most common pizza size ordered.

 SELECT 
    pizzas.size,
    COUNT(order_details.order_details_id) AS Total_Count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY Total_count DESC;


            -- List the top 5 most ordered pizza types along with their quantities.
 
SELECT 
    pizza_types.name,
    SUM(order_details.quantity) AS Total_Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY total_quantity DESC
LIMIT 5;


             -- Join the necessary tables to find the total quantity of each pizza category ordered.-- 

SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;


            -- Determine the distribution of orders by hour of the day.

SELECT 
    HOUR(order_time) AS Hour, COUNT(order_id) AS Order_Count
FROM
    orders
GROUP BY hour;


              -- Join relevant tables to find the category-wise distribution of pizzas.

SELECT 
    category, COUNT(name)
FROM
    pizza_types
GROUP BY category;


              -- Group the orders by date and calculate the average number of pizzas ordered per day.

SELECT 
    ROUND(AVG(quantity), 0)
FROM
    (SELECT 
        orders.order_date, SUM(order_details.quantity) AS Quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS Order_quantity;


              -- Determine the top 3 most ordered pizza types based on revenue.

SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS Revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;


              -- Calculate the percentage contribution of each pizza type to total revenue.

select pizza_types.category , 
round(sum(order_details.quantity * pizzas.price) / (select
round(sum(order_details.quantity * pizzas.price),2) as Total_Sales
from order_details
join pizzas
on pizzas.pizza_id = order_details.pizza_id) * 100 , 2 ) as Revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category order by revenue desc;


              -- Analyze the cumulative revenue generated over time.

select order_date , 
sum(revenue) over(order by order_date) as cum_revenue
from
(select orders.order_date ,
sum(order_details.quantity * pizzas.price) as Revenue
from order_details join pizzas
on order_details.pizza_id = pizzas.pizza_id
join orders
on orders.order_id = order_details.order_id
group by orders.order_date) as Sales;


              -- Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select name , revenue from
(select category , name , revenue , 
rank() over (partition by category order by revenue desc) as rn
from
(select pizza_types.category , pizza_types.name , 
sum((order_details.quantity) * pizzas.price) as Revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category , pizza_types.name) as a) as b
where rn <= 3;
