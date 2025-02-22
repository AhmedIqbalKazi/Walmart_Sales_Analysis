
# Walmart Data Analysis: End-to-End SQL + Python Project 

## Project Overview

![Project Pipeline](https://github.com/najirh/Walmart_SQL_Python/blob/main/walmart_project-piplelines.png)

This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions. 

---

## Project Steps

### 1. Set Up the Environment
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (MySQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Set Up Kaggle API
   - **API Setup**: Obtain your Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to your profile settings and downloading the JSON file.
   - **Configure Kaggle**: 
      - Place the downloaded `kaggle.json` file in your local `.kaggle` folder.
      - Use the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into your project.

### 3. Download Walmart Sales Data
   - **Data Source**: Use the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
   - **Storage**: Save the data in the `data/` folder for easy reference and access.

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas numpy sqlalchemy mysql-connector-python psycopg2
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Use functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

### 6. Data Cleaning
   - **Remove Duplicates**: Identify and remove duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Drop rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Use `.replace()` to handle and format currency values for analysis.
   - **Validation**: Check for any remaining inconsistencies and verify the cleaned data.

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

### 8. Load Data into MySQL 
   - **Set Up Connections**: Connect to MySQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in  MySQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Run initial SQL queries to confirm that the data has been loaded accurately.

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as:
     
     - **Find different payment methods, number of transactions, and quantity sold by payment method**
       ```sql
       SELECT payment_method, COUNT(*) AS No_payments, SUM(quantity) AS no_quantity_sold
       FROM walmart
       GROUP BY payment_method;
       ```

     - **Identify the highest-rated category in each branch**
       ```sql
       SELECT branch, category, avg_rating
       FROM (
           SELECT 
               branch,
               category,
               AVG(rating) AS avg_rating,
               RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS `rank`
           FROM walmart
           GROUP BY branch, category
       ) AS ranked
       WHERE `rank` = 1;
       ```

     - **Identify the busiest day for each branch based on the number of transactions**
       ```sql
       SELECT *
       FROM (
           SELECT Branch, DAYNAME(`date`) AS Day_name,
               COUNT(*) AS NO_transactions,
               RANK() OVER(PARTITION BY Branch ORDER BY COUNT(*) DESC) AS `Rank`
           FROM walmart
           GROUP BY Branch, Day_name
       ) AS Branch_rank
       WHERE `Rank` = 1;
       ```

     - **Determine the average, minimum, and maximum rating of categories for each city**
       ```sql
       SELECT city, category, MIN(rating) AS min_rating, MAX(rating) AS max_rating, AVG(rating) AS avg_rating
       FROM walmart
       GROUP BY city, category;
       ```

     - **Calculate the total profit for each category**
       ```sql
       SELECT category, SUM(Total_Price * profit_margin) AS total_profit
       FROM walmart
       GROUP BY category
       ORDER BY total_profit DESC;
       ```

     - **Determine the most common payment method for each branch**
       ```sql
       WITH cte AS (
           SELECT branch, payment_method, COUNT(*) AS total_trans,
                  RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS `rank`
           FROM walmart
           GROUP BY branch, payment_method
       )
       SELECT branch, payment_method AS preferred_payment_method
       FROM cte
       WHERE `rank` = 1;
       ```

     - **Categorize sales into Morning, Afternoon, and Evening shifts**
       ```sql
       SELECT branch, 
              CASE 
                  WHEN HOUR(time) < 12 THEN 'Morning'
                  WHEN HOUR(time) BETWEEN 12 AND 17 THEN 'Afternoon'
                  ELSE 'Evening'
              END AS shift, 
              COUNT(*) AS num_invoices
       FROM walmart
       GROUP BY branch, shift
       ORDER BY branch, num_invoices DESC;
       ```

     - **Identify the 5 branches with the highest revenue decrease ratio from last year to current year**
       ```sql
       WITH revenue_2022 AS (
           SELECT Branch, SUM(Total_Price) AS Revenue
           FROM walmart
           WHERE YEAR(`date`) = 2022
           GROUP BY Branch
       ), revenue_2023 AS (
           SELECT Branch, SUM(Total_Price) AS Revenue
           FROM walmart
           WHERE YEAR(`date`) = 2023
           GROUP BY Branch
       )
       SELECT r2022.Branch, r2022.revenue AS last_year_revenue, r2023.revenue AS current_year_revenue,
              ROUND(((r2022.revenue - r2023.revenue) / r2022.revenue) * 100, 2) AS revenue_decrease_ratio
       FROM revenue_2022 AS r2022
       JOIN revenue_2023 AS r2023 ON r2022.Branch = r2023.Branch
       WHERE r2022.revenue > r2023.revenue
       ORDER BY revenue_decrease_ratio DESC
       LIMIT 5;
       ```

---


## Requirements

- **Python 3.8+**
- **SQL Databases**: MySQL, 
- **Python Libraries**:
  - `pandas`, `numpy`, `sqlalchemy`, `mysql-connector-python`, `psycopg2`
- **Kaggle API Key** (for data downloading)



## Acknowledgments

- **Data Source**: Kaggle’s Walmart Sales Dataset
- **Inspiration**: Walmart’s business case studies on sales and supply chain optimization.

---

