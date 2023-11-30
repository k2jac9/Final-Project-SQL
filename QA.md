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
```
-- LANGUAGE: SQL
-- Database: ecommerce
-- Author: Alejandro Castellanos
-- GitHub: k2jac9
-- Tools: PostgreSQL, pgAdmin4, vsCode, Gihut copilot, Excel
-- Date: 2023-11-24
-- Time: 11:11:11🕚 America/Toronto Timezone abbreviated as EDT (Eastern Time) UTC -5:00
-- Final Project SQL
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
```
## Sistematic QA analysis leveraging dynamic SQL using PostgreSQL
```
import psycopg2

# Database connection parameters - update these with your database information
dbname = "postgres"  # Typically, you can use 'postgres' to list other databases
user = "your_username"
password = "your_password"
host = "your_host"

# Connect to your postgres DB
conn = psycopg2.connect(dbname=dbname, user=user, password=password, host=host)

# Open a cursor to perform database operations
cur = conn.cursor()

try:
    # Execute a query to retrieve all database names
    cur.execute("SELECT datname FROM pg_database WHERE datistemplate = false;")
    databases = cur.fetchall()

    for db in databases:
        print(db[0])

finally:
    # Close the cursor and the connection
    cur.close()
    conn.close()

    # Importing necessary library
import pandas as pd

# Function to get total count, null values, non-distinct values, and data types for each column in a DataFrame
def get_stats(df):
    stats = pd.DataFrame({
        'Column': df.columns,
        'Data Type': df.dtypes.values,
        'Total Records': len(df),
        'Null Values': df.isnull().sum().values,
        'Non-Distinct Values': df.nunique().values
    })
    return stats

# Function to display statistics with a table name
def display_stats_with_table_name(stats, table_name):
    print(f"Statistics for {table_name} Table:")
    display(stats)
    print("\n")

# Function to drop columns with total records equal to null values
def drop_columns_with_null_records(df):
    return df.drop(df.columns[df.isnull().sum() == len(df)], axis=1)

# Get statistics for each table and drop columns
all_sessions = drop_columns_with_null_records(all_sessions)
analytics = drop_columns_with_null_records(analytics)
products = drop_columns_with_null_records(products)
sales_by_sku = drop_columns_with_null_records(sales_by_sku)
sales_report = drop_columns_with_null_records(sales_report)

# Get updated statistics for each table
all_sessions_stats = get_stats(all_sessions)
analytics_stats = get_stats(analytics)
products_stats = get_stats(products)
sales_by_sku_stats = get_stats(sales_by_sku)
sales_report_stats = get_stats(sales_report)

# Display the updated statistics with table names
display_stats_with_table_name(all_sessions_stats, "All Sessions")
display_stats_with_table_name(analytics_stats, "Analytics")
display_stats_with_table_name(products_stats, "Products")
display_stats_with_table_name(sales_by_sku_stats, "Sales by SKU")
display_stats_with_table_name(sales_report_stats, "Sales Report")

Statistics for All Sessions Table:
Column	Data Type	Total Records	Null Values	Non-Distinct Values
0	fullVisitorId	uint64	15134	0	14223
1	channelGrouping	object	15134	0	7
2	time	int64	15134	0	9600
3	country	object	15134	0	136
4	city	object	15134	0	266
5	totalTransactionRevenue	float64	15134	15053	72
6	transactions	float64	15134	15053	1
7	timeOnSite	float64	15134	3300	1266
8	pageviews	int64	15134	0	29
9	sessionQualityDim	float64	15134	13906	44
10	date	int64	15134	0	366
11	visitId	int64	15134	0	14556
12	type	object	15134	0	2
13	productQuantity	float64	15134	15081	8
14	productPrice	int64	15134	0	141
15	productRevenue	float64	15134	15130	4
16	productSKU	object	15134	0	536
17	v2ProductName	object	15134	0	471
18	v2ProductCategory	object	15134	0	74
19	productVariant	object	15134	0	11
20	currencyCode	object	15134	272	1
21	transactionRevenue	float64	15134	15130	4
22	transactionId	object	15134	15125	9
23	pageTitle	object	15134	1	268
24	pagePathLevel1	object	15134	0	11
25	eCommerceAction_type	int64	15134	0	7
26	eCommerceAction_step	int64	15134	0	3
27	eCommerceAction_option	object	15134	15103	3


Statistics for Analytics Table:
Column	Data Type	Total Records	Null Values	Non-Distinct Values
0	visitNumber	int64	4301122	0	222
1	visitId	int64	4301122	0	148642
2	visitStartTime	int64	4301122	0	148853
3	date	int64	4301122	0	93
4	fullvisitorId	uint64	4301122	0	120018
5	channelGrouping	object	4301122	0	8
6	socialEngagementType	object	4301122	0	1
7	units_sold	float64	4301122	4205975	134
8	pageviews	float64	4301122	72	128
9	timeonsite	float64	4301122	477465	3269
10	bounces	float64	4301122	3826283	1
11	revenue	float64	4301122	4285767	5269
12	unit_price	int64	4301122	0	1442


Statistics for Products Table:
Column	Data Type	Total Records	Null Values	Non-Distinct Values
0	SKU	object	1092	0	1092
1	name	object	1092	0	313
2	orderedQuantity	int64	1092	0	224
3	stockLevel	int64	1092	0	262
4	restockingLeadTime	int64	1092	0	27
5	sentimentScore	float64	1092	1	17
6	sentimentMagnitude	float64	1092	1	20


Statistics for Sales by SKU Table:
Column	Data Type	Total Records	Null Values	Non-Distinct Values
0	productSKU	object	462	0	462
1	total_ordered	int64	462	0	60


Statistics for Sales Report Table:
Column	Data Type	Total Records	Null Values	Non-Distinct Values
0	productSKU	object	454	0	454
1	total_ordered	int64	454	0	60
2	name	object	454	0	237
3	stockLevel	int64	454	0	219
4	restockingLeadTime	int64	454	0	26
5	sentimentScore	float64	454	0	16
6	sentimentMagnitude	float64	454	0	20
7	ratio	float64	454	78	244
```
