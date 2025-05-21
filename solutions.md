# Walmart SQL Analysis – SQL Solutions

-- querying all rows and columns 
select * from sales 

--How many total sales records are there?
SELECT COUNT(*) AS total_records FROM sales;

-- What are the different product lines sold
select DISTINCT Product_line from sales

-- What is the total quantity sold by each branch?
SELECT Branch, SUM(Quantity) AS total_quantity
FROM sales
GROUP BY Branch;

--Which city had the highest gross income?
SELECT City, SUM(gross_income) AS total_gross_income
FROM sales
GROUP BY City
ORDER BY total_gross_income DESC
LIMIT 1;

--How many sales were made by male vs female customers?
SELECT Gender, COUNT(*) AS total_sales
FROM sales
GROUP BY Gender;

-- Monthly gross income trend per branch
WITH monthly_income AS (
    SELECT 
        Branch,
        strftime('%m', Date) AS month,
        SUM([gross_income]) AS monthly_income
    FROM sales
    GROUP BY Branch, month
)
SELECT * FROM monthly_income
ORDER BY Branch, month;

-- Get branches with above-average total sales
WITH branch_sales AS (
    SELECT Branch, SUM(Total) AS total_sales
    FROM sales
    GROUP BY Branch
),
avg_sales AS (
    SELECT AVG(total_sales) AS avg_branch_sales FROM branch_sales
)SELECT bs.*
FROM branch_sales bs, avg_sales
WHERE bs.total_sales > avg_sales.avg_branch_sales;

-- Find the date with the highest total sales.
SELECT Date, SUM(Total) AS daily_sales
FROM sales
GROUP BY Date
ORDER BY daily_sales DESC

--What is the average unit price per product line?
SELECT Product_line, AVG(Unit_price) AS avg_unit_price
FROM sales
GROUP BY Product_line;

--What are the top 3 product lines by gross income?
SELECT Product_line, SUM(gross_income) AS total_income
FROM sales
GROUP BY Product_line
ORDER BY total_income DESC
LIMIT 5

-- Calculate average quantity and filter transactions above that average
WITH avg_qty AS (
    SELECT AVG(Quantity) AS avg_quantity FROM sales
)
SELECT *
FROM sales, avg_qty
WHERE sales.Quantity > avg_qty.avg_quantity;

--What is the total Tax 5% collected per branch?
SELECT Branch, SUM("Tax 5%") AS total_tax_collected
FROM sales
GROUP BY Branch;

-- Find the top 3 product lines by average rating in each city
WITH avg_rating_cte AS (
    SELECT 
        City,
        Product_line,
        AVG(Rating) AS avg_rating,
        RANK() OVER (PARTITION BY city ORDER BY AVG(Rating) DESC) AS rank_in_city
    FROM sales
    GROUP BY City, [Product_line]
)
SELECT * 
FROM avg_rating_cte
WHERE rank_in_city <= 3;


--Rank branches by average customer rating
SELECT Branch,
       AVG(Rating) AS avg_rating,
       DENSE_RANK() OVER (ORDER BY AVG(Rating) DESC) AS branch_rank
FROM sales
GROUP BY Branch;

--Rank each customer’s purchases within their city by total amount spent
SELECT Invoice_ID,
       Customer_type,
       City,
       Total,
       RANK() OVER (PARTITION BY City, Customer_type ORDER BY Total DESC) AS purchase_rank
FROM sales;

--Add a cumulative total sales column per branch (running total)
SELECT Invoice_ID,
       Branch,
       Date,
       Total,
       SUM(Total) OVER (PARTITION BY Branch ORDER BY Date) AS running_total
FROM sales;

--Find top 3 highest rated transactions per branch
SELECT *
FROM (
    SELECT *,
           RANK() OVER (PARTITION BY Branch ORDER BY Rating DESC) AS rating_rank
    FROM sales
) sub
WHERE rating_rank <= 3;

--Find the top-selling product line by total quantity sold.
SELECT Product_line, SUM(Quantity) AS total_quantity
FROM sales
GROUP BY Product_line
ORDER BY total_quantity DESC
LIMIT 5

--Rank product lines by total gross income (across all branches)
SELECT Product_line,
       SUM(gross_income) AS total_income,
       RANK() OVER (ORDER BY SUM(gross_income) DESC) AS income_rank
FROM sales
GROUP BY Product_line;



