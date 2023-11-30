## Data Cleaning and Exploration

During the Data Exploration and cleaning process, we developed a workflow that helped us get familiar with the data, understand it while including a vast amount of checks for our Quality Assurance process ensuring data integrity and consistency.

### Queries: Below, provide the SQL queries you used to clean your data.

#### Check the first few rows of each table to get an overview

During the Data Exploration and cleaning process we developed a workflow that helped us get familiar with the data, understand it while including a vast amount of checks for our Quality Assurance process ensuring data integrity and consistency.

Queries: Below, provide the SQL queries you used to clean your data.

Data Cleaning and Exploration
```
-- Check the first few rows of each table to get an overview SELECT * FROM temp_all_sessions LIMIT 10; SELECT * FROM temp_analytics LIMIT 10; SELECT * FROM temp_products LIMIT 10; SELECT * FROM temp_sales_by_sku LIMIT 10; SELECT * FROM temp_sales_report LIMIT 10;

-- Check the number of rows in each table SELECT COUNT() FROM temp_all_sessions;
-- 15,134 SELECT COUNT() FROM temp_analytics;
-- 4,301,122 SELECT COUNT() FROM temp_products;
-- 1,092
SELECT COUNT() FROM temp_sales_by_sku;
-- 462 SELECT COUNT(*) FROM temp_sales_report;
-- 454

-- Initial number of columns in each table SELECT COUNT() FROM information_schema.columns WHERE table_name = 'temp_all_sessions'; -- 32 SELECT COUNT() FROM information_schema.columns WHERE table_name = 'temp_analytics'; -- 14 SELECT COUNT() FROM information_schema.columns WHERE table_name = 'temp_products'; -- 7 SELECT COUNT() FROM information_schema.columns WHERE table_name = 'temp_sales_by_sku'; -- 2 SELECT COUNT(*) FROM information_schema.columns WHERE table_name = 'temp_sales_report'; -- 8 -- Total 63

-- Update unit_price in the 'analytics' table based on cleaning hint into a new temp table called temp_analytics as updated_unit_price SELECT unit_price FROM temp_analytics;

SELECT * FROM temp_analytics;

-- Add the temp_updated_price_unit column and set its values: ALTER TABLE temp_analytics ADD COLUMN updated_price_unit numeric;

UPDATE temp_analytics SET updated_price_unit = unit_price / 1000000.0;

-- Review changes SELECT * FROM temp_analytics; -- 15 columns now

-- QA:

SELECT COUNT(unit_price) FROM analytics WHERE unit_price = 0; -- 188,314 same value in both tables

SELECT COUNT(unit_price) FROM temp_analytics WHERE unit_price = 0; -- 188,314 same value in both tables

SELECT COUNT(updated_price_unit) FROM temp_analytics WHERE unit_price = 0; -- 188,314 same value in updated_price_unit

-- Check missing values with a function to a temp table

CREATE OR REPLACE FUNCTION check_missing_values() RETURNS void AS $$ DECLARE v_table_name text; v_column_name text; column_data_type text; missing_count integer; BEGIN -- Drop the temporary table if it already exists DROP TABLE IF EXISTS temp_missing_values;

-- Create a new temporary table
CREATE TEMP TABLE temp_missing_values (
    table_name text,
    column_name text,
    missing_count integer
);

-- Loop through tables, columns, and data types in the pg_temp_5 schema, where the temp tables are located
FOR v_table_name, v_column_name, column_data_type IN 
    (SELECT table_name, column_name, data_type 
     FROM information_schema.columns 
     WHERE table_schema = 'pg_temp_5')
LOOP
    IF column_data_type = 'text' THEN
        EXECUTE format('SELECT COUNT(*) FROM %I WHERE %I IS NULL OR %I = ''''', 
                       v_table_name, v_column_name, v_column_name) INTO missing_count;
    ELSE
        EXECUTE format('SELECT COUNT(*) FROM %I WHERE %I IS NULL', 
                       v_table_name, v_column_name) INTO missing_count;
    END IF;
                   
    -- Insert results into the temporary table
    INSERT INTO temp_missing_values (table_name, column_name, missing_count)
    VALUES (v_table_name, v_column_name, missing_count);
END LOOP;
END; $$ LANGUAGE plpgsql;

-- Invoke the function to execute and will populate the missing values in the temp table SELECT check_missing_values(); -- Query from the temp table SELECT * FROM temp_missing_values;

-- See missing values in descensding order SELECT * FROM temp_missing_values WHERE missing_count > 0 ORDER BY missing_count DESC, table_name, column_name; /* "table_name" "column_name" "missing_count" "analytics" "userid" 4,301,122 "analytics" "revenue" 4,285,767 "analytics" "units_sold" 4,205,975 "analytics" "bounces" 3,826,283 "analytics" "timeonsite" 477,465 "temp_all_sessions" "itemquantity" 1,5134 "temp_all_sessions" "itemrevenue" 1,5134 "temp_all_sessions" "productrefundamount" 1,5134 "temp_all_sessions" "searchkeyword" 1,5134 "temp_all_sessions" "productrevenue" 1,5130 "temp_all_sessions" "transactionrevenue" 1,5130 "temp_all_sessions" "transactionid" 1,5125 "temp_all_sessions" "ecommerceaction_option" 1,5103 "temp_all_sessions" "productquantity" 1,5081 "temp_all_sessions" "totaltransactionrevenue" 1,5053 "temp_all_sessions" "transactions" 1,5053 "temp_all_sessions" "sessionqualitydim" 1,3906 "temp_all_sessions" "timeonsite" 3300 "temp_all_sessions" "currencycode" 272 "temp_sales_report" "ratio" 78 "analytics" "pageviews" 72 "temp_all_sessions" "pagetitle" 1 "products" "sentimentmagnitude" 1 "products" "sentimentscore" 1 */

-- Total records: 4,301,122 "analytics.revenue" = 4,285,767 = % 99.6

-- Total number of counts of missing values group by table name SELECT table_name, COUNT() as count_per_table FROM temp_missing_values WHERE missing_count > 0 GROUP BY table_name ORDER BY count_per_table DESC, table_name; / "table_name" "count_per_table" "temp_all_sessions" 15 "analytics" 6 "products" 2 "temp_sales_report" 1 */

-- NOTES: -- Products table: After checking products table sentiment score values and sentiment magnitude values will be updated in temp_products. -- temp_all_sessions table: Ratio could be calculated.

-- Dealing with missing values: -- temp_product:

SELECT * FROM temp_products;

-- sentimentscore column: -- It seems as missing sentiment score indicates neutral sentiment therefore it's appropriate to fill missing values with zeros. UPDATE temp_products SET sentimentscore = COALESCE(sentimentscore, 0);

-- sentimentmagnitude column: -- Mean Imputation: replace the NULL values in the sentimentmagnitude column with the average of the non-NULL values in the same column.

WITH MeanScore AS ( -- Calculate the average, round it, and cast it to a numeric type with specific precision SELECT CAST(ROUND(AVG(sentimentmagnitude), 2) AS NUMERIC(10,2)) as mean_score FROM temp_products WHERE sentimentmagnitude IS NOT NULL ) -- Update the NULL values with the rounded average UPDATE temp_products SET sentimentmagnitude = (SELECT mean_score FROM MeanScore) WHERE sentimentmagnitude IS NULL;

-- Check for changes with comment SELECT * FROM temp_products WHERE SKU = 'GGADFBSBKS42347'; -- Comment to prevent caching

SELECT column_name, missing_count FROM temp_missing_values WHERE table_name = 'products' AND missing_count > 0 ORDER BY missing_count DESC, column_name;

SELECT * FROM temp_products WHERE sentimentmagnitude IS NULL OR sentimentscore IS NULL;

SELECT SUM(CASE WHEN sentimentmagnitude IS NULL THEN 1 ELSE 0 END) AS missing_sentimentmagnitude, SUM(CASE WHEN sentimentscore IS NULL THEN 1 ELSE 0 END) AS missing_sentimentscore FROM temp_products;

-- Search for NULL values in each column of temp_products SELECT 'sku' AS column_name, COUNT() AS null_count FROM temp_products WHERE sku IS NULL UNION ALL SELECT 'name', COUNT() FROM temp_products WHERE name IS NULL UNION ALL SELECT 'orderedQuantity', COUNT() FROM temp_products WHERE orderedQuantity IS NULL UNION ALL SELECT 'stockLevel', COUNT() FROM temp_products WHERE stockLevel IS NULL UNION ALL SELECT 'restockingLeadTime', COUNT() FROM temp_products WHERE restockingLeadTime IS NULL UNION ALL SELECT 'sentimentScore', COUNT() FROM temp_products WHERE sentimentScore IS NULL UNION ALL SELECT 'sentimentMagnitude', COUNT(*) FROM temp_products WHERE sentimentMagnitude IS NULL ORDER BY null_count DESC, column_name;

-- Total number of columns with missing values group by table name after fixing temp_products missing values SELECT table_name, COUNT() as count_per_table FROM temp_missing_values WHERE missing_count > 0 GROUP BY table_name ORDER BY count_per_table DESC, table_name; / "table_name" "count_per_table" "temp_all_sessions" 15 "analytics" 6 "temp_sales_report" 1 */

-- Dealing with missing values: -- temp_sales_report: ratio column

SELECT * FROM temp_sales_report;

SELECT ratio FROM temp_sales_report;

-- Calculate sales-to-stock ratio and compare with the provided ratio using a subquery with division by zero handling -- Calculate sales-to-stock ratio and update the original table WITH computed_ratios AS ( SELECT name, CASE WHEN stocklevel = 0 THEN NULL ELSE total_ordered::float / stocklevel::float END AS calculated_ratio FROM temp_sales_report ) UPDATE temp_sales_report SET ratio = computed_ratios.calculated_ratio FROM computed_ratios WHERE temp_sales_report.name = computed_ratios.name AND (temp_sales_report.ratio IS NULL OR temp_sales_report.ratio = 0); -- This ensures we only update where ratio is missing or zero

SELECT SUM(CASE WHEN name IS NULL THEN 1 ELSE 0 END) AS null_name, SUM(CASE WHEN total_ordered IS NULL THEN 1 ELSE 0 END) AS null_total_ordered, SUM(CASE WHEN stocklevel IS NULL THEN 1 ELSE 0 END) AS null_stocklevel, SUM(CASE WHEN ratio IS NULL THEN 1 ELSE 0 END) AS null_ratio FROM temp_sales_report;

/* 55 values: This values are Null because the sotck level is 0 Products with zero orders might be new or not popular. The similarity between calculated_ratio and provided_ratio suggests the ratio is a "sales-to-stock" measure. A ratio of 0.125 implies one item is sold for every eight in stock. The "ratio" column likely shows how often a product sells compared to its stock. Recommendations: Restock items with higher ratios first. Boost marketing for items with low ratios. Think about discontinuing long-standing, unsold products. */

-- Checking for duplicates: Investigate if productSKU from other tables are the same values as SKU from the products table ALTER TABLE temp_products RENAME COLUMN sku TO productSKU; ALTER TABLE temp_products RENAME COLUMN productSKU TO sku;

SELECT productSKU, COUNT() FROM temp_products GROUP BY productSKU HAVING COUNT() > 1; -- 0

SELECT sku, COUNT() FROM temp_products GROUP BY sku HAVING COUNT() > 1; -- 0

SELECT productSKU, COUNT() FROM temp_sales_report GROUP BY productSKU HAVING COUNT() > 1 ORDER BY COUNT(*) DESC; -- 0

SELECT productSKU, COUNT() FROM temp_sales_by_sku GROUP BY productSKU HAVING COUNT() > 1 ORDER BY COUNT(*) DESC; -- 0

SELECT productSKU, COUNT() FROM temp_all_sessions GROUP BY productSKU HAVING COUNT() > 1 ORDER BY COUNT(*) DESC; -- product_sku multiple times

SELECT sr.productSKU AS sales_report_sku, ss.productSKU AS sales_by_sku_sku FROM temp_sales_report sr JOIN temp_sales_by_sku ss ON sr.productSKU = ss.productSKU ORDER BY sr.productSKU;

SELECT (SELECT COUNT(DISTINCT productSKU) FROM temp_sales_report) AS distinct_skus_sales_report, (SELECT COUNT(DISTINCT productSKU) FROM temp_sales_by_sku) AS distinct_skus_sales_by_sku, COUNT(DISTINCT sr.productSKU) AS matching_skus FROM temp_sales_report sr JOIN temp_sales_by_sku ss ON sr.productSKU = ss.productSKU;

-- 454. It sugges a 100% match between temp_sales_report and temp_sales_by_sku for the productSKU column.
```
