# Data-Analysis-For-Zomato

**SQL Project: Data Analysis for Zomato - A Food Delivery Company**

**Overview**

This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

**Project Structure**

**Database Setup:** Creation of the zomato_db database and the required tables.

**Data Import:** Inserting sample data into the tables.

**Data Cleaning:** Handling null values and ensuring data integrity.

**Business Problems:** Solving 18 specific business problems using SQL queries.

**Database Setup**

```
  CREATE DATABASE zomato_db;
```

1. **Dropping Existing Tables**
```
 DROP TABLE IF EXISTS deliveries;
 DROP TABLE IF EXISTS Orders;
 DROP TABLE IF EXISTS customers;
 DROP TABLE IF EXISTS restaurants;
 DROP TABLE IF EXISTS riders;
```

2. **Creating Tables**
```   
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    restaurant_name VARCHAR(100) NOT NULL,
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);
```
```
    CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    reg_date DATE
);
```
```
    CREATE TABLE riders (
    rider_id SERIAL PRIMARY KEY,
    rider_name VARCHAR(100) NOT NULL,
    sign_up DATE
);
```
```
    CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(255),
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    order_status VARCHAR(20) DEFAULT 'Pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);
```
```
    CREATE TABLE deliveries (
    delivery_id SERIAL PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(20) DEFAULT 'Pending',
    delivery_time TIME,
    rider_id INT,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
);
```

Data Import
Data Cleaning and Handling Null Values
Before performing analysis, I ensured that the data was clean and free from null values where necessary. For instance:

**UPDATE orders**
```
SET total_amount = COALESCE(total_amount, 0);
```

**Business Problems Solved**
1. **Most Frequently Ordered Dishes by a Specific Customer in the Last Year**
```
SELECT
    customer_id,
    order_item,
    COUNT(*)
FROM orders
WHERE order_date > CURRENT_DATE - INTERVAL '360 days'
GROUP BY 1, 2
ORDER BY 1, 3 DESC;
```

3. **Popular Time Slots for Orders (2-Hour Intervals)**
**Approach 1:**
```
SELECT
    CASE 
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
        -- Additional cases
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM Orders
GROUP BY time_slot
ORDER BY order_count DESC;
```

**Approach 2:**
```
SELECT
    FLOOR(EXTRACT(HOUR FROM order_time) / 2) * 2 as start_hour,
    FLOOR(EXTRACT(HOUR FROM order_time) / 2) * 2 + 2 AS end_hour,
    COUNT(*)
FROM orders
GROUP BY 1, 2;
```

3. **Average Order Value for High-Volume Customers**

**This query identifies customers who have placed more than 750 orders and calculates their average spend.**
```
SELECT
    customer_id,
    AVG(total_amount)
FROM orders
GROUP BY 1
HAVING COUNT(*) > 750;
```

4. **High-Value Customers (Spending Over 100K)**
**This query focuses on the total monetary value contributed by each customer.**
```
SELECT
    customer_id,
    AVG(total_amount)
FROM orders
GROUP BY 1
HAVING SUM(total_amount) > 100000;
```

5. **Orders Without Delivery**
**This query uses a subquery to find all records in the orders table that do not have a corresponding entry in the deliveries table.**
```
SELECT * FROM orders
WHERE order_id NOT IN (SELECT order_id FROM deliveries);
```

**Detailed Business Problem Version:**

**Question: Write a query to find orders that were placed but not delivered.**

**Return: Restaurant name, city, and the number of undelivered orders.**
```
SELECT
    r.restaurant_name,
    r.city,
    COUNT(o.order_id) AS undelivered_orders
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE o.order_id NOT IN (SELECT order_id FROM deliveries)
GROUP BY r.restaurant_name, r.city;
```

