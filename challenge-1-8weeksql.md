# Case Study #1: Danny's Diner

You can read the case study in here : https://8weeksqlchallenge.com/case-study-1/

## Solution

***

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT s.customer_id, SUM(price) AS sales_total
FROM dannys_diner.sales AS s
	JOIN dannys_diner.menu AS m
		ON s.product_id = m.product_id
GROUP BY customer_id;
````

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS count_customer
FROM dannys_diner.sales
GROUP BY customer_id;
````

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

***

### 3. What was the first item from the menu purchased by each customer?

````sql
WITH tempo AS
(
   SELECT customer_id, order_date, product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM dannys_diner.sales AS s
   JOIN dannys_diner.menu AS m
      ON s.product_id = m.product_id
)
SELECT customer_id, product_name
FROM tempo
WHERE rank = 1
GROUP BY customer_id, product_name;
````

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT TOP 1 (COUNT(s.product_id)) AS most_purchased, product_name
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m
   ON s.product_id = m.product_id
GROUP BY s.product_id, product_name
ORDER BY most_purchased DESC;
````

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |

***

### 5. Which item was the most popular for each customer?

````sql
WITH tempo AS (
SELECT customer_id, product_id, DENSE_RANK() OVER( PARTITION BY customer_id ORDER BY COUNT(product_id)) AS ranking
FROM dannys_diner.sales
GROUP BY customer_id, product_id
)

SELECT customer_id, product_id, ranking
FROM tempo
WHERE ranking = 1;
````

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

***

### 6. Which item was purchased first by the customer after they became a member?

````sql
WITH tempo AS(
SELECT s.customer_id, m.join_date, s.order_date, s.product_id, 
	DENSE_RANK() OVER( PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS urutan
FROM dannys_diner.sales AS s
	JOIN dannys_diner.members AS m
    	ON s.customer_id = m.customer_id
WHERE s.order_date >= m.join_date
) 
SELECT t.customer_id, t.product_id, m.product_name
FROM tempo AS t
	JOIN dannys_diner.menu as m
    	ON t.product_id = m.product_id
WHERE urutan = 1;
````

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH tempo AS(
SELECT s.customer_id, s.product_id, s.order_date,
DENSE_RANK() OVER ( PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank
FROM dannys_diner.sales AS s
	JOIN dannys_diner.members AS m
    	ON s.customer_id = m.customer_id
WHERE m.join_date > s.order_date
GROUP BY s.customer_id, s.product_id, s.order_date
)

SELECT customer_id, product_id, rank
FROM tempo
WHERE rank=1;
````

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-01 |  sushi        |
| A           | 2021-01-01 |  curry        |
| B           | 2021-01-04 |  sushi        |

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT s.customer_id, s.product_id, n.product_name, COUNT(s.product_id) jumlah, SUM(price) total_harga
FROM dannys_diner.sales AS s
	JOIN dannys_diner.members AS m
    	ON s.customer_id = m.customer_id
    JOIN dannys_diner.menu AS n
    	ON s.product_id = n.product_id
WHERE s.order_date < m.join_date
GROUP BY s.customer_id, s.product_id, n.product_name;
````

#### Answer:
| customer_id | unique_menu_item | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 2 |  40       |

***

### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

````sql
WITH tempo AS(
SELECT *,
CASE 
	WHEN s.product_id = 1 THEN price*20
    WHEN s.product_id = 2 THEN price*10
    WHEN s.product_id = 3 THEN price*10
END point
FROM dannys_diner.sales AS s
	JOIN dannys_diner.menu AS m
    	ON s.product_id = m.product_id
)
SELECT customer_id, SUM(point) total_point
FROM tempo
GROUP BY customer_id;
````

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
WITH dates_cte AS 
(
   SELECT *, 
      DATEADD(DAY, 6, join_date) AS valid_date, 
      EOMONTH('2021-01-31') AS last_date
   FROM members AS m
)

SELECT d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price,
   SUM(CASE
      WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
      WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price
      ELSE 10 * m.price
      END) AS points
FROM dates_cte AS d
JOIN sales AS s
   ON d.customer_id = s.customer_id
JOIN menu AS m
   ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price
````

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

***
