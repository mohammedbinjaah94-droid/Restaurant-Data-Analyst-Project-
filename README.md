# Restaurant-Data-Analyst-Project-

Of course. Here is the revised and complete project summary, updated to include all the analytical questions and with the Power BI section removed, ready for your GitHub `README.md` file.

-----

# Restaurant Performance Analysis Project

This project is a comprehensive analysis of a restaurant's operational data. The goal is to extract actionable insights to improve service efficiency, optimize staff performance, and enhance customer experience. The project workflow covers data cleaning with Python, database management with PostgreSQL, and in-depth analysis using advanced SQL queries.

##  Project Workflow

The project was executed in three main stages:

### 1\. Data Cleaning and Preparation (Python)

  - **Objective**: To prepare the raw dataset (`.csv`) for analysis by handling inconsistencies.
  - **Tools**: Python with the `pandas` library.
  - **Process**:
      - Loaded the raw data into a pandas DataFrame.
      - Inspected the data for missing values (`NaN`) and duplicates.
      - Handled missing `Total_Stay_Minutes` by filling them with `0`, as these corresponded to "Takeaway" orders.
      - Converted the `Total_Stay_Minutes` column to an integer type (`int`) to ensure compatibility with the database schema.
      - Saved the processed data into a new, clean file: `restaurant_data_cleaned.csv`.

### 2\. Database Setup (PostgreSQL)

  - **Objective**: To create a structured database to host and query the cleaned data.
  - **Tools**: PostgreSQL and `pgAdmin`.
  - **Process**:
      - Created a new database named `restaurant_db`.
      - Defined a table schema named `orders` with appropriate data types for each column.
      - Used the `COPY` command to efficiently bulk-import the cleaned data from `restaurant_data_cleaned.csv` into the `orders` table.

### 3\. Data Analysis (SQL Queries)

  - **Objective**: To ask and answer critical business questions using SQL.
  - **Process**: A series of SQL queries, ranging from basic to advanced, were written and executed to uncover patterns related to operations, staff, customers, and menu items.

-----

## 📊 SQL Queries for Analysis

Here is the complete list of analytical questions and the corresponding SQL queries used in this project.

### Level 1: Basic Operational Metrics

**1. What is the average, min, and max wait time for Dine-in vs. Takeaway?**

  * *Purpose: To measure service efficiency and customer experience.*

<!-- end list -->

```sql
SELECT
    Order_Type,
    AVG(Wait_Time_Minutes) AS "Average_Wait_Time",
    MIN(Wait_Time_Minutes) AS "Minimum_Wait_Time",
    MAX(Wait_Time_Minutes) AS "Maximum_Wait_Time"
FROM
    orders
GROUP BY
    Order_Type;
```

**2. What are the busiest hours of the restaurant?**

  * *Purpose: To identify peak times for better staff and resource allocation.*

<!-- end list -->

```sql
SELECT
    EXTRACT(HOUR FROM Visit_Time) AS "Hour_of_the_Day",
    COUNT(Customer_ID) AS "Number_of_Orders"
FROM
    orders
GROUP BY
    "Hour_of_the_Day"
ORDER BY
    "Number_of_Orders" DESC;
```

**3. Which staff member handles the most orders?**

  * *Purpose: To evaluate staff productivity and workload distribution.*

<!-- end list -->

```sql
SELECT
    Service_Staff,
    COUNT(Customer_ID) AS "Total_Orders_Handled"
FROM
    orders
GROUP BY
    Service_Staff
ORDER BY
    "Total_Orders_Handled" DESC;
```

**4. What are the top 10 most frequently ordered items?**

  * *Purpose: Essential for menu engineering and inventory management.*

<!-- end list -->

```sql
SELECT
    TRIM(UNNEST(STRING_TO_ARRAY(Order_Items, ','))) AS "Menu_Item",
    COUNT(*) AS "Total_Orders"
FROM
    orders
GROUP BY
    "Menu_Item"
ORDER BY
    "Total_Orders" DESC
LIMIT 10;
```

**5. What is the preferred payment method?**

  * *Purpose: To understand customer payment behavior.*

<!-- end list -->

```sql
SELECT
    Payment_Method,
    COUNT(Customer_ID) AS "Number_of_Transactions"
FROM
    orders
GROUP BY
    Payment_Method
ORDER BY
    "Number_of_Transactions" DESC;
```

**6. Which day of the week is the busiest?**

  * *Purpose: To plan for weekly demand fluctuations.*

<!-- end list -->

```sql
SELECT
    TO_CHAR(Visit_Date, 'Day') AS "Day_of_Week",
    COUNT(Customer_ID) AS "Number_of_Orders"
FROM
    orders
GROUP BY
    "Day_of_Week", EXTRACT(DOW FROM Visit_Date)
ORDER BY
    EXTRACT(DOW FROM Visit_Date);
```

### Level 2: Advanced Performance & Behavioral Analysis

**7. Who is the most efficient staff member at turning over tables?**

  * *Purpose: To measure staff efficiency in table management by analyzing the time after an order is completed.*

<!-- end list -->

```sql
SELECT
    Service_Staff,
    AVG(EXTRACT(EPOCH FROM (Exit_Time - Order_Completed_Time))) / 60 AS "Average_Dwell_Time_Minutes"
FROM
    orders
WHERE
    Order_Type = 'Dine-in' AND Exit_Time IS NOT NULL
GROUP BY
    Service_Staff
ORDER BY
    "Average_Dwell_Time_Minutes" ASC;
```

