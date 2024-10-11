# Case Study #1: Danny's Diner 
![1](https://github.com/user-attachments/assets/73cfd6eb-a67b-4c07-9b34-a92297d085c3)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***

## Entity Relationship Diagram

<img width="695" alt="Screenshot 2024-10-10 at 3 24 13 PM" src="https://github.com/user-attachments/assets/50cadd7f-3751-426b-b203-959692ad418f">


***

## Question and Solution

Please join me in executing the queries using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138). It would be great to work together on the questions!

Additionally, I have also published this case study on [Medium](https://katiehuangx.medium.com/8-week-sql-challenge-case-study-week-1-dannys-diner-2ba026c897ab?source=friends_link&sk=ed355696f5a70ff8b3d5a1b905e5dabe).

If you have any questions, reach out to me on [LinkedIn](https://www.linkedin.com/in/katiehuangx/).

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
  customer_id,
  SUM(price) AS total_price
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY customer_id;
````

#### Steps:
- JOIN: Combines the sales and menu tables using product_id. This is needed because sales only contains product_id but not the price.
- SUM(price): Sums the prices of all products a customer purchased.
- GROUP BY customer_id: Groups the purchases by customer so we get the total amount for each customer separately.

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT 
	customer_id,
	COUNT(DISTINCT order_date) AS visit_count
FROM sales
GROUP BY customer_id;
````

How:
	- COUNT(DISTINCT order_date): Counts unique visit days by excluding duplicate dates within each customer’s records.
	- GROUP BY customer_id: Groups the results by customer so each customer has their own visit count.
Outcome: Total number of distinct days each customer visited the restaurant.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

***

**3. What was the first item from the menu purchased by each customer?**

````sql
SELECT
    s.customer_id,
    s.order_date,
    m.product_name
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN (
    SELECT customer_id, MIN(order_date) AS first_order_date
    FROM sales
    GROUP BY customer_id
) AS first_order ON s.customer_id = first_order.customer_id
                 AND s.order_date = first_order.first_order_date;
````
How:
	- Subquery:
	- MIN(order_date): Finds the earliest (first) order date for each customer.
	- GROUP BY customer_id: Ensures we get the first order date for each customer.
	- Main Query:
	  - JOIN: Combines sales with menu to get product names and then joins with the subquery to get the order corresponding to the earliest order date.
	- Outcome: First item purchased by each customer along with the purchase date.

#### Answer:
| customer_id | product_name | 
| ----------- | -----------  |
| A           | sushi        | 
| A           | curry        | 
| B           | curry        | 
| C           | ramen        |

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT 
    product_name,
    COUNT(s.product_id) AS purchase_frequency
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY product_name
ORDER BY purchase_frequency DESC
Limit 1;
````

How:
	- COUNT(s.product_id): Counts how many times each item (product) was purchased.
	- GROUP BY product_name: Groups results by product to calculate purchase counts per item.
	- ORDER BY purchase_frequency DESC: Orders the items by purchase count in descending order to find the most purchased one.
	- LIMIT 1: Limits the result to the top one most purchased item.
Outcome: The most purchased item and how many times it was bought.

#### Answer:
| product_name | purchase_frequency | 
| ----------- | ----------- |
| ramen       | 8 |

***

**5. Which item was the most popular for each customer?**

````sql
WITH most_popular AS (
SELECT 
	customer_id,
    product_name,
    COUNT('product_name') AS popular_item,
DENSE_RANK() OVER(
	PARTITION BY s.customer_id
    ORDER BY COUNT(s.customer_id) DESC) AS popularity
FROM sales s
JOIN menu m 
ON s.product_id = m.product_id
GROUP BY customer_id, product_name
)
SELECT 
    customer_id,
    product_name,
    popular_item
FROM most_popular
WHERE popularity = 1;
````

How:
	- COUNT(product_name): Counts how many times each item was purchased by each customer.
	- DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(product_name) DESC): Ranks items by purchase frequency for each customer. DENSE_RANK assigns ranks starting from 1, with no gaps in ranking.
	- WITH: Creates a temporary result set called most_popular to store the ranking information.
	- WHERE popularity = 1: Filters to get only the most popular item (ranked 1).
Outcome: The most popular (frequently purchased) item for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH member_join AS (
SELECT 
	mb.customer_id,
    s.product_id,
	ROW_NUMBER() OVER(
    PARTITION BY mb.customer_id
		ORDER BY s.order_date) AS row_num
FROM members mb
JOIN sales s ON
	s.customer_id = mb.customer_id
AND s.order_date > mb.join_date
)
SELECT 
	customer_id,
    product_name
FROM member_join mj
JOIN menu m
ON mj.product_id = m.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```

How:
	- ROW_NUMBER() OVER (PARTITION BY mb.customer_id ORDER BY s.order_date): Assigns a sequential number to each purchase after the customer became a member.
	- WHERE row_num = 1: Filters to get the first purchase after membership.
	- Main Query: Joins the result with menu to get the product name of the first purchase.
Outcome: The first item purchased by each customer after joining the program.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH prior_purchase AS (
SELECT 
	mb.customer_id,
    s.product_id,
	ROW_NUMBER() OVER(
    PARTITION BY mb.customer_id
		ORDER BY s.order_date DESC) AS tier
FROM members mb
JOIN sales s ON
	s.customer_id = mb.customer_id
AND s.order_date < mb.join_date
)
SELECT 
	customer_id,
    product_name
FROM prior_purchase pp
JOIN menu m
ON pp.product_id = m.product_id
WHERE tier = 1
ORDER BY customer_id ASC;
````

How:
	- ROW_NUMBER() OVER (PARTITION BY mb.customer_id ORDER BY s.order_date DESC): Assigns a sequential number to purchases in reverse chronological order (last purchase first).
	- WHERE tier = 1: Filters to get the last purchase before membership.
	- Outcome: The last item purchased by each customer before becoming a member.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |

***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT 
	s.customer_id,
    COUNT(s.product_id) AS total_item,
    SUM(price) AS total_price
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
JOIN members mb 
ON s.customer_id = mb.customer_id
AND s.order_date < mb.join_date
GROUP BY customer_id
ORDER BY customer_id;
```

How:
	- COUNT(s.product_id): Counts the total number of items purchased before membership.
	- SUM(price): Sums the total amount of money spent before membership.
	- GROUP BY customer_id: Groups results by customer for individual totals.
Outcome: Total items and amount spent before becoming a member for each customer.

#### Answer:
| customer_id | total_items | total_price |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
WITH point_gain AS (
SELECT 
	m.product_id,
	CASE
	WHEN product_id = 1 THEN m.price * 20
    ELSE m.price * 10 END AS points
FROM menu m
)

SELECT 
	s.customer_id,
    SUM(pg.points) AS total_point
FROM sales s
JOIN point_gain pg
ON s.product_id = pg.product_id
GROUP BY s.customer_id;
```

How:
	- CASE: Multiplies the price of sushi by 20 points and other items by 10 points.
	- SUM(pg.points): Adds up all points earned by each customer.
	- GROUP BY s.customer_id: Groups results by customer to calculate total points for each.
Outcome: Total points earned by each customer.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```sql
WITH january_sales AS (
    SELECT 
        s.customer_id,
        s.order_date,
        m.product_name,
        m.price,
        mem.join_date,
        CASE 
            WHEN s.order_date BETWEEN mem.join_date AND DATE_ADD(mem.join_date, INTERVAL 6 DAY) THEN m.price * 2 * 10
            ELSE m.price * 10
        END AS points
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id
    WHERE s.customer_id IN ('A', 'B')
      AND s.order_date BETWEEN '2021-01-01' AND '2021-01-31'
)
SELECT 
    customer_id,
    SUM(points) AS total_points
FROM january_sales
GROUP BY customer_id;
```

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1270 |
| B           | 720 |


***