6. **Restaurant Revenue Ranking**
**Question: Rank restaurants by their total revenue from the last year, including their name, total revenue, and rank within their city.**
```
SELECT
    r.city,
    r.restaurant_name,
    SUM(o.total_amount) AS total_revenue,
    RANK() OVER (PARTITION BY r.city ORDER BY SUM(o.total_amount) DESC) AS revenue_rank
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE o.order_date > CURRENT_DATE - INTERVAL '1 year'
GROUP BY r.city, r.restaurant_name;
```

8. **Cancellation Rate Comparison**
**Question: Calculate and compare the order cancellation rate for each restaurant between the current year and the previous year.**
```
SELECT
    r.restaurant_name,
    EXTRACT(YEAR FROM o.order_date) AS year,
    SUM(CASE WHEN o.order_status = 'Cancelled' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS cancellation_rate
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE EXTRACT(YEAR FROM o.order_date) IN (EXTRACT(YEAR FROM CURRENT_DATE), EXTRACT(YEAR FROM CURRENT_DATE) - 1)
GROUP BY r.restaurant_name, year;
```

10. **Rider Average Delivery Time**
**Question: Determine each rider's average delivery time.**
```
SELECT
    r.rider_name,
    AVG(EXTRACT(EPOCH FROM (d.delivery_time - o.order_time))/60) AS avg_delivery_time_minutes
FROM deliveries d
JOIN orders o ON d.order_id = o.order_id
JOIN riders r ON d.rider_id = r.rider_id
GROUP BY r.rider_name;
```

11. **Monthly Restaurant Growth Ratio**
**Question: Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining.**
```
SELECT
    r.restaurant_name,
    TO_CHAR(o.order_date, 'YYYY-MM') AS month,
    COUNT(d.delivery_id) AS delivered_orders,
    LAG(COUNT(d.delivery_id)) OVER (PARTITION BY r.restaurant_name ORDER BY TO_CHAR(o.order_date, 'YYYY-MM')) AS prev_month_orders,
    (COUNT(d.delivery_id) * 1.0 / LAG(COUNT(d.delivery_id)) OVER (PARTITION BY r.restaurant_name ORDER BY TO_CHAR(o.order_date, 'YYYY-MM'))) AS growth_ratio
FROM orders o
JOIN deliveries d ON o.order_id = d.order_id
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY r.restaurant_name, month;
```

12.**Customer Segmentation**
**Question: Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). If a customer's total spending exceeds the AOV, label them as 'Gold'; otherwise, label them as 'Silver'. Determine each segment's total number of orders and total revenue.**
```
WITH aov AS (
    SELECT AVG(total_amount) AS avg_order_value
    FROM orders
),
customer_spending AS (
    SELECT
        o.customer_id,
        SUM(o.total_amount) AS total_spending,
        COUNT(o.order_id) AS total_orders
    FROM orders o
    GROUP BY o.customer_id
)
SELECT
    cs.customer_id,
    CASE
        WHEN cs.total_spending > (SELECT avg_order_value FROM aov) THEN 'Gold'
        ELSE 'Silver'
    END AS customer_segment,
    cs.total_orders,
    cs.total_spending
FROM customer_spending cs;
```


11. **Rider Monthly Earnings**
**Question: Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.**
```
SELECT
    r.rider_name,
    TO_CHAR(o.order_date, 'YYYY-MM') AS month,
    SUM(o.total_amount) * 0.08 AS total_earnings
FROM deliveries d
JOIN orders o ON d.order_id = o.order_id
JOIN riders r ON d.rider_id = r.rider_id
GROUP BY r.rider_name, month;
```
12. **Rider Ratings Analysis**
**Question: Calculate the number of 5-star, 4-star, and 3-star ratings each rider receives based on delivery time.**

**5 stars:** Delivered in less than 15 minutes.

**4 stars:** Delivered in 15-20 minutes.

