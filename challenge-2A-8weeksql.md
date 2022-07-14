# 🍕 Case Study #2 - Pizza Runner

You can read the study case in here : https://8weeksqlchallenge.com/case-study-2/

## Solution - 2A. Pizza Metrics

### 1. How many pizzas were ordered?

````sql
SELECT COUNT(order_id) total_ordered 
FROM pizza.customer_orders;
````

**Answer:**

![Screenshot 2022-07-14 193719](https://user-images.githubusercontent.com/67949585/178985216-837cb15b-3071-48b0-ba38-1d40fb0a6faf.png)

***


### 2. How many unique customer orders were made?

````sql
SELECT COUNT(DISTINCT(customer_id)) unique_customer
FROM pizza.customer_orders;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178985802-adf77ad2-f443-4652-82fe-40583656024d.png)

***

### 3. How many successful orders were delivered by each runner?

````sql
SELECT runner_id, COUNT(distance) success 
FROM pizza.runner_orders
	WHERE distance != 0
GROUP BY runner_id;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178986514-9ee695b3-5c4f-43df-b72c-f5f8a7a58e02.png)

***

### 4. How many of each type of pizza was delivered?

````sql
WITH temp AS (
SELECT * FROM pizza.customer_orders AS c
	JOIN pizza.runner_orders AS r
    	ON c.order_id = r.order_id
)
SELECT pizza_id, count(distance) delivered
FROM temp
	WHERE distance !=0
GROUP BY pizza_id;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178986819-9831a72d-2b3e-413e-8943-93f1868867a4.png)

***

### 5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
SELECT customer_id, pizza_name, count(c.pizza_id) count_pizza
FROM pizza.customer_orders AS c
	JOIN pizza.pizza_names AS p
    	ON c.pizza_id = p.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY customer_id;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178987068-054e96e8-1a6c-4568-9e21-5adcf4934431.png)

***

### 6. What was the maximum number of pizzas delivered in a single order?

````sql
SELECT order_id, COUNT(pizza_id) max_number_ordered
FROM pizza.customer_orders
GROUP BY order_id
ORDER BY max_number_ordered DESC
LIMIT 1;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178987609-93817cbd-c7f7-45dc-b107-e6a3af4f04e8.png)

***

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT o.customer_id,
SUM(CASE 
	WHEN o.exclusions > 0 AND o.extras > 0 THEN 0
    ELSE 1
END) AS no_changes,
SUM(CASE
    WHEN o.exclusions > 0 OR o.extras > 0 THEN 1
    ELSE 0
END) AS at_least_one_change
FROM pizza.customer_orders AS o
JOIN pizza.runner_orders AS r
	ON o.order_id = r.order_id
GROUP BY o.customer_id, o.order_id;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178990668-c5cceed3-2871-4bc6-a904-44b4cfef6719.png)

***

### 8. How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT SUM(pizza_id) number_pizza
FROM pizza.customer_orders AS o
JOIN pizza.runner_orders AS r
	ON o.order_id = r.order_id
WHERE o.exclusions IS NOT NULL AND o.extras IS NOT NULL; 
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178991154-7c75187f-7941-46d9-821c-9a80b5f21ecc.png)

***

### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT 
  DATE_PART('hour', order_time) AS hour_of_day, 
  COUNT(order_id) AS pizza_count
FROM pizza.customer_orders
GROUP BY DATE_PART('hour', order_time);
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178991463-67e12ed7-f25d-4973-bc38-85b9490dda44.png)

- Highest volume of pizza ordered is at 13 (1:00 pm), 18 (6:00 pm), 21 (9:00 pm), and 23 (11:00 pm).
- Lowest volume of pizza ordered is at 11 (11:00 am) and 19 (7:00 pm).

### 10. What was the volume of orders for each day of the week?

````sql
SELECT 
  EXTRACT (DOW FROM order_time) as HARIAN,
  COUNT(order_id) AS pizza_count
FROM pizza.customer_orders
GROUP BY harian;
````

**Answer:**

![image](https://user-images.githubusercontent.com/67949585/178991804-819f0411-5b2c-4f73-a94a-037980beb989.png)

- 1 = Sunday, 7 = Saturday

***Lets continue to the 2B. Runner and Customer Experience! case***

