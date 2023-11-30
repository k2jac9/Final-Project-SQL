### Question 1: What is the engagement level?
```
-- Average Time Spent & Page Views on Top Pages. This will help us understand the engagement level of users on these pages.
SELECT 
    pageTitle,
    AVG(timeOnSite) AS average_time_spent,
    AVG(pageviews) AS average_page_views
FROM temp_all_sessions
WHERE pageTitle IN (
    'YouTube | Shop by Brand | Google Merchandise Store', 
    'Men''s T-Shirts | Apparel | Google Merchandise Store',
    'Men''s-T-Shirts',
    'Apparel | Google Merchandise Store',
    'Men''s Outerwear | Apparel | Google Merchandise Store',
    'Electronics | Google Merchandise Store',
    'Bags | Google Merchandise Store',
    'Store search results',
    'Google | Shop by Brand | Google Merchandise Store',
    'Drinkware | Google Merchandise Store'
)
GROUP BY pageTitle
ORDER BY average_time_spent DESC;
```
![image](https://github.com/k2jac9/Project-SQL/assets/5405628/eb233957-c8fe-4f6c-9c11-4604a693eb61)


### Question 2: What are the conversion rates
```
--  Conversion rate: calculate the conversion rate for top pages as the ratio of sessions that resulted in a purchase to the total sessions.
SELECT 
    pageTitle,
    COUNT(*) AS total_sessions,
    SUM(CASE WHEN transactions > 0 THEN 1 ELSE 0 END) AS conversion_sessions,
    (SUM(CASE WHEN transactions > 0 THEN 1 ELSE 0 END)::FLOAT / COUNT(*)) * 100 AS conversion_rate
FROM temp_all_sessions
WHERE pageTitle IN (
    'YouTube | Shop by Brand | Google Merchandise Store', 
    'Men''s T-Shirts | Apparel | Google Merchandise Store',
    'Men''s-T-Shirts',
    'Apparel | Google Merchandise Store',
    'Men''s Outerwear | Apparel | Google Merchandise Store',
    'Electronics | Google Merchandise Store',
    'Bags | Google Merchandise Store',
    'Store search results',
    'Google | Shop by Brand | Google Merchandise Store',
    'Drinkware | Google Merchandise Store'
)
GROUP BY pageTitle
ORDER BY conversion_rate DESC LIMIT 5;
```
![image](https://github.com/k2jac9/Project-SQL/assets/5405628/26bbcdc1-7991-4073-b8f0-56bb865ca4f5)


### Question 3: What are the top pages viewed from organic search
```
SELECT pageTitle, COUNT(*) as view_count
FROM temp_all_sessions
WHERE channelGrouping = 'Organic Search' 
GROUP BY pageTitle
ORDER BY view_count DESC
LIMIT 10;
```
![image](https://github.com/k2jac9/Project-SQL/assets/5405628/876f3919-65f8-46d0-be33-06a0019a71bd)


### Question 4: What is the breakdown of missing revenue across different channels.
```
SELECT channelgrouping, COUNT(*)
FROM temp_analytics
WHERE revenue IS NULL
GROUP BY channelgrouping
ORDER BY COUNT(*);

Answer:
It could indicate the rank of effectiviness of certain channels over others or problems recording the data.
```
![image](https://github.com/k2jac9/Project-SQL/assets/5405628/ffdbedd3-b110-480d-ab26-9da7a052a15a)


### Question 5: How to set a workflow for Ecommerce Analysis?
```
-- ECOMMERCE ANALYSIS
-- 1. Check columns with missing values and nature of the missing data. For the analytics table, we noticed that almost all values for userid are missing:
SELECT channelgrouping, COUNT(userid)
FROM temp_analytics
GROUP BY channelgrouping;		
-- 0

-- 2. Time Series Analysis. Look for patterns of missing data across different dates to make sure it's not a specific time related problem.
SELECT date, COUNT(userid)
FROM temp_analytics
WHERE userid IS NULL
GROUP BY date
ORDER BY date DESC
LIMIT 100;					
-- 0

-- Associative Analysis. 
SELECT COUNT(*)
FROM temp_analytics
WHERE revenue IS NULL AND units_sold = 0;		
-- Answer: 0 There are no rows where revenue is missing, and units_sold is 0. 
-- This suggests that the missing revenue data is not directly tied to zero sales. We might need to further explore other potential reasons or patterns for why revenue might be missing in some records. Potentially we could have to calculate it. 
```
