PROPERTY PORTIFOLIO ANALYTICS:SQL JOINS & WINDOW FUNCTIONS PROJECT

COURSE :DATABASE DEVRELOPMENT WITH PL/SQL 
INSTRUCTOR:MANIRAGUHA ERIC STUDENT NAME:MUGISHA IYAN GAEL 
STUDENTID:29190

1.PROBLEM DEFINITION STEP1
TechMart is a regional e-commerce company operating across East Africa (Rwanda, Kenya, Uganda). 
The sales department needs to understand customer purchasing patterns, product performance, 
and regional sales trends to optimize inventory and marketing strategies.

**Data Challenge:**
The company currently lacks visibility into which products perform best in specific regions, 
how customer purchasing behavior changes over time, and which customer segments generate 
the most revenue. Without this analysis, the company risks overstocking low-performing 
products and missing opportunities to retain high-value customers.

**Expected Outcome:**
Identify top-performing products per region, analyze customer purchase frequency and value, 
segment customers into actionable marketing groups, and detect sales trends to inform 
inventory planning and targeted promotional campaigns.

 Step 2: Success Criteria

Copy this into your README.md:

markdown
Success Criteria

This analysis aims to achieve five measurable goals:

1. **Top Product Identification:** Rank top 5 products per region by revenue using RANK()
2. **Sales Trend Analysis:** Calculate running monthly sales totals using SUM() OVER()
3. **Growth Measurement:** Compute month-over-month sales growth using LAG()
4. **Customer Segmentation:** Divide customers into 4 quartiles by total spending using NTILE(4)
5. **Moving Average Trends:** Calculate 3-month moving averages for sales forecasting using AVG() OVER()

 Step 4: Part A - SQL JOINs

### JOIN 1: INNER JOIN
Retrieve all completed transactions with customer and product details

JOIN 1: INNER JOIN
 Retrieve transactions with valid customers and products

SELECT 
    o.order_id,
    c.customer_name,
    c.region,
    p.product_name,
    p.category,
    o.quantity,
    o.total_amount,
    o.order_date
