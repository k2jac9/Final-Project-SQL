# Final Project SQL Questions


Answer the following questions and provide the SQL queries used to find the answer.

## Question 1: Which cities and countries have the highest level of transaction revenues on the site?

SQL Queries:

Question 1: Which cities and countries have the highest level of transaction revenues on the site?

SQL Queries: SELECT DISTINCT country, city FROM temp_all_sessions WHERE city NOT IN ('(not set)', 'not available in demo dataset') ORDER BY country, city;

/* Mistaken Cities: Australia: Los Angeles. Mountain View Canada: New York France: San Francisco, Singapore Hungary: Istanbul, Japan: Mountain View, San Francisco Netherlands: Dublin United States: Amsterdam, Bangkok, Hong Kong, London, Mexico City Paris, Toronto, Vancouver, Yokohama */

-- QA: FIX -- Since we don't have a way to dertermine or asusme the proper city -- Create temp table

-- DROP TABLE IF EXISTS MistakenCities; CREATE TEMP TABLE MistakenCities(country TEXT, city TEXT);

INSERT INTO MistakenCities(country, city) VALUES ('Australia', 'Los Angeles'), ('Australia', 'Mountain View'), ('Canada', 'New York'), ('France', 'San Francisco'), ('France', 'Singapore'), ('Hungary', 'Istanbul'), ('Japan', 'Mountain View'), ('Japan', 'San Francisco'), ('Netherlands', 'Dublin'), ('United States', 'Amsterdam'), ('United States', 'Bangkok'), ('United States', 'Hong Kong'), ('United States', 'London'), ('United States', 'Mexico City'), ('United States', 'Paris'), ('United States', 'Toronto'), ('United States', 'Vancouver'), ('United States', 'Yokohama');

-- Update temp_all_sessions directly by setting city to 'not in right country' for matching records UPDATE temp_all_sessions SET city = 'not in right country' FROM MistakenCities mc WHERE temp_all_sessions.country = mc.country AND temp_all_sessions.city = mc.city;

-- Delete those records from temp_all_sessions where city is 'not in right country' DELETE FROM temp_all_sessions WHERE city = 'not in right country'; -- DELETE 20

-- Remove rows where totaltransactionrevenue is not greater than 0 and city is either '(not set)' or 'not available in demo dataset' DELETE FROM temp_all_sessions WHERE NOT (totaltransactionrevenue > 0 AND city NOT IN ('(not set)', 'not available in demo dataset')); -- DELETE 8,656

-- Top city by revenue for each country from TempAllSessions WITH RankedCities AS ( SELECT country, city, SUM(totaltransactionrevenue) as total_revenue, ROW_NUMBER() OVER(PARTITION BY country ORDER BY SUM(totaltransactionrevenue) DESC) as rnk FROM temp_all_sessions WHERE totaltransactionrevenue > 0 AND city NOT IN ('(not set)', 'not available in demo dataset') GROUP BY country, city ) SELECT country, city, total_revenue FROM RankedCities WHERE rnk = 1 ORDER BY total_revenue DESC; /*"country" "city" "total_revenue" "United States" "San Francisco" 1,564,320,000 "Israel" "Tel Aviv-Yafo" 602,000,000 "Australia" "Sydney" 358,000,000 "Canada" "Toronto" 82,160,000 "Switzerland" "Zurich" 16,990,000 */ -- Top countries by revenue from TempAllSessions SELECT country, SUM(totaltransactionrevenue) AS total_revenue FROM temp_all_sessions WHERE totaltransactionrevenue > 0 AND city NOT IN ('(not set)', 'not available in demo dataset') GROUP BY country ORDER BY total_revenue DESC; -- Same countries, different amountd due to countries with totaltransactions revenue and no city are included.

-- Top countries and city by revenue from temp_all_sessions and all_sessions SELECT country, city, SUM(totaltransactionrevenue) as total_revenue FROM temp_all_sessions WHERE totaltransactionrevenue > 0 AND city NOT IN ('(not set)', 'not available in demo dataset') GROUP BY country, city ORDER BY total_revenue DESC;

SELECT country, city, SUM(totaltransactionrevenue) as total_revenue FROM all_sessions WHERE totaltransactionrevenue > 0 AND city NOT IN ('(not set)', 'not available in demo dataset') GROUP BY country, city ORDER BY total_revenue DESC;

-- The results are the same as using TempAllSessions indicating data integritry and that the values of the top cities are not affected by the QA fix.

Answer: "country" "city" "total_revenue" "United States" "San Francisco" 1,564,320,000 "Israel" "Tel Aviv-Yafo" 602,000,000 "Australia" "Sydney" 358,000,000 "Canada" "Toronto" 82,160,000 "Switzerland" "Zurich" 16,990,000

## Question 2: What is the average number of products ordered from visitors in each city and country?

SQL Queries: -- Joining temp_all_sessions and temp_analytics based on fullvisitorId, calculating the average, and filtering out NULLs SELECT tas.country, tas.city, ROUND(AVG(ta.units_sold), 2) AS average_units_sold FROM temp_all_sessions tas JOIN temp_analytics ta ON tas.fullvisitorId = ta.fullvisitorId WHERE tas.country IS NOT NULL AND tas.city IS NOT NULL AND ta.units_sold IS NOT NULL GROUP BY tas.country, tas.city ORDER BY average_units_sold DESC, tas.country, tas.city;

Answer: "country" "city" "average_units_sold" "United States" "San Bruno" 52.67 "United States" "Mountain View" 16.17 "United States" "San Jose" 8.57 "United States" "Salem" 7.55 "United States" "New York" 6.90 "United States" "Chicago" 6.19 "United States" "Nashville" 5.33 "United States" "Jersey City" 5.27 "United States" "Charlotte" 4.65 "United States" "Sunnyvale" 4.07 "United States" "Pittsburgh" 4.00 "United States" "Austin" 3.61 "United States" "Los Angeles" 3.52 "United States" "Atlanta" 3.27 "United States" "Seattle" 3.10 "United States" "Cambridge" 2.69 "United States" "Houston" 2.50 "Canada" "Toronto" 2.19 "United States" "Denver" 2.00 "United States" "Kirkland" 1.90 "Germany" "Hamburg" 1.71 "India" "Hyderabad" 1.50 "United States" "Milpitas" 1.46 "United States" "Santa Monica" 1.40 "United States" "San Francisco" 1.26

## Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

SQL Queries: -- Counting the occurrences of each product category by country and city SELECT tas.country, tas.city, tas.v2ProductCategory, COUNT(*) AS order_count FROM temp_all_sessions tas WHERE tas.country IS NOT NULL AND tas.city IS NOT NULL AND tas.v2ProductCategory IS NOT NULL GROUP BY tas.country, tas.city, tas.v2ProductCategory ORDER BY tas.country, tas.city, order_count DESC;

-- Counting the distinct product categories by country and city WITH CategoryRank AS ( SELECT tas.country, tas.city, tas.v2ProductCategory, COUNT() AS order_count, ROW_NUMBER() OVER (PARTITION BY tas.country, tas.city ORDER BY COUNT() DESC) AS rank FROM temp_all_sessions tas WHERE tas.country IS NOT NULL AND tas.city IS NOT NULL AND tas.v2ProductCategory IS NOT NULL GROUP BY tas.country, tas.city, tas.v2ProductCategory )

SELECT cr.country, cr.city, COUNT(DISTINCT cr.v2ProductCategory) AS distinct_category_count, MAX(CASE WHEN rank = 1 THEN cr.v2ProductCategory END) AS top_category FROM CategoryRank cr GROUP BY cr.country, cr.city ORDER BY cr.country, cr.city, distinct_category_count DESC;

-- Identifying the product category with the highest overall order count Answer: -- Home/Apparel/Men's/Men's-T-Shirts/ SELECT tas.v2ProductCategory, COUNT(*) AS total_order_count FROM temp_all_sessions tas WHERE tas.v2ProductCategory IS NOT NULL GROUP BY tas.v2ProductCategory ORDER BY total_order_count DESC LIMIT 1;

Answer: Home/Apparel/Men's/Men's-T-Shirts/

## Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?

SQL Queries: WITH ProductRanks AS ( SELECT tas.country, tas.city, tas.v2ProductName, COUNT() AS product_sales_count, ROW_NUMBER() OVER (PARTITION BY tas.country, tas.city ORDER BY COUNT() DESC) AS rank FROM temp_all_sessions tas WHERE tas.v2ProductName IS NOT NULL GROUP BY tas.country, tas.city, tas.v2ProductName )

SELECT pr.country, pr.city, pr.v2ProductName AS top_selling_product, pr.product_sales_count FROM ProductRanks pr WHERE pr.rank = 1 ORDER BY pr.country, pr.city;

WITH ProductRanks AS ( SELECT tas.country, tas.city, tas.v2ProductName, COUNT() AS product_sales_count, ROW_NUMBER() OVER (PARTITION BY tas.country, tas.city ORDER BY COUNT() DESC) AS rank FROM temp_all_sessions tas WHERE tas.v2ProductName IS NOT NULL GROUP BY tas.country, tas.city, tas.v2ProductName ) SELECT pr.country, pr.city, pr.v2ProductName AS top_selling_product, pr.product_sales_count FROM ProductRanks pr WHERE pr.rank = 1 ORDER BY pr.country, pr.city;

SELECT v2ProductName AS product_name, COUNT(*) AS product_sales_count FROM temp_all_sessions WHERE v2ProductName IS NOT NULL GROUP BY v2ProductName ORDER BY product_sales_count DESC LIMIT 5;

Answer:

"product_name" "product_sales_count" "Google Men's 100% Cotton Short Sleeve Hero Tee White" 126 "Google Men's 100% Cotton Short Sleeve Hero Tee Navy" 76 "Google Men's 100% Cotton Short Sleeve Hero Tee Black" 68 "Google Men's Vintage Badge Tee Black" 66 "YouTube Twill Cap" 64

## Question 5: Can we summarize the impact of revenue generated from each city/country?

SQL Queries: SELECT country, city, SUM(totaltransactionrevenue) AS total_revenue FROM temp_all_sessions WHERE totaltransactionrevenue IS NOT NULL GROUP BY country, city ORDER BY total_revenue DESC;

Answer: United States represent 92% of total revenue "country" "city" "total_revenue" "United States" "San Francisco" 1564320000 "United States" "Sunnyvale" 992230000 "United States" "Atlanta" 854440000 "United States" "Palo Alto" 608000000 "Israel" "Tel Aviv-Yafo" 602000000 "United States" "New York" 530360000 "United States" "Mountain View" 483360000 "United States" "Los Angeles" 479480000 "United States" "Chicago" 449520000 "United States" "Seattle" 358000000 "Australia" "Sydney" 358000000 "United States" "San Jose" 262380000 "United States" "Austin" 157780000 "United States" "Nashville" 157000000 "United States" "San Bruno" 103770000 "Canada" "Toronto" 82160000 "United States" "Houston" 38980000 "United States" "Columbus" 21990000 "Switzerland" "Zurich" 16990000

We can see the same cities from Q1 "country" "city" "total_revenue" "United States" "San Francisco" 1,564,320,000 "Israel" "Tel Aviv-Yafo" 602,000,000 "Australia" "Sydney" 358,000,000 "Canada" "Toronto" 82,160,000 "Switzerland" "Zurich" 16,990,000







