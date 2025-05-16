-- Retrieve the total number of orders placed
SELECT COUNT(order_id) AS total_orders FROM orders;

-- Identify the highest-priced pizza
SELECT
    pizza_types.name,
    pizzas.price
FROM
    pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1 OFFSET 0;

-- Identify the most common pizza size ordered
SELECT
    pizzas.size,
    COUNT(order_details.order_details_id) AS pizzas_count
FROM
    pizzas
JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY pizzas_count DESC
LIMIT 2;

-- List the top 5 most ordered pizza types along with their quantities
USE sqlproject;

SELECT
    pizza_types.name,
    SUM(order_details.quantity) AS quantity
FROM
    pizzas
JOIN pizza_types ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;

-- Find the total quantity of each pizza category ordered
SELECT
    pizza_types.category,
    SUM(order_details.quantity) AS quantity
FROM
    pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;

-- Determine the distribution of orders by hour of the day
SELECT
    HOUR(time) AS hour,
    COUNT(order_id) AS count
FROM orders
GROUP BY HOUR(time);

-- Find category-wise distribution of pizzas
SELECT
    category,
    COUNT(name)
FROM
    pizza_types
GROUP BY category;

-- Calculate average number of pizzas ordered per day
SELECT ROUND(AVG(quantity), 0) FROM (
    SELECT
        orders.date,
        SUM(order_details.quantity) AS quantity
    FROM orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.date
) AS order_quantity;

-- Determine the top 3 most ordered pizza types based on revenue
SELECT
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS revenue
FROM
    pizza_types
JOIN pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 5;

-- Calculate percentage contribution of each pizza type to total revenue
SELECT
    pizza_types.category,
    ROUND(SUM(order_details.quantity * pizzas.price) /
    (
        SELECT ROUND(SUM(order_details.quantity * pizzas.price), 2) AS total_sales
        FROM order_details
        JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    ) * 100, 2) AS revenue
FROM
    pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;

-- Analyze cumulative revenue generated over time
SELECT
    date,
    SUM(revenue) OVER (ORDER BY date) AS cum_revenue
FROM (
    SELECT
        orders.date,
        SUM(order_details.quantity * pizzas.price) AS revenue
    FROM order_details
    JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.date
) AS sales;

-- Determine top 3 most ordered pizza types by revenue for each category
SELECT
    name,
    revenue
FROM (
    SELECT
        category,
        name,
        revenue,
        RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM (
        SELECT
            pizza_types.category,
            pizza_types.name,
            SUM(order_details.quantity * pizzas.price) AS revenue
        FROM
            pizza_types
        JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
        GROUP BY pizza_types.category, pizza_types.name
    ) AS a
) AS b
WHERE rn <= 3;