FROM Orders o
INNER JOIN Customers c ON o.customer_id = c.customer_id
INNER JOIN Products p ON o.product_id = p.product_id
ORDER BY o.order_date DESC;
![INNER JOIN](https://github.com/user-attachments/assets/bdb146b8-aaa6-4bb3-8ae9-5b80e9a76184)






 JOIN 2: LEFT JOIN



SELECT 
    c.customer_id,
    c.customer_name,
    c.region,
    c.join_date,
    COUNT(o.order_id) as total_orders,
    COALESCE(SUM(o.total_amount), 0) as total_spent
FROM Customers c
LEFT JOIN Orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.region, c.join_date
HAVING COUNT(o.order_id) = 0
ORDER BY c.join_date;
![INNER JOIN](https://github.com/user-attachments/assets/b8af414d-f576-4b50-a87b-37f527b1dc5c)




 JOIN 3: RIGHT JOIN

 Detect products that have never been sold



SELECT 
    p.product_id,
    p.product_name,
    p.category,
    p.unit_price,
    COUNT(o.order_id) as times_sold,
    COALESCE(SUM(o.quantity), 0) as total_units_sold
FROM Orders o
RIGHT JOIN Products p ON o.product_id = p.product_id
GROUP BY p.product_id, p.product_name, p.category, p.unit_price
HAVING COUNT(o.order_id) = 0
ORDER BY p.category, p.unit_price DESC;
![WhatsApp Image 2026-02-08 at 11 42 49 PM](https://github.com/user-attachments/assets/28ea8915-f00b-43fb-aa6f-9ede977f2857)


 JOIN 4: FULL OUTER JOIN

 Complete view of customers and products including unmatched records



SELECT 
    c.customer_name,
    c.region,
    p.product_name,
    p.category,
    o.total_amount,
    o.order_date,
    CASE 
        WHEN o.order_id IS NULL THEN 'No Transaction'
        ELSE 'Transaction Exists'
    END as transaction_status
FROM Customers c
FULL OUTER JOIN Orders o ON c.customer_id = o.customer_id
FULL OUTER JOIN Products p ON o.product_id = p.product_id
ORDER BY c.customer_name, o.order_date;
![WhatsApp Image 2026-02-08 at 11 43 39 PM](https://github.com/user-attachments/assets/83936428-9d61-4c9f-9031-11d1fdf3217c)



JOIN 5: SELF JOIN

 Compare customers within the same region



SELECT 
    c1.customer_name as customer_1,
    c2.customer_name as customer_2,
    c1.region as shared_region,
    c1.join_date as customer1_join_date,
    c2.join_date as customer2_join_date,
    ABS(c1.join_date - c2.join_date) as days_between_joining
FROM Customers c1
INNER JOIN Customers c2 
    ON c1.region = c2.region 
    AND c1.customer_id < c2.customer_id
ORDER BY c1.region, days_between_joining;
![WhatsApp Image 2026-02-08 at 11 44 28 PM](https://github.com/user-attachments/assets/0953c208-ae72-4ac7-b38c-aabc35339b1f)

## Step 5: Part B - Window Functions

### Category 1: Ranking Functions

#### 1.1 ROW_NUMBER()



SELECT 
    p.product_name,
    p.category,
    SUM(o.total_amount) as total_revenue,
    ROW_NUMBER() OVER (ORDER BY SUM(o.total_amount) DESC) as revenue_rank
FROM Products p
INNER JOIN Orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY revenue_rank;

![WhatsApp Image 2026-02-08 at 11 45 23 PM](https://github.com/user-attachments/assets/d6a0cf71-54ba-4a0b-bb60-7ba8f95b812a)



 1.2 RANK() - Top Products per Region



WITH 
    SELECT 
        c.region,
        p.product_name,
        SUM(o.total_amount) as revenue
    FROM Orders o
    INNER JOIN Customers c ON o.customer_id = c.customer_id
    INNER JOIN Products p ON o.product_id = p.product_id
    GROUP BY c.region, p.product_name
)
SELECT 
    region,
    product_name,
    revenue,
    RANK() OVER (PARTITION BY region ORDER BY revenue DESC) as product_rank
FROM RegionalSales
ORDER BY region, product_rank;
![WhatsApp Image 2026-02-08 at 11 46 05 PM](https://github.com/user-attachments/assets/b5846b82-fbdc-4a4d-8537-27c0d8625e93)



 Category 2: Aggregate Window Functions

 2.1 Running Total with SUM() OVER()



SELECT 
    TO_CHAR(order_date, 'YYYY-MM') as sales_month,
    SUM(total_amount) as monthly_revenue,
    SUM(SUM(total_amount)) OVER (ORDER BY TO_CHAR(order_date, 'YYYY-MM') 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM Orders
GROUP BY TO_CHAR(order_date, 'YYYY-MM')
ORDER BY sales_month;
![WhatsApp Image 2026-02-08 at 11 46 05 PM](https://github.com/user-attachments/assets/db10a091-8eba-4b37-b354-db9e99569ffd)

: Distribution Functions

 4.1 NTILE(4) - Customer Quartile Segmentation



SELECT 
    c.customer_name,
    c.region,
    SUM(o.total_amount) as total_spent,
    NTILE(4) OVER (ORDER BY SUM(o.total_amount)) as spending_quartile,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY SUM(o.total_amount)) = 4 THEN 'Premium'
        WHEN NTILE(4) OVER (ORDER BY SUM(o.total_amount)) = 3 THEN 'High Value'
        WHEN NTILE(4) OVER (ORDER BY SUM(o.total_amount)) = 2 THEN 'Medium Value'
        ELSE 'Low Value'
    END as customer_segment
FROM Customers c
INNER JOIN Orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.region
ORDER BY total_spent DESC;
![WhatsApp Image 2026-02-08 at 11 53 13 PM](https://github.com/user-attachments/assets/b02d22dd-7107-4499-9a10-d70a6a6d144b)