**3 stars:** Delivered in more than 20 minutes.
```
SQL
SELECT
    r.rider_name,
    SUM(CASE WHEN EXTRACT(EPOCH FROM (d.delivery_time - o.order_time))/60 < 15 THEN 1 ELSE 0 END) AS five_stars,
    SUM(CASE WHEN EXTRACT(EPOCH FROM (d.delivery_time - o.order_time))/60 BETWEEN 15 AND 20 THEN 1 ELSE 0 END) AS four_stars,
    SUM(CASE WHEN EXTRACT(EPOCH FROM (d.delivery_time - o.order_time))/60 > 20 THEN 1 ELSE 0 END) AS three_stars
FROM deliveries d
JOIN orders o ON d.order_id = o.order_id
JOIN riders r ON d.rider_id = r.rider_id
GROUP BY r.rider_name;
```

13. **Order Frequency by Day**
**Question: Analyze order frequency per day of the week and identify the peak day for each restaurant.**
```
SELECT
    r.restaurant_name,
    TO_CHAR(o.order_date, 'Day') AS day_of_week,
    COUNT(o.order_id) AS order_count
FROM orders o
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY r.restaurant_name, day_of_week
ORDER BY r.restaurant_name, order_count DESC;
```


14. **Customer Lifetime Value (CLV)**
**Question: Calculate the total revenue generated by each customer over all their orders.**
```
SELECT
    c.customer_name,
    SUM(o.total_amount) AS lifetime_value
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_name
ORDER BY lifetime_value DESC;
```

15. **Monthly Sales Trends**
**Question: Identify sales trends by comparing each month's total sales to the previous month.**
```
SELECT
    TO_CHAR(order_date, 'YYYY-MM') AS month,
    SUM(total_amount) AS total_sales,
    LAG(SUM(total_amount)) OVER (ORDER BY TO_CHAR(order_date, 'YYYY-MM')) AS previous_month_sales,
    (SUM(total_amount) - LAG(SUM(total_amount)) OVER (ORDER BY TO_CHAR(order_date, 'YYYY-MM'))) AS sales_diff
FROM orders
GROUP BY month
ORDER BY month;
```

16. **Rider Efficiency**
**Question: Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.**
```
SELECT
    r.rider_name,
    AVG(EXTRACT(EPOCH FROM (d.delivery_time - o.order_time))/60) AS avg_delivery_time_minutes
FROM deliveries d
JOIN orders o ON d.order_id = o.order_id
JOIN riders r ON d.rider_id = r.rider_id
GROUP BY r.rider_name
ORDER BY avg_delivery_time_minutes;
```

17. **Order Item Popularity**
**Question: Track the popularity of specific order items over time and identify seasonal demand spikes.**
```
SELECT
    o.order_item,
    TO_CHAR(o.order_date, 'YYYY-MM') AS month,
    COUNT(o.order_id) AS order_count
FROM orders o
GROUP BY o.order_item, month
ORDER BY o.order_item, month;
```

18. **Monthly Restaurant Growth Ratio**
**Question: Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining.**
```
SELECT
    r.restaurant_name,
    TO_CHAR(o.order_date, 'YYYY-MM') AS month,
    COUNT(d.delivery_id) AS delivered_orders,
    LAG(COUNT(d.delivery_id)) OVER (PARTITION BY r.restaurant_name ORDER BY TO_CHAR(o.order_date, 'YYYY-MM')) AS prev_month_orders,
    (COUNT(d.delivery_id) - LAG(COUNT(d.delivery_id)) OVER (PARTITION BY r.restaurant_name ORDER BY TO_CHAR(o.order_date, 'YYYY-MM'))) / 
    LAG(COUNT(d.delivery_id)) OVER (PARTITION BY r.restaurant_name ORDER BY TO_CHAR(o.order_date, 'YYYY-MM'))::FLOAT AS growth_ratio
FROM restaurants r
JOIN orders o ON r.restaurant_id = o.restaurant_id
JOIN deliveries d ON o.order_id = d.order_id
GROUP BY r.restaurant_name, month
ORDER BY r.restaurant_name, month;
```

**Conclusion**

This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems in the context of a food delivery service like Zomato. 
The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

**Notice**

All customer names and data used in this project are computer-generated using AI and random functions. 
They do not represent real data associated with Zomato or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.


