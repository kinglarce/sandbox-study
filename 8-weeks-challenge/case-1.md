# Case 1

{% embed url="https://8weeksqlchallenge.com/case-study-1/" %}

{% embed url="https://dbdiagram.io/d/608d07e4b29a09603d12edbd/?utm_medium=bottom_open&utm_source=dbdiagram_embed" %}

Each of the following case study questions can be answered using a single SQL statement:

What is the total amount each customer spent at the restaurant?

```sql
SELECT 
  sales.customer_id, 
  SUM(price) 
FROM 
  dannys_diner.sales as sales 
  JOIN dannys_diner.menu as menu ON menu.product_id = sales.product_id 
GROUP BY 
  sales.customer_id;
```

How many days has each customer visited the restaurant?

```sql
SELECT 
  sales.customer_id, 
  COUNT(
    DISTINCT(sales.order_date)
  ) as visit_count
FROM 
  dannys_diner.sales as sales 
GROUP BY 
  sales.customer_id;
```

What was the first item from the menu purchased by each customer?

```sql
WITH ranked AS (
  SELECT 
    m.product_name, 
    s.customer_id, 
    s.order_date, 
    DENSE_RANK() OVER(
      PARTITION BY s.customer_id 
      ORDER BY 
        s.order_date, 
        s.product_id
    ) rank 
  FROM 
    dannys_diner.sales as s 
    JOIN dannys_diner.menu as m ON m.product_id = s.product_id
)
-- SELECT * FROM ranked # for testing the CTE(common table expression)
SELECT 
  customer_id, 
  product_name 
FROM 
  ranked 
WHERE 
  rank = 1 
GROUP BY 
  customer_id, 
  product_name

```

What is the most purchased item on the menu and how many times was it purchased by all customers?

<pre class="language-sql"><code class="lang-sql"><strong>-- Approach 1
</strong>SELECT 
  COUNT(s.product_id) as most_purchased, 
  m.product_name 
FROM 
  dannys_diner.sales as s 
  JOIN dannys_diner.menu as m ON m.product_id = s.product_id 
GROUP BY 
  s.product_id, 
  m.product_name 
ORDER BY 
  most_purchased DESC 
LIMIT 
  1;

-- Approach 2
WITH most_purchased AS (
  SELECT 
    m.product_name, 
    COUNT(s.product_id) as total_purchase, 
    ROW_NUMBER() OVER() as rank 
  FROM 
    dannys_diner.sales as s 
    JOIN dannys_diner.menu as m ON m.product_id = s.product_id 
  GROUP BY 
    m.product_name
) 
-- SELECT * FROM most_purchased;
SELECT 
  product_name, 
  total_purchase 
FROM 
  most_purchased 
WHERE 
  rank = 1</code></pre>

Which item was the most popular for each customer?

```sql
WITH most_popular AS (
  SELECT 
    m.product_name, 
    s.customer_id, 
    COUNT(s.product_id) as order_count, 
    DENSE_RANK() OVER(
      PARTITION BY s.customer_id 
      ORDER BY 
        COUNT(s.product_id) DESC
    ) rank 
  FROM 
    dannys_diner.sales as s 
    JOIN dannys_diner.menu as m ON m.product_id = s.product_id 
  GROUP BY 
    s.customer_id, 
    m.product_name
) 
-- SELECT * FROM most_popular
SELECT 
  product_name, 
  customer_id, 
  order_count 
FROM 
  most_popular AS mp 
WHERE 
  rank = 1

```

Which item was purchased first by the customer after they became a member?

```sql
WITH purchased_after_membership AS (
  SELECT 
    mb.customer_id, 
    s.product_id, 
    mb.join_date, 
    s.order_date, 
    ROW_NUMBER() OVER(PARTITION BY s.customer_id) rank 
  FROM 
    dannys_diner.members AS mb 
    JOIN dannys_diner.sales AS s ON s.customer_id = mb.customer_id 
  WHERE 
    s.order_date >= mb.join_date
) 
SELECT 
  customer_id, 
  product_name, 
  order_date 
FROM 
  purchased_after_membership as pm 
  JOIN dannys_diner.menu AS m ON m.product_id = pm.product_id 
WHERE 
  rank = 1

```

Which item was purchased just before the customer became a member?

<pre><code><strong>// Some code</strong></code></pre>

What is the total items and amount spent for each member before they became a member?

<pre><code><strong>// Some code</strong></code></pre>

If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

<pre><code><strong>// Some code</strong></code></pre>

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

<pre><code><strong>// Some code</strong></code></pre>
