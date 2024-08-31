# Data Cleaning

The exclusions and extras columns in the customer_order table consists of ‘null’ and missing/blank spaces.

Removing the null values and replacing them with  blank space‘’ by creating a temporary table.

```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ''
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras LIKE 'null' THEN ''
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;
```

The runner_orders table also consists of null and missing values.

- In `pickup_time` column, remove nulls and replace with blank space ' '.
- In `distance` column, remove "km" and nulls and replace with blank space ' '.
- In `duration` column, remove "minutes", "minute" and nulls and replace with blank space ' '.
- In `cancellation` column, remove NULL and null and and replace with blank space ' '.
- Change the columns into appropriate data type

```sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT 
  order_id, 
  runner_id, 
  CAST(
  	CASE WHEN pickup_time LIKE 'null' THEN NULL ELSE pickup_time END 
      AS Timestamp) AS pickup_time,
  CAST(
  	CASE WHEN distance LIKE 'null' THEN NULL
        WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
        ELSE distance END
    AS FLOAT) AS distance,
  CAST(
  	CASE WHEN duration LIKE 'null' THEN NULL
        WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
        ELSE duration END
    AS INT) AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ''
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;
```