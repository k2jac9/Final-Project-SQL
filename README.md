# Project

To analyze user behavior and product performance using data from various tables in a digital merchandise store.

## Goals

- Identify best-selling products by region.
- Determine the impact of revenue from each city/country.
- Pinpoint issues in the data, such as missing values or inconsistencies, and find solutions for them.
- Analyze user search behavior and its correlation with sales.

## Process

### Data Preparation

- Created temporary tables for easier handling and to make sure original data remains untouched.
- Renamed columns for consistent terminology and transformed data types for efficient querying.

### Data Cleaning

- Identified and filled missing values in key columns using imputation techniques.
- Detected and rectified anomalies in city-country relationships and product data.

### Data Analysis

- Combined tables using JOIN operations to get holistic datasets.
- Used aggregation functions to get insights on product sales, user behavior, and revenue generation.

### Optimization

- Created reusable functions to streamline the quality assurance process.
- Fine-tuned queries to minimize run-time and maximize efficiency.

## Results

- Unearthed the top-selling products across different regions, giving a clear direction for targeted marketing.
- Discovered cities and countries that contributed the most to revenue, allowing for better resource allocation in those regions.
- Identified discrepancies in the data, such as mismatched city-country pairs and missing sentiment scores, and devised strategies to handle them.

## Challenges

- Handling Missing Values: A significant number of rows had missing data, especially in sentiment scores and certain location data.
- Inconsistent Data Formats: Data from different regions sometimes used different naming conventions and units.
- Large Dataset: The sheer size of the dataset posed challenges in processing speed and efficiency.
- Data Anomalies: Encountered unexpected anomalies, like the wrong cities listed for countries.

## Future Goals

- Deep Dive into User Behavior: Analyze patterns in user search behavior to make website navigation more intuitive.
- Predictive Analytics: Use this data to predict future sales trends and stock requirements.
- Improve Data Collection: Work with the data collection team to ensure better data quality at the source.
- Interactive Dashboard: Develop a real-time dashboard to showcase these insights in a more visual and interactive manner.
- Scalability: Ensure the system can handle even larger datasets in the future without compromising on speed or efficiency.

## Appendix

- Presentation
