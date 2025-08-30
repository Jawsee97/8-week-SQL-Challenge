# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="500">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to analyze customer data to understand visiting patterns, spending habits, and favorite items. These insights will guide loyalty program decisions and provide simple datasets his team can review without using SQL.

***

## Entity Relationship Diagram

<img width="717" height="348" alt="Screenshot 2025-08-26 at 12 50 34 AM" src="https://github.com/user-attachments/assets/e3e33bd0-4d59-4251-a2a3-cb2b8722714a" />

***

## Question and Solution

The Database, along with the queries that will be executed for this project can be found here -> [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138).


**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
	sales.customer_id,
    SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
````
#### Steps:
- Used **JOIN** to merge `dannys_diner.sales` and `dannys_diner.menu` tables as `sales.customer_id` and `menu.price` are from both tables.
- Used **SUM** to calculate the total sales contributed by each customer.
- Grouped the aggregated results by `sales.customer_id`. 

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

From this Table, we can gather that:
- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT 
	customer_id,
    COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
````

#### Steps:
- To determine the unique number of visits for each customer, utilized **COUNT(DISTINCT `order_date`)**.
- Used the **DISTINCT** keyword to calculate the visit count to avoid duplicate counting of days. For example, if Customer A visited the restaurant twice on '2021–01–07', counting without **DISTINCT** would result in 2 days instead of the accurate count of 1 day.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

From this Table, we can gather that:
- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH ranked_sales AS (
	SELECT 
  		sales.customer_id,
  		sales.order_date,
  		menu.product_name,
  		ROW_NUMBER() OVER (
          PARTITION BY sales.customer_id
          ORDER BY sales.order_date ASC, sales.product_id ASC
        ) AS rn
  	FROM sales
  	JOIN menu
  		ON sales.product_id = menu.product_id
  )
  SELECT 
  	customer_id,
    order_date,
    product_name AS first_item
    FROM ranked_sales
    WHERE rn = 1;
````
#### Steps:
- Used **ROW_NUMBER()** to assign a rank to each purchase per customer ordered by the earliest `order_date`
- On the off chance that items were bought on the same earliest date, the **ORDER BY sales.product_id** would ensure consistent picking.
- We lastly filter this query with 'WHERE rn = 1' to keep the first item per customer.
  
| customer_id | product_name | 
| ----------- | ----------- |
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

From this Table, we can gather that:
- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT
	menu.product_name,
    COUNT(sales.product_id) AS total_purchases
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY total_purchases DESC
LIMIT 1;
````

#### Steps:
- Used a **COUNT** aggregation on the `product_id` column and **ORDER BY** the result in descending order using `total_purchases` field.
- Used the **LIMIT** 1 clause to filter and retrieve the top-selling item.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


From this Table, we can gather that:
- Ramen is the most purchased item on the menu !

***

**5. Which item was the most popular for each customer?**

````sql
WITH customer_item_counts AS (
  SELECT 
  	s.customer_id,
  	m.product_name,
  	COUNT(*) AS purchase_count
  FROM sales s
  JOIN menu m
  	ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
),
ranked_items AS ( 
  SELECT 
  	customer_id,
  	product_name,
  	purchase_count,
  	RANK() OVER (
      PARTITION BY customer_id
      ORDER BY purchase_count DESC
    ) AS rnk
   FROM customer_item_counts
  )
 SELECT 
 	customer_id,
    product_name AS most_popular_itme,
    purchase_count
 FROM ranked_items
 WHERE rnk = 1
 ORDER BY customer_id;
````

*Each user may have more than 1 favourite item.*

#### Steps:
- Used a **COUNT** aggregation to see how many times each customer ordered each product `customer_item_counts`.
- Using `purchase_count` we ranked items per customer
- Then used the **RANK** function in the case of a tie, we return all top items.
- Finally, used a filter with `WHERE rnk = 1`.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

From this Table, we can gather that:
- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu.

