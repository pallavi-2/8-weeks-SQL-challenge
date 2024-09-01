# Solutions

### **A. Pizza Metrics**

1. How many pizzas were ordered?

```sql
SELECT 
	COUNT(order_id) AS total_count 
FROM customer_orders_temp
```

| total_count |
| --- |
| 14 |
2. How many unique customer orders were made?

```sql
SELECT 
	COUNT(DISTINCT order_id) as total_unique_order
FROM customer_orders_temp
```

| total_unique_order |
| --- |
| 10 |
3. How many successful orders were delivered by each runner?

```sql
SELECT 
	runner_id,
	COUNT(order_id) AS total_successful_orders
FROM runner_orders_temp
WHERE cancellation not like '%Cancellation'
GROUP BY runner_id
ORDER BY runner_id
```

| runner_id | total_successful_orders |
| --- | --- |
| 1 | 4 |
| 2 | 3 |
| 3 | 1 |
4. How many of each type of pizza was delivered?

```sql
SELECT
	pizza_id,
	COUNT(pizza_id) as total_delivered
FROM customer_orders_temp
JOIN runner_orders_temp
	ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation not like '%Cancellation'
GROUP BY customer_orders_temp.pizza_id
ORDER BY pizza_id
```

| pizza_id | total_delivered |
| --- | --- |
| 1 | 9 |
| 2 | 3 |
5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT
 customer_id,
 pizza_name,
 COUNT(customer_orders_temp.pizza_id) as total_order
FROM customer_orders_temp 
JOIN pizza_runner.pizza_names
 ON customer_orders_temp.pizza_id = pizza_names.pizza_id
GROUP BY customer_id,pizza_name
ORDER BY customer_id
```

| customer_id | pizza_name | total_order |
| --- | --- | --- |
| 101 | Meatlovers | 2 |
| 101 | Vegetarian | 1 |
| 102 | Meatlovers | 2 |
| 102 | Vegetarian | 1 |
| 103 | Meatlovers | 3 |
| 103 | Vegetarian | 1 |
| 104 | Meatlovers | 3 |
| 105 | Vegetarian | 1 |
6.  What was the maximum number of pizzas delivered in a single order?

```sql
SELECT 
	order_id,
	COUNT(pizza_id) AS max_count
FROM customer_orders_temp
GROUP BY order_id
ORDER BY COUNT(pizza_id) DESC LIMIT 1
```

| order_id | max_count |
| --- | --- |
| 4 | 3 |
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT
	customer_id,
	SUM(
    CASE WHEN exclusions <> ' ' OR extras <> ' ' THEN 1
    ELSE 0
    END) AS change,
	SUM(
    CASE WHEN exclusions = ' ' AND extras = ' ' THEN 1 
    ELSE 0
    END) AS no_change
FROM customer_orders_temp 
JOIN pizza_runner.runner_orders
 ON customer_orders_temp.order_id = runner_orders.order_id
WHERE cancellation not like '%Cancellation'
GROUP BY customer_id
```

| customer_id | change | no_change |
| --- | --- | --- |
| 101 | 2 | 0 |
| 104 | 1 | 1 |
| 105 | 1 | 0 |
| 102 | 0 | 1 |
8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT
	SUM(
		CASE WHEN exclusions != '' AND extras!='' THEN 1 
		ELSE 0
    END) aAS changed
FROM customer_orders_temp
JOIN runner_orders_temp
ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation
NOT LIKE '%Cancellation'
```

| changed |
| --- |
| 1 |
9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
extract(hour from order_time) as hours,
COUNT(pizza_id) as volume
FROM customer_orders_temp
GROUP BY hours
ORDER BY hours
```

| hours | volume |
| --- | --- |
| 11 | 1 |
| 13 | 3 |
| 18 | 3 |
| 19 | 1 |
| 21 | 3 |
| 23 | 3 |
10. What was the volume of orders for each day of the week?

```sql
SELECT 
TO_CHAR(order_time, 'Day') as days,
COUNT(pizza_id) as volume
FROM customer_orders_temp
GROUP BY days,DATE_PART('dow', order_time)
ORDER BY DATE_PART('dow', order_time)
```

| days | volume |
| --- | --- |
| Wednesday | 5 |
| Thursday | 3 |
| Friday | 1 |
| Saturday | 5 |

### **B. Runner and Customer Experience**

1. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
SELECT runner_id,
	ROUND(AVG(extract (epoch FROM (pickup_time::timestamp - order_time::timestamp)))::numeric/60,2) 
	AS Average_time_taken
FROM customer_orders_temp
JOIN runner_orders_temp
	ON runner_orders_temp.order_id = customer_orders_temp.order_id
GROUP BY runner_id
```

| runner_id | average_time_taken |
| --- | --- |
| 1 | 15.68 |
| 2 | 23.72 |
| 3 | 10.47 |
2. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH order_count_cte AS
(
	SELECT customer_orders_temp.order_id,
		COUNT(customer_orders_temp.order_id) AS pizzas_order_count,
		(extract (epoch FROM (pickup_time::timestamp - order_time::timestamp)))::numeric/60 as 			time_taken
	FROM runner_orders_temp
	JOIN customer_orders_temp 
		ON runner_orders_temp.order_id = customer_orders_temp.order_id
  WHERE cancellation not like '%Cancellation'
  GROUP BY customer_orders_temp.order_id, time_taken
  ORDER BY customer_orders_temp.order_id
)
  
SELECT pizzas_order_count , ROUND(AVG(time_taken),2) as avg_time_taken
FROM order_count_cte
GROUP BY pizzas_order_count
```

