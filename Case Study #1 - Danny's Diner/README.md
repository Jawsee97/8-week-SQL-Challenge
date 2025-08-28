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
