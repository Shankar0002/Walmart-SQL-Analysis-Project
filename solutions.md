
# Walmart SQL Analysis – SQL Solutions

---

### Query 1: Query all rows and columns
```sql
SELECT * FROM sales;
```

---

### Query 2: How many total sales records are there?
```sql
SELECT COUNT(*) AS total_records FROM sales;
```

---

### Query 3: What are the different product lines sold?
```sql
SELECT DISTINCT Product_line FROM sales;
```

---

### Query 4: What is the total quantity sold by each branch?
```sql
SELECT Branch, SUM(Quantity) AS total_quantity
FROM sales
GROUP BY Branch;
```

---

### Query 5: Which city had the highest gross income?
```sql
SELECT City, SUM(gross_income) AS total_gross_income
FROM sales
GROUP BY City
ORDER BY total_gross_income DESC
LIMIT 1;
```

---

### Query 6: How many sales were made by male vs female customers?
```sql
SELECT Gender, COUNT(*) AS total_sales
FROM sales
GROUP BY Gender;
```

---

### Query 7: Monthly gross income trend per branch
```sql
WITH monthly_income AS (
    SELECT 
        Branch,
        strftime('%m', Date) AS month,
        SUM(gross_income) AS monthly_income
    FROM sales
    GROUP BY Branch, month
)
SELECT * FROM monthly_income
ORDER BY Branch, month;
```

---

### Query 8: Get branches with above-average total sales
```sql
WITH branch_sales AS (
    SELECT Branch, SUM(Total) AS total_sales
    FROM sales
    GROUP BY Branch
),
avg_sales AS (
    SELECT AVG(total_sales) AS avg_branch_sales FROM branch_sales
)
SELECT bs.*
FROM branch_sales bs, avg_sales
WHERE bs.total_sales > avg_sales.avg_branch_sales;
```

---

### Query 9: Find the date with the highest total sales
```sql
SELECT Date, SUM(Total) AS daily_sales
FROM sales
GROUP BY Date
ORDER BY daily_sales DESC
LIMIT 1;
```

---

### Query 10: What is the average unit price per product line?
```sql
SELECT Product_line, AVG(Unit_price) AS avg_unit_price
FROM sales
GROUP BY Product_line;
```

---

### Query 11: What are the top 3 product lines by gross income?
```sql
SELECT Product_line, SUM(gross_income) AS total_income
FROM sales
GROUP BY Product_line
ORDER BY total_income DESC
LIMIT 3;
```

---

### Query 12: Calculate average quantity and filter transactions above that average
```sql
WITH avg_qty AS (
    SELECT AVG(Quantity) AS avg_quantity FROM sales
)
SELECT *
FROM sales, avg_qty
WHERE sales.Quantity > avg_qty.avg_quantity;
```

---

### Query 13: What is the total Tax 5% collected per branch?
```sql
SELECT Branch, SUM("Tax 5%") AS total_tax_collected
FROM sales
GROUP BY Branch;
```

---

### Query 14: Find the top 3 product lines by average rating in each city
```sql
WITH avg_rating_cte AS (
    SELECT 
        City,
        Product_line,
        AVG(Rating) AS avg_rating,
        RANK() OVER (PARTITION BY City ORDER BY AVG(Rating) DESC) AS rank_in_city
    FROM sales
    GROUP BY City, Product_line
)
SELECT * 
FROM avg_rating_cte
WHERE rank_in_city <= 3;
```

---

### Query 15: Rank branches by average customer rating
```sql
SELECT Branch,
       AVG(Rating) AS avg_rating,
       DENSE_RANK() OVER (ORDER BY AVG(Rating) DESC) AS branch_rank
FROM sales
GROUP BY Branch;
```

---

### Query 16: Rank each customer’s purchases within their city by total amount spent
```sql
SELECT Invoice_ID,
       Customer_type,
       City,
       Total,
       RANK() OVER (PARTITION BY City, Customer_type ORDER BY Total DESC) AS purchase_rank
FROM sales;
```

---

### Query 17: Add a cumulative total sales column per branch (running total)
```sql
SELECT Invoice_ID,
       Branch,
       Date,
       Total,
       SUM(Total) OVER (PARTITION BY Branch ORDER BY Date) AS running_total
FROM sales;
```

---

### Query 18: Find top 3 highest rated transactions per branch
```sql
SELECT *
FROM (
    SELECT *,
           RANK() OVER (PARTITION BY Branch ORDER BY Rating DESC) AS rating_rank
    FROM sales
) sub
WHERE rating_rank <= 3;
```

---

### Query 19: Find the top-selling product line by total quantity sold
```sql
SELECT Product_line, SUM(Quantity) AS total_quantity
FROM sales
GROUP BY Product_line
ORDER BY total_quantity DESC
LIMIT 1;
```

---

### Query 20: Rank product lines by total gross income (across all branches)
```sql
SELECT Product_line,
       SUM(gross_income) AS total_income,
       RANK() OVER (ORDER BY SUM(gross_income) DESC) AS income_rank
FROM sales
GROUP BY Product_line;
```