| pizzas_order_count | avg_time_taken |
| --- | --- |
| 3 | 29.28 |
| 2 | 18.38 |
| 1 | 12.36 |
3. What was the average distance travelled for each customer?

```sql
SELECT customer_orders_temp.customer_id,
ROUND(AVG(distance)::numeric,2)
FROM customer_orders_temp
JOIN runner_orders_temp
ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation not like '%Cancellation'
GROUP BY customer_orders_temp.customer_id
ORDER BY customer_orders_temp.customer_id
```

| customer_id | round |
| --- | --- |
| 101 | 20.00 |
| 102 | 16.73 |
| 103 | 23.40 |
| 104 | 10.00 |
| 105 | 25.00 |
4. What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT 
	MAX(duration) as max_duration,
	MIN(duration) min_duration,
	MAX(duration) - MIN(duration) as difference
FROM runner_orders_temp
```

| max_duration | min_duration | difference |
| --- | --- | --- |
| 40 | 10 | 30 |
5. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
SELECT runner_id,customer_id,
	ROUND(AVG((distance/(duration/60::float)))::numeric,2) AS speed
FROM runner_orders_temp
JOIN customer_orders_temp
	ON runner_orders_temp.order_id = customer_orders_temp.order_id
WHERE cancellation not like '%Cancellation'
GROUP BY runner_orders_temp.order_id,runner_id,customer_id
ORDER BY runner_id,customer_id
```

| 1 | 101 | 37.50 |
| --- | --- | --- |
| 1 | 101 | 44.44 |
| 1 | 102 | 40.20 |
| 1 | 104 | 60.00 |
| 2 | 102 | 93.60 |
| 2 | 103 | 35.10 |
| 2 | 105 | 60.00 |
| 3 | 104 | 40.00 |
6. What is the successful delivery percentage for each runner?

```sql
SELECT 
	runner_id,
	count(pickup_time) AS delivered_orders,
	count(*) AS total_orders,
	100*count(pickup_time)::float/count(*) AS percentage
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id
```

| runner_id | delivered_orders | total_orders | percentage |
| --- | --- | --- | --- |
| 1 | 4 | 4 | 100 |
| 2 | 3 | 4 | 75 |
| 3 | 1 | 2 | 50 |

**C. Ingredient Optimization**

Creating a temporary table to store the name and pizza topping id

```sql
CREATE TEMP TABLE toppings_temp AS
	SELECT 
		pizza_name ,
		CAST(trim(unnest(string_to_array(toppings, ',')))AS integer) AS topping_id
FROM pizza_runner.pizza_names AS p_name
JOIN
pizza_runner.pizza_recipes AS p_recipe
ON p_name.pizza_id = p_recipe.pizza_id;
```

| pizza_name | topping_id |
| --- | --- |
| Meatlovers | 1 |
| Meatlovers | 2 |
| Meatlovers | 3 |
| Meatlovers | 4 |
| Meatlovers | 5 |
| Meatlovers | 6 |
| Meatlovers | 8 |
| Meatlovers | 10 |
| Vegetarian | 4 |
| Vegetarian | 6 |
| Vegetarian | 7 |
| Vegetarian | 9 |
| Vegetarian | 11 |
| Vegetarian | 12 |
1.  What are the standard ingredients for each pizza?

```sql
SELECT 
	pizza_name,
	STRING_AGG(topping_name::TEXT, ', ') FROM toppings_temp
JOIN pizza_runner.pizza_toppings
	ON toppings_temp.topping_id = pizza_toppings.topping_id
GROUP BY pizza_name;
```

| pizza_name | string_agg |
| --- | --- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce |
2.  What was the most commonly added extra?

```sql
WITH extra_cte AS
	(SELECT 
		CAST(trim(unnest(string_to_array(extras, ',')))AS integer) AS extra_id,
	COUNT(*) AS total
	FROM customer_orders_temp 
	WHERE extras IS NOT NULL
	GROUP BY extra_id)

SELECT 
	topping_name, 
	total FROM pizza_runner.pizza_toppings
INNER JOIN extra_cte 
ON extra_cte.extra_id = pizza_toppings.topping_id
ORDER BY total DESC LIMIT 1
```

| topping_name | total |
| --- | --- |
| Bacon | 4 |
3. What was the most common exclusion?

```sql
WITH exclusion_cte AS
	(SELECT 
		CAST(trim(unnest(string_to_array(exclusions, ',')))AS integer) AS exclusion_id,
	COUNT(*) AS total
	FROM customer_orders_temp 
	WHERE exclusions IS NOT NULL
	GROUP BY exclusion_id)

SELECT 
	topping_name, 
	total FROM pizza_runner.pizza_toppings
INNER JOIN exclusion_cte 
ON exclusion_cte.exclusion_id = pizza_toppings.topping_id
ORDER BY total DESC LIMIT 1
```

| topping_name | total |
| --- | --- |
| Cheese | 4 |