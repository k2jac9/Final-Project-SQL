# Risks Identified

## Problems with data Importation and Ingestion:

Large datasets or varying formats can lead to challenges during data import.
Network issues, corrupted files, or unstructured data can create ingestion problems.

## Challenges in Building Robust Data Pipelines

* Developing a seamless and efficient data pipeline can be intricate, requiring careful planning and testing.
* Real-time data, evolving schemas, or changing requirements can impact pipeline robustness.

## Inconsistent Code Formats from Different Regions
With data gathered from multiple regions, there's a risk of encountering various data formats, naming conventions, or units.

## Need for Data Translation
Some data might be in a foreign language or contain regional slang, requiring translation or standardization for proper analysis.

## Quality Assurance Process

The Quality Assurance process adopted, the team wisely used TEMPORARY TABLES and functions in PostgreSQL. This approach had several advantages:

### Reproducibility
Using TEMPORARY TABLES ensured that the original data remained untouched, allowing the process to be repeated without affecting the base dataset.

### Data Integrity
Since the main tables weren't directly manipulated, data integrity was maintained throughout the process.

### Modularity
The use of functions promoted a tidy and organized workflow. Specific data checks or modifications were bundled within these functions for efficiency.

### Iterative Development
The approach allowed for easy adjustments and refinements. If any issues were found, the team could swiftly make changes and re-run the QA process.

### Efficiency 
TEMPORARY TABLES often led to faster query execution, speeding up the entire QA process.

### Error Handling
PostgreSQL functions supported exception handling, ensuring thoroughness in the QA process.

### Clear Workflow
The step-by-step structure provided an understandable audit trail of the data validations and checks that were executed.

By adopting this method, the team ensured a robust, consistent, and efficient Quality Assurance process, positioning them well for maintaining high data quality standards.

## QA Process:
Describe your QA process and include the SQL queries used to execute it.

-- LANGUAGE: SQL
-- Database: ecommerce
-- Author: Alejandro Castellanos
-- GitHub: k2jac9
-- Tools: PostgreSQL, pgAdmin4, vsCode, Gihut copilot, Excel
-- Date: 2021-10-04
-- Time: 18:00:00 America/Toronto Timezone abbreviated as EDT (Eastern Daylight Time) UTC -4:00
-- Project SQL
-- Description: This is a project to create a database called ecommerce and load data into 5 tables and create 5 temporary tables with the proper data types assigned to them

-- 1. Create a database called ecommerce
-- 2. Connect to the database using vsCode
-- 3. Create 5 tables
-- 3.1 Create table called all_sessions
-- 3.2 Create table called analytics
-- 3.3 Create table called products
-- 3.4 Create table called sales_report
-- 3.5 Create table called sales_by_sku
-- 4. Import data to 5 tables
-- 4.1 Import data to all_sessions
-- 4.2 Import data to analytics
-- 4.3 Import data to products,
-- 4.4 Import data to sales_report
-- 4.5 Import data to sales_by_sku
-- 5. Create TEMPORARY TABLES
-- 5.1 Create TEMPORARY TABLE called  temp_all_sessions FROM all_sessions
-- 5.2 Create TEMPORARY TABLE called temp_analytics FROM analytics
-- 5.3 Create TEMPORARY TABLE called temp_products FROM products
-- 5.4 Create TEMPORARY TABLE called temp_sales_report FROM
-- 5.3 Create TEMPORARY TABLE called temp_sales_by_sku FROM sales_by_sku
-- 6. Data cleaning and Exploration using temp tables


-- 1. Create a database called ecommerce
-- CREATE DATABASE ecommerce;


-- 2. CONNECTION TO THE DATABASE
-- host | port | user | password | \c ecommerce


-- 3. Create 5 Tables

-- 3.1 Create a table called all_sessions

-- DROP TABLE IF EXISTS all_sessions;

CREATE TABLE all_sessions (
    fullVisitorId NUMERIC,
    channelGrouping TEXT,
    time INTEGER,
    country TEXT,
    city TEXT,
    totalTransactionRevenue DECIMAL,
    transactions INTEGER,
    timeOnSite INTEGER,
    pageviews INTEGER,
    sessionQualityDim INTEGER,
    date DATE,
    visitId BIGINT,
    type TEXT,
    productRefundAmount DECIMAL,
    productQuantity INTEGER,
    productPrice DECIMAL,
    productRevenue DECIMAL,
    productSKU TEXT,
    v2ProductName TEXT,
    v2ProductCategory TEXT,
    productVariant TEXT,
    currencyCode TEXT,
    itemQuantity INTEGER,
    itemRevenue DECIMAL,
    transactionRevenue DECIMAL,
    transactionId TEXT,
    pageTitle TEXT,
    searchKeyword TEXT,
    pagePathLevel1 TEXT,
    eCommerceAction_type INTEGER,
    eCommerceAction_step INTEGER,
    eCommerceAction_option TEXT
);

-- 3.2 Create table called analytics

-- DROP TABLE IF EXISTS analytics;

CREATE TABLE analytics (
    visitNumber INTEGER,
    visitId BIGINT,
    visitStartTime BIGINT,
    date DATE,
    fullvisitorId NUMERIC,
    userid TEXT,
    channelGrouping TEXT,
    socialEngagementType TEXT,
    units_sold INTEGER,
    pageviews INTEGER,
    timeonsite INTEGER,
    bounces INTEGER,
    revenue DECIMAL,
    unit_price DECIMAL
);

-- 3.3 Create table called products

-- DROP TABLE IF EXISTS products;

CREATE TABLE products (
    sku TEXT,
    name TEXT,
    orderedQuantity INTEGER,
    stockLevel INTEGER,
    restockingLeadTime INTEGER,
    sentimentScore DECIMAL,
    sentimentMagnitude DECIMAL
);

-- 3.4 Create table called sales_report

-- DROP TABLE IF EXISTS sales_report;

CREATE TABLE sales_report (
    productsku TEXT,
    total_ordered INTEGER,
    name TEXT,
    stockLevel INTEGER,
    restockingLeadTime INTEGER,
    sentimentScore DECIMAL,
    sentimentMagnitude DECIMAL,
    ratio DECIMAL
);

-- 3.5 Create table called sales_by_sku

-- DROP TABLE IF EXISTS sales_by_sku;

CREATE TABLE sales_by_sku (
    productsku TEXT,
    total_ordered INTEGER
);

-- 4. Import data to 5 tables: Data imported using pgAdmin 4

-- 5. Create TEMPORARY TABLES
-- 5.1 Create TEMPORARY TABLE called  temp_all_sessions FROM all_sessions

DROP TABLE IF EXISTS temp_all_sessions;

CREATE TEMPORARY TABLE temp_all_sessions AS
    SELECT * FROM all_sessions;

-- 5.2 Create TEMPORARY TABLE called temp_analytics FROM analytics

DROP TABLE IF EXISTS temp_analytics;

CREATE TEMPORARY TABLE temp_analytics AS
    SELECT * FROM analytics;

-- 5.3 Create TEMPORARY TABLE called temp_products FROM products

DROP TABLE IF EXISTS temp_products;

CREATE TEMPORARY TABLE temp_products AS
    SELECT * FROM products;

-- 5.4 Create TEMPORARY TABLE called temp_sales_report FROM sales_report

DROP TABLE IF EXISTS temp_sales_report;

CREATE TEMPORARY TABLE temp_sales_report AS
    SELECT * FROM sales_report;

-- 5.5 Create TEMPORARY TABLE called temp_sales_by_sku FROM sales_by_sku

DROP TABLE IF EXISTS temp_sales_by_sku;

CREATE TEMPORARY TABLE temp_sales_by_sku AS
    SELECT * FROM sales_by_sku;


-- See data types orders by table name and position of the column in the table
SELECT 
    table_name,
    column_name,
    data_type
FROM 
    information_schema.columns
WHERE 
    table_schema = 'public'
ORDER BY 
    table_name, ordinal_position;

-- QA: Data Structure and Integrity
-- See data types orders by table name and position of the column in the temp table from pg_temp_5
SELECT 
    table_name,
    column_name,
    data_type
FROM 
    information_schema.columns
WHERE 
    table_schema = 'pg_temp_5'
ORDER BY 
    table_name, ordinal_position;

