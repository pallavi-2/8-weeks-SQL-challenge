# Solutions

Solutions

1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
  sales.customer_id, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

| **customer_id** | **total_sales** |
| --- | --- |
| A | 76 |
| B | 74 |
| C | 36 |
1. How many days has each customer visited the restaurant?

```sql
SELECT
  customer_id, 
  COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
```

| **customer_id** | **visit_count** |
| --- | --- |
| A | 4 |
| B | 6 |
| C | 2 |
1. What was the first item FROM the menu purchased by each customer?

```sql
WITH ordered_sales AS
(
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    Dense_rank() over (PARTITION BY sales.customer_id
                       ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales 
  JOIN dannys_diner.menu 
     ON sales.product_id = menu.product_id
 )

SELECT customer_id, 
 product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id,product_name
```

| **customer_id** | **product_name** |
| --- | --- |
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |
1. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
  menu.product_name,
  count(menu.product_id)
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name 
ORDER BY count(menu.product_id) DESC LIMIT 1
```

| **product_name** | **count** |
| --- | --- |
| ramen | $8.00 |

```sql
SELECT sales.customer_id,
  menu.product_name,
  count(menu.product_id)
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
WHERE menu.product_name =
(
  SELECT
    menu.product_name
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  GROUP BY menu.product_name 
  ORDER BY count(menu.product_id) DESC LIMIT 1
  )
GROUP BY sales.customer_id, menu.product_name
ORDER BY sales.customer_id
```

| **customer_id** | **product_name** | **count** |
| --- | --- | --- |
| A | ramen | 3 |
| B | ramen | 2 |
| C | ramen | 3 |
1. Which item was the most popular for each customer?

```sql
WITH most_popular AS
(
  SELECT
    sales.customer_id, 
    menu.product_name, 
    COUNT(sales.product_id) AS order_count,
    DENSE_RANK() OVER (
        PARTITION BY sales.customer_id 
        ORDER BY COUNT(sales.product_id) DESC) AS rank
   FROM dannys_diner.menu
   INNER JOIN dannys_diner.sales
     ON menu.product_id = sales.product_id
   GROUP BY sales.customer_id, menu.product_name
)
  
SELECT customer_id,
  product_name,
  order_count
FROM most_popular
WHERE rank = 1
```

| customer_id | product_name | order_count |
| --- | --- | --- |
| A | ramen | 3 |
| B | ramen | 2 |
| B | curry | 2 |
| B | sushi | 2 |
| C | ramen | 3 |
1.  Which item was purchased first by the customer after they became a member?

```sql
WITH first_purchased AS 
(
  SELECT members.customer_id,
    sales.product_id,
    Row_number() over(
        PARTITION BY members.customer_id
        ORDER BY sales.order_date) AS rank
  FROM dannys_diner.members 
  JOIN dannys_diner.sales 
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.JOIN_date
    )

SELECT first_purchased.customer_id,
  menu.product_name
FROM first_purchased 
JOIN dannys_diner.menu
  ON first_purchased.product_id=menu.product_id 
WHERE first_purchased.rank = 1
ORDER BY customer_id
```

| customer_id | product_name |
| --- | --- |
| A | ramen |
| B | sushi |
1. Which item was purchased just before the customer became a member?

```sql
WITH purchased_just_before AS 
(
  SELECT members.customer_id,
    sales.product_id,
    Row_Number() over(
       PARTITION BY members.customer_id
       ORDER BY sales.order_date DESC
    ) AS rank
  FROM dannys_diner.members 
  JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date < members.JOIN_date 
)
  
SELECT
  purchased_just_before.customer_id,
  product_name
FROM purchased_just_before
JOIN dannys_diner.menu
  ON purchased_just_before.product_id = menu.product_id
WHERE rank = 1
```

| customer_id | product_name |
| --- | --- |
| B | sushi |
| A | sushi |
1. What is the total items and amount spent for each member before they became a member?

```sql
SELECT sales.customer_id,
  count(sales.product_id),
  SUM(menu.price)
FROM dannys_diner.sales
JOIN dannys_diner.members 
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.JOIN_date
JOIN dannys_diner.menu
   ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
```

| customer_id | count | sum |
| --- | --- | --- |
| B | 3 | 40 |
| A | 2 | 25 |
1. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT
  customer_id,
SUM(CASE
	    WHEN product_name = 'sushi' THEN menu.price*2*10
      ELSE menu.price * 10
    END) AS points    
FROM dannys_diner.sales
JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id
```

| customer_id | points |
| --- | --- |
| A | 860 |
| B | 940 |
| C | 360 |
1. In the first week after a customer JOINs the program (including their JOIN date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
WITH dates AS 
(
  SELECT
    customer_id, 
    JOIN_date, 
    JOIN_date + 6 AS week_date, 
    DATE('2021-01-31T00:00:00.000Z') AS last_date
  FROM dannys_diner.members
)

SELECT
  sales.customer_id, 
  SUM(CASE
        WHEN menu.product_name = 'sushi' THEN menu.price*2 * 10
        WHEN sales.order_date BETWEEN dates.JOIN_date AND dates.week_date THEN menu.price*2*10  
        ELSE menu.price*10
      END) AS points
FROM dannys_diner.sales
INNER JOIN dates
  ON sales.customer_id = dates.customer_id
  AND dates.JOIN_date <= sales.order_date
  AND sales.order_date <= dates.last_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id
```

| customer_id | points |
| --- | --- |
| A | 1020 |
| B | 320 |

### Bonus Questions

Join all tables

```sql
SELECT 
  sales.customer_id, 
  sales.order_date,  
  menu.product_name, 
  menu.price,
  CASE
    WHEN members.join_date > sales.order_date THEN 'N'
    WHEN members.join_date <= sales.order_date THEN 'Y'
    ELSE 'N' 
  END AS member_status
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
ORDER BY members.customer_id, sales.order_date
```

| customer_id | order_date | product_name | price | member_status |
| --- | --- | --- | --- | --- |
| A | 2021-01-01T00:00:00.000Z | sushi | 10 | N |
| A | 2021-01-01T00:00:00.000Z | curry | 15 | N |
| A | 2021-01-07T00:00:00.000Z | curry | 15 | Y |
| A | 2021-01-10T00:00:00.000Z | ramen | 12 | Y |
| A | 2021-01-11T00:00:00.000Z | ramen | 12 | Y |
| A | 2021-01-11T00:00:00.000Z | ramen | 12 | Y |
| B | 2021-01-01T00:00:00.000Z | curry | 15 | N |
| B | 2021-01-02T00:00:00.000Z | curry | 15 | N |
| B | 2021-01-04T00:00:00.000Z | sushi | 10 | N |
| B | 2021-01-11T00:00:00.000Z | sushi | 10 | Y |
| B | 2021-01-16T00:00:00.000Z | ramen | 12 | Y |
| B | 2021-02-01T00:00:00.000Z | ramen | 12 | Y |
| C | 2021-01-01T00:00:00.000Z | ramen | 12 | N |
| C | 2021-01-01T00:00:00.000Z | ramen | 12 | N |
| C | 2021-01-07T00:00:00.000Z | ramen | 12 | N |

Rank all the things 

```sql
WITH Ranking AS
(
  SELECT 
    sales.customer_id, 
    sales.order_date,  
    menu.product_name, 
    menu.price,
    CASE
      WHEN members.join_date > sales.order_date THEN 'N'
      WHEN members.join_date <= sales.order_date THEN 'Y'
      ELSE 'N' 
    END AS member_status

 FROM dannys_diner.sales
 FULL JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
 FULL JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)
    
SELECT *,
  CASE
	  WHEN member_status = 'N' THEN NULL
    ELSE RANK() OVER (
      PARTITION BY customer_id,member_status
      ORDER BY order_date
      )
  END as ranking
FROM Ranking 
```

| customer_id | order_date | product_name | price | member_status | ranking |
| --- | --- | --- | --- | --- | --- |
| A | 2021-01-01T00:00:00.000Z | curry | 15 | N | null |
| A | 2021-01-01T00:00:00.000Z | sushi | 10 | N | null |
| A | 2021-01-07T00:00:00.000Z | curry | 15 | Y | 1 |
| A | 2021-01-10T00:00:00.000Z | ramen | 12 | Y | 2 |
| A | 2021-01-11T00:00:00.000Z | ramen | 12 | Y | 3 |
| A | 2021-01-11T00:00:00.000Z | ramen | 12 | Y | 3 |
| B | 2021-01-01T00:00:00.000Z | curry | 15 | N | null |
| B | 2021-01-02T00:00:00.000Z | curry | 15 | N | null |
| B | 2021-01-04T00:00:00.000Z | sushi | 10 | N | null |
| B | 2021-01-11T00:00:00.000Z | sushi | 10 | Y | 1 |
| B | 2021-01-16T00:00:00.000Z | ramen | 12 | Y | 2 |
| B | 2021-02-01T00:00:00.000Z | ramen | 12 | Y | 3 |
| C | 2021-01-01T00:00:00.000Z | ramen | 12 | N | null |
| C | 2021-01-01T00:00:00.000Z | ramen | 12 | N | null |
| C | 2021-01-07T00:00:00.000Z | ramen | 12 | N | null |