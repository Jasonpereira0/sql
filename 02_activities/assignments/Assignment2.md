# Jason Pereira
# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:
* Submission Due Date: `December 22, 2024`
* Weight: 70% of total grade
* The branch name for your repo should be: `assignment-two`
* What to submit for this assignment:
    * This markdown (Assignment2.md) with written responses in Section 1
    * Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
    * One .sql file 
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [X] Create a branch called `assignment-two`.
- [X] Ensure that the repository is public.
- [X] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [X] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack at `#cohort-5-help`. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

***

## Section 1:
You can start this section following *session 1*, but you may want to wait until you feel comfortable wtih basic SQL query writing. 

Steps to complete this part of the assignment:
- Design a logical data model
- Duplicate the logical data model and add another table to it following the instructions
- Write, within this markdown file, an answer to Prompt 3


###  Design a Logical Model

#### Prompt 1
Design a logical model for a small bookstore. 📚

At the minimum it should have employee, order, sales, customer, and book entities (tables). Determine sensible column and table design based on what you know about these concepts. Keep it simple, but work out sensible relationships to keep tables reasonably sized. 

Additionally, include a date table. 

There are several tools online you can use, I'd recommend [Draw.io](https://www.drawio.com/) or [LucidChart](https://www.lucidchart.com/pages/).

**HINT:** You do not need to create any data for this prompt. This is a conceptual model only. 

- The logical model PNG file is stored in /images/Assignment2--Section1_Prompt1_SQL_Jason _Pereira_drawio.png
<img src="./images/Assignment2--Section1_Prompt1_SQL_Jason _Pereira_drawio.png" width="900">

#### Prompt 2
We want to create employee shifts, splitting up the day into morning and evening. Add this to the ERD.

- The logical model PNG file is stored in /images/Assignment2--Section1_Prompt2_SQL_Jason _Pereira_drawio.png
<img src="./images/Assignment2--Section1_Prompt2_SQL_Jason _Pereira_drawio.png" width="900">

#### Prompt 3
The store wants to keep customer addresses. Propose two architectures for the CUSTOMER_ADDRESS table, one that will retain changes, and another that will overwrite. Which is type 1, which is type 2? 

**HINT:** search type 1 vs type 2 slowly changing dimensions. 

```

Type 1 Slowly Changing Dimension:  
- The 'customer_address' table overwrites the existing address whenever a customer’s address changes.  
- The table only retains the most recent address for each customer.  
- Historical data in the table is not preserved.  
- The table consists of columns: 'customer_id', 'street', 'city', 'province', 'postal_code', and 'country'.  
- This architecture does not allow for tracking of the customer address changes over time.  

Type 2 Slowly Changing Dimension:  
- The 'customer_adress' table adds a new row with the latest change in address details to reflect the new address.  
- The table retains the most recent address and all previous addresses recorded for each customer.  
- Historical data in the table is preserved.  
- The table consists of columns: 'customer_id', 'street', 'city', 'province', 'postal_code', and 'country'.  
- Every address row must include start and end dates (start_date and end_date) columns to indicate the address validity period.  
- The most recent address row end_date field should be NULL to indicate it is the latest recorded address.  
- This architecture will allow for better tracking of the customer address changes over time.  

```

***

## Section 2:
You can start this section following *session 4*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question


### Write SQL

#### COALESCE
1. Our favourite manager wants a detailed long list of products, but is afraid of tables! We tell them, no problem! We can produce a list with all of the appropriate details. 

Using the following syntax you create our super cool and not at all needy manager a list:
```
SELECT 
product_name || ', ' || product_size|| ' (' || product_qty_type || ')'
FROM product
```

But wait! The product table has some bad data (a few NULL values). 
Find the NULLs and then using COALESCE, replace the NULL with a blank for the first problem, and 'unit' for the second problem. 

**HINT**: keep the syntax the same, but edited the correct components with the string. The `||` values concatenate the columns into strings. Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. All the other rows will remain the same.

SELECT   
COALESCE(product_name, '') || ', ' || COALESCE(product_size, '') || ' (' || COALESCE(product_qty_type, 'unit') || ')'   
FROM product;  


<div align="center">-</div>

#### Windowed Functions
1. Write a query that selects from the customer_purchases table and numbers each customer’s visits to the farmer’s market (labeling each market date with a different number). Each customer’s first visit is labeled 1, second visit is labeled 2, etc. 

You can either display all rows in the customer_purchases table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits. 

**HINT**: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK().

-- Using ROW_NUMBER  
SELECT customer_id  
	,market_date  
    ,ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date) AS visit_number  
FROM customer_purchases;  

--Using dense_rank  
SELECT customer_id  
	 ,market_date   
       ,DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY market_date) AS visit_number  
FROM customer_purchases;  


2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, then write another query that uses this one as a subquery (or temp table) and filters the results to only the customer’s most recent visit.

WITH ranked_visits AS (  
    SELECT customer_id  
	,market_date   
		,ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date DESC) AS visit_number  
    FROM customer_purchases  
)  
SELECT customer_id  
,market_date  
,visit_number  
FROM ranked_visits  
WHERE visit_number = 1;  

3. Using a COUNT() window function, include a value along with each row of the customer_purchases table that indicates how many different times that customer has purchased that product_id.

SELECT customer_id  
    ,product_id  
    ,market_date  
    ,COUNT(*) OVER (PARTITION BY customer_id, product_id) AS purchase_count  
FROM customer_purchases;  


<div align="center">-</div>

#### String manipulations
1. Some product names in the product table have descriptions like "Jar" or "Organic". These are separated from the product name with a hyphen. Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. Remove any trailing or leading whitespaces. Don't just use a case statement for each product! 

| product_name               | description |
|----------------------------|-------------|
| Habanero Peppers - Organic | Organic     |

**HINT**: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column. 

SELECT product_name  
    ,TRIM(SUBSTR(product_name, INSTR(product_name, '-') + 1)) AS description  
FROM product  
WHERE INSTR(product_name, '-') > 0;  

/* 2. Filter the query to show any product_size value that contain a number with REGEXP. */

SELECT product_name  
    ,product_size  
FROM product  
WHERE product_size REGEXP '[0-9]';  

<div align="center">-</div>

#### UNION
1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

**HINT**: There are a possibly a few ways to do this query, but if you're struggling, try the following: 1) Create a CTE/Temp Table to find sales values grouped dates; 2) Create another CTE/Temp table with a rank windowed function on the previous query to create "best day" and "worst day"; 3) Query the second temp table twice, once for the best day, once for the worst day, with a UNION binding them. 

--Calculating total_sales grouped by market_date  
WITH sales_values_grouped_by_date AS (  
    SELECT market_date   
        ,SUM(quantity*cost_to_customer_per_qty) AS total_sales  
    FROM customer_purchases  
    GROUP BY market_date  
),  

--Ranking the market dates based on total_sales  
ranked_sales AS (  
    SELECT market_date   
        ,total_sales  
        ,RANK() OVER (ORDER BY total_sales DESC) AS rank_desc  
        ,RANK() OVER (ORDER BY total_sales ASC) AS rank_asc  
    FROM sales_values_grouped_by_date  
)  
--Returning the best and worst sales days using UNION  
SELECT market_date  
    ,total_sales  
	,'Best Day of Sales' AS description  
FROM ranked_sales   
WHERE rank_desc = 1  
UNION  
SELECT market_date   
    ,total_sales  
	,'Worst Day of Sales' AS description  
FROM ranked_sales  
WHERE rank_asc = 1;  


***

## Section 3:
You can start this section following *session 5*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question

### Write SQL

#### Cross Join
1. Suppose every vendor in the `vendor_inventory` table had 5 of each of their products to sell to **every** customer on record. How much money would each vendor make per product? Show this by vendor_name and product name, rather than using the IDs.

**HINT**: Be sure you select only relevant columns and rows. Remember, CROSS JOIN will explode your table rows, so CROSS JOIN should likely be a subquery. Think a bit about the row counts: how many distinct vendors, product names are there (x)? How many customers are there (y). Before your final group by you should have the product of those two queries (x\*y). 

--Counting total number of unique customers  
WITH total_customers AS (  
    SELECT COUNT(DISTINCT customer_id) AS num_customers  
    FROM customer_purchases  
),  

--Calculating total_product_revenue  
product_revenue AS (  
    SELECT   
        product_id  
        ,SUM(quantity * cost_to_customer_per_qty) AS total_product_revenue  
    FROM customer_purchases  
    GROUP BY product_id  
),  

--JOIN vendor, product, and revenue values  
vendor_product_revenue AS (  
    SELECT v.vendor_name  
        ,p.product_name  
        ,pr.total_product_revenue * 5 * tc.num_customers AS total_revenue  
    FROM vendor v  
    INNER JOIN vendor_inventory vi ON v.vendor_id = vi.vendor_id  
    INNER JOIN product p ON vi.product_id = p.product_id  
    INNER JOIN product_revenue pr ON vi.product_id = pr.product_id  
    CROSS JOIN total_customers tc  
)  

SELECT vendor_name  
    ,product_name  
    ,total_revenue  
FROM vendor_product_revenue;  

<div align="center">-</div>

#### INSERT
1. Create a new table "product_units". This table will contain only products where the `product_qty_type = 'unit'`. It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  Name the timestamp column `snapshot_timestamp`.

DROP TABLE IF EXISTS product_units;  

CREATE TABLE product_units AS  
SELECT *   
    ,CURRENT_TIMESTAMP AS snapshot_timestamp  
FROM product  
WHERE product_qty_type = 'unit';  

SELECT * FROM product_units;  


2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie). 

INSERT INTO product_units (product_id, product_name, product_size, product_qty_type, snapshot_timestamp)  
VALUES (5, 'Butter Tart', 'small', 'unit', CURRENT_TIMESTAMP);  
SELECT * FROM product_units;  

<div align="center">-</div>

#### DELETE 
1. Delete the older record for the whatever product you added.

**HINT**: If you don't specify a WHERE clause, [you are going to have a bad time](https://imgflip.com/i/8iq872).

DELETE FROM product_units  
WHERE product_name = 'Butter Tart'  
  AND snapshot_timestamp = (  
      SELECT MIN(snapshot_timestamp)  
      FROM product_units  
      WHERE product_name = 'Butter Tart'  
  );  
SELECT * FROM product_units;  

<div align="center">-</div>

#### UPDATE
1. We want to add the current_quantity to the product_units table. First, add a new column, `current_quantity` to the table using the following syntax.
```
ALTER TABLE product_units
ADD current_quantity INT;
```

Then, using `UPDATE`, change the current_quantity equal to the **last** `quantity` value from the vendor_inventory details. 

**HINT**: This one is pretty hard. First, determine how to get the "last" quantity per product. Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) Third, `SET current_quantity = (...your select statement...)`, remembering that WHERE can only accommodate one column. Finally, make sure you have a WHERE statement to update the right row, you'll need to use `product_units.product_id` to refer to the correct row within the product_units table. When you have all of these components, you can run the update statement.

--include the current_quantity column  
ALTER TABLE product_units  
ADD current_quantity;  
SELECT * FROM product_units;  

--updating current_quantity column with the last quantity per customer  
UPDATE product_units  
SET current_quantity = (  
    SELECT COALESCE(vi.quantity, 0)   
    FROM vendor_inventory vi  
    WHERE vi.product_id = product_units.product_id  
)  
WHERE product_id IN (  
    SELECT DISTINCT product_id   
    FROM vendor_inventory  
);  
SELECT * FROM product_units  
ORDER BY product_units.current_quantity DESC;  