***

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH member_sales AS (
  SELECT
  	s.customer_id,
  	s.order_date,
  	m.product_name,
  	ROW_NUMBER() OVER (
      PARTITION BY s.customer_id
      ORDER BY s.order_date ASC, s.product_id ASC
      ) AS rn
  	FROM sales s
  	JOIN menu m
  	ON s.product_id = m.product_id
  	JOIN members mem
  	ON s.customer_id = mem.customer_id
  	WHERE s.order_date >= mem.join_date
  )
SELECT customer_id, order_date, product_name AS first_item_after_join
FROM member_sales
WHERE rn = 1
ORDER BY customer_id;
```

#### Steps:
- Created a CTE named `member_sales` then selected the appropriate columns and calculated the row number using the **ROW_NUMBER()** function. The **PARTITION BY** clause divides the data by `s.customer_id` and the **ORDER BY** clause orders the rows within each `s.order_date` and `s.product_id` ASC.
- Then joined the tables `members` and `sales` on `customer_id` column. Additionally, applied a condition to include sales that occurred *On or After* the member's `join_date` (`s.order_date >= mem.join_date`).
- Then used the **WHERE** clause in the **Outer Query** to filter and retrieve only the rows where the rn column = 1, representing the first row within each `customer_id` partition.
- Lastly, ordered result by `customer_id`.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | curry        |
| B           | sushi        |

From this Table, we can gather that:
- Customer A's first order as a member is curry.
- Customer B's first order as a member is sushi.

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH pre_member_sales AS ( 
  SELECT
  	s.customer_id,
  	s.order_date,
  	m.product_name,
  	ROW_NUMBER() OVER (
      PARTITION BY s.customer_id
      ORDER BY s.order_date DESC, s.product_id DESC
      ) AS rn
  FROM sales s
  JOIN menu m 
  	ON s.product_id = m.product_id
  JOIN members mem
  	ON s.customer_id = mem.customer_id
  WHERE s.order_date < mem.join_date
  )
  SELECT
  	customer_id,
    order_date,
    product_name AS last_item_before_join
  FROM pre_member_sales
  WHERE rn = 1
  ORDER BY customer_id;
````

#### Steps:
- Created a CTE called `pre_member_sales`. 
- Select the appropriate columns and calculate the rank using the **ROW_NUMBER()** window function. The rank is determined based on the order dates of the sales along with the product ID in descending order.
- Join `dannys_diner.sales` table with `dannys_diner.menu` table and the `dannys_diner.members` table so we can filter by membership join date.
- Then used the **WHERE** clause to keep sales that only happened before the customer became a member.
- Then filtered the result set to include only the rows where the rank = 1, representing the earliest purchase made by each customer before they became a member.
- Finally, sorted the result by `customer_id`.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | curry        |
| B           | sushi        |

From this Table, we can gather that:
- Customers A last item purchased pre-membership was curry
- Customers B last item purchased pre-membership was sushi

***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT 
	mem.customer_id,
    COUNT(s.product_id) AS total_items,
    SUM(m.price) AS total_amount
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
JOIN members mem
	ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date
GROUP BY mem.customer_id
ORDER BY mem.customer_id;
```

#### Steps:
- Selected the columns `mem.customer_id` and then used the **COUNT** aggregation to count `sales.product_id` as total_items for each customer. Then used the  **SUM** aggregation to sum the `m.price` as total_sales.
- Joined the `sales`,`menu`, and `members` table, then used the **WHERE** clause to filter the `s.order_date` and `mem.join_date` to keep only the rows where the purchase date is before the customer's join date.
- Grouped the results by `mem.customer_id`.
- Ordered the result by `mem.customer_id`.

#### Answer:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

From this Table, we can gather that:
Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
SELECT s.customer_id,
	SUM(
      CASE
      	WHEN m.product_name = 'sushi' THEN m.price * 10 * 2
      	ELSE m.price * 10
      END
     ) AS total_points
FROM sales s
JOIN menu m 
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Steps:
- Used a `CASE` expression to calculate points, if product name = sushi, then apply the 2x points multiplier, otherwise, multiply price by 10.
- Then joined the `sales` and `menu` table
- Finally Grouped and Ordered rows by `s.customer_id`

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

From this Table, we can gather that:
- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**