**8. How does customer group size affect wait times and stay duration?**

  * *Purpose: To understand the service dynamics for different party sizes.*

<!-- end list -->

```sql
SELECT
    Customer_Count,
    AVG(Wait_Time_Minutes) AS "Average_Wait_Time",
    AVG(Total_Stay_Minutes) AS "Average_Total_Stay"
FROM
    orders
WHERE
    Order_Type = 'Dine-in'
GROUP BY
    Customer_Count
ORDER BY
    Customer_Count;
```

**9. What are the most popular pairs of items ordered together?**

  * *Purpose: To identify cross-selling and combo meal opportunities (Market Basket Analysis).*

<!-- end list -->

```sql
WITH items AS (
  SELECT
    Customer_ID,
    TRIM(UNNEST(STRING_TO_ARRAY(Order_Items, ','))) AS item
  FROM orders
)
SELECT
    i1.item AS "Item_1",
    i2.item AS "Item_2",
    COUNT(*) AS "Frequency"
FROM
    items i1
JOIN
    items i2 ON i1.Customer_ID = i2.Customer_ID AND i1.item < i2.item
GROUP BY
    "Item_1", "Item_2"
ORDER BY
    "Frequency" DESC
LIMIT 10;
```

**10. What is the average service interval time for each staff member?**

  * *Purpose: To measure the core service speed from customer arrival to order completion.*

<!-- end list -->

```sql
SELECT
    Service_Staff,
    AVG(EXTRACT(EPOCH FROM (Order_Completed_Time - Visit_Time))) / 60 AS "Average_Service_Interval_Minutes"
FROM
    orders
GROUP BY
    Service_Staff
ORDER BY
    "Average_Service_Interval_Minutes" ASC;
```

### Level 3: Strategic & Predictive Intelligence

**11. Is there a correlation between order complexity and preparation time?**

  * *Purpose: To see if larger orders cause kitchen bottlenecks and how different staff handle them.*

<!-- end list -->

```sql
WITH OrderMetrics AS (
    SELECT
        Service_Staff,
        CARDINALITY(STRING_TO_ARRAY(Order_Items, ',')) AS Number_of_Items,
        EXTRACT(EPOCH FROM (Order_Completed_Time - Visit_Time)) / 60 AS Preparation_Time
    FROM
        orders
)
SELECT
    Service_Staff,
    Number_of_Items,
    AVG(Preparation_Time) AS "Average_Preparation_Time"
FROM
    OrderMetrics
GROUP BY
    Service_Staff, Number_of_Items
ORDER BY
    Service_Staff, Number_of_Items;
```

**12. Where are the biggest bottlenecks in the customer journey?**

  * *Purpose: To break down the total customer experience into stages (Wait -\> Prep -\> Dwell) to identify areas for improvement.*

<!-- end list -->

```sql
SELECT
    AVG(Wait_Time_Minutes) AS "Avg_Initial_Wait_Time",
    AVG(EXTRACT(EPOCH FROM (Order_Completed_Time - Visit_Time)) / 60) - AVG(Wait_Time_Minutes) AS "Avg_Preparation_Time",
    AVG(EXTRACT(EPOCH FROM (Exit_Time - Order_Completed_Time)) / 60) AS "Avg_Post_Meal_Dwell_Time"
FROM
    orders
WHERE
    Order_Type = 'Dine-in';
```

**13. How do menu preferences change throughout the day?**

  * *Purpose: To tailor daily specials and optimize kitchen prep-work.*

<!-- end list -->

```sql
WITH ItemSalesByTime AS (
  SELECT
    TRIM(UNNEST(STRING_TO_ARRAY(Order_Items, ','))) AS Menu_Item,
    CASE
      WHEN EXTRACT(HOUR FROM Visit_Time) BETWEEN 7 AND 11 THEN 'Morning'
      WHEN EXTRACT(HOUR FROM Visit_Time) BETWEEN 12 AND 16 THEN 'Afternoon'
      WHEN EXTRACT(HOUR FROM Visit_Time) BETWEEN 17 AND 21 THEN 'Evening'
      ELSE 'Night'
    END AS Time_Slot
  FROM orders
),
RankedItems AS (
  SELECT
    Time_Slot,
    Menu_Item,
    COUNT(*) as Order_Count,
    ROW_NUMBER() OVER(PARTITION BY Time_Slot ORDER BY COUNT(*) DESC) as Rank
  FROM ItemSalesByTime
  GROUP BY Time_Slot, Menu_Item
)
SELECT
  Time_Slot,
  Menu_Item,
  Order_Count
FROM RankedItems
WHERE Rank <= 2;
```

**14. Which staff member is the most consistent in their service time?**

  * *Purpose: To measure service predictability and reliability, using standard deviation.*

<!-- end list -->

```sql
SELECT
    Service_Staff,
    AVG(EXTRACT(EPOCH FROM (Order_Completed_Time - Visit_Time)) / 60) AS "Average_Service_Time",
    STDDEV(EXTRACT(EPOCH FROM (Order_Completed_Time - Visit_Time)) / 60) AS "Service_Time_Standard_Deviation"
FROM
    orders
GROUP BY
    Service_Staff
ORDER BY
    "Service_Time_Standard_Deviation" ASC;
```
