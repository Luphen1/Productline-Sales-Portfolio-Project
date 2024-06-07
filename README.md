# Productline-Sales-Portfolio-Project

#### Table of Contents
---------------------

-  [Project Overview](#Project_Overview)

-  [Data Source](#Data_Source)

-  [Tools](#Tools)

-  [Data Cleaning/Preparation](#Data_Cleaning/Preparation)

-  [Exploration Data Analysis](#Exploration_Data_Analysis)

-  [Data Analysis](#Data_Analysis)

-  [Results/Findings](#Results/Findings)

-  [Limitations](#Limitations)

-  [Recommendations](#Recommendations)

### Project Overview
I explored a product sales dataset to generate various analytics and insights, identifying trends and patterns. I progressed from basic SQL queries to addressing complex problems using subqueries, CTEs, aggregate functions, and window functions.

### Data Source



### Tools


### Data Cleaning/Preparation



### Exploration Data Analysis



### Data Analysis
Below are some interesting codes/features and cleaning I worked with:


```

-- inspecting Dataset  2823 records
SELECT * FROM [Femi].[dbo].[sales_data]

```

```

-- Check for duplicate values. We have no dupliacte values in the dataset
WITH cte as 
(
SELECT *, ROW_NUMBER() 
OVER(PARTITION BY ORDERNUMBER,QUANTITYORDERED,
PRICEEACH,ORDERLINENUMBER,
SALES,ORDERDATE,STATUS,QTR_ID,MONTH_ID,YEAR_ID,PRODUCTLINE,MSRP,
PRODUCTLINE,CUSTOMERNAME,PHONE,ADDRESSLINE1,ADDRESSLINE2,CITY,STATE,POSTALCODE
,COUNTRY,TERRITORY,CONTACTLASTNAME,CONTACTFIRSTNAME,
DEALSIZE ORDER BY ORDERNUMBER)
as duplicate_check
FROM  [Femi].[dbo].[sales_data]
)
SELECT * from cte
WHERE duplicate_check > 1

```

```

--Checking unique values
SELECT DISTINCT STATUS FROM  [Femi].[dbo].[sales_data]
SELECT DISTINCT year_id  FROM  [Femi].[dbo].[sales_data]
SELECT DISTINCT PRODUCTLINE  FROM  [Femi].[dbo].[sales_data]
SELECT DISTINCT COUNTRY  FROM  [Femi].[dbo].[sales_data]
SELECT DISTINCT CITY FROM [Femi].[dbo].[sales_data]
SELECT DISTINCT MONTH_ID FROM [Femi].[dbo].[sales_data]
SELECT DISTINCT QTR_ID FROM [Femi].[dbo].[sales_data]
SELECT DISTINCT DEALSIZE   FROM  [Femi].[dbo].[sales_data]
SELECT DISTINCT TERRITORY FROM [Femi].[dbo].[sales_data] 
SELECT STATE FROM [Femi].[dbo].[sales_data]

```

```
   
-- What was the total number of orders across the dataset? 2823 orders
SELECT 
		COUNT(ORDERDATE) AS total_order
		FROM [Femi].[dbo].[sales_data]

```

```
-- What was the total revenue across the dataset? 
-- Total revenue generated was $10,032,628.85
SELECT 
     FORMAT(SUM(SALES),'C','en-US') AS Revenue
FROM [Femi].[dbo].[sales_data]

```

```

-- What was the best month for sales in a specific year? How much was earned that month? 
-- Sales in 2003 generated $1,029,837.66 and order of 158,2003
-- Sales in 2004 generated $1,089,048.01 and order of 301
-- Sales in 2005 generated $457,861.06 and order of 120
WITH CTE AS (
    SELECT 
        YEAR_ID,
        CASE MONTH_ID
            WHEN 1 THEN 'Jan'
            WHEN 2 THEN 'Feb'
            WHEN 3 THEN 'Mar'
            WHEN 4 THEN 'Apr'
            WHEN 5 THEN 'May'
            WHEN 6 THEN 'Jun'
            WHEN 7 THEN 'Jul'
            WHEN 8 THEN 'Aug'
            WHEN 9 THEN 'Sep'
            WHEN 10 THEN 'Oct'
            WHEN 11 THEN 'Nov'
            WHEN 12 THEN 'Dec'
        END AS Month,
        ROUND(SUM(SALES),2) AS Revenue,
        COUNT(QUANTITYORDERED) AS Order_count
    FROM [Femi].[dbo].[sales_data]
    WHERE YEAR_ID IN (2003, 2004, 2005)
    GROUP BY YEAR_ID, MONTH_ID
)
SELECT*,  
    FORMAT(FIRST_VALUE(Revenue) OVER(PARTITION BY YEAR_ID ORDER BY Revenue DESC), 'C', 'en-US') AS Revenue_rank,
    FIRST_VALUE(Order_count) OVER(PARTITION BY YEAR_ID ORDER BY Order_count DESC) AS Order_count_rank
FROM CTE
ORDER BY YEAR_ID, Revenue DESC, Order_count DESC


```

```

-- November seems to be the month,what product do they sell in November?
-- Classic cars was sold most in the month of november 
SELECT 
MONTH_ID, PRODUCTLINE, FORMAT(SUM(SALES),'C','en-US') AS Revenue, COUNT(ORDERNUMBER) as ordercount
FROM [Femi].[dbo].[sales_data]
WHERE YEAR_ID IN (2003,2004,2005) AND MONTH_ID = 11 --change year to see the rest
GROUP BY MONTH_ID, PRODUCTLINE
ORDER BY 3 DESC

```

```

--- What was the Year-over-Year growth for Each Product Line?
SELECT 
    p1.PRODUCTLINE,
    p1.YEAR_ID,
    p1.Revenue AS SalesCurrentYear,
    p2.Revenue AS SalesPreviousYear,
    (p1.Revenue - p2.Revenue) / p2.Revenue * 100 AS YoYGrowth
FROM (
    SELECT 
    PRODUCTLINE, 
    YEAR_ID, 
    ROUND(SUM(SALES),2) AS Revenue
    FROM [Femi].[dbo].[sales_data]    
    GROUP BY 
        PRODUCTLINE, 
        YEAR_ID
) p1
JOIN (
    SELECT 
    PRODUCTLINE, 
    YEAR_ID, 
    ROUND(SUM(SALES),2) AS Revenue
    FROM 
    [Femi].[dbo].[sales_data] 
    GROUP BY 
        PRODUCTLINE, 
        YEAR_ID
) p2 ON p1.PRODUCTLINE = p2.PRODUCTLINE AND p1.YEAR_ID = p2.YEAR_ID + 1
ORDER BY 
    p1.PRODUCTLINE DESC, 
    p1.YEAR_ID DESC

```

 ```
-- How frequently does customers place orders?

WITH OrderTimes AS (
    SELECT
       CUSTOMERNAME,
	      PRODUCTLINE,
        ORDERDATE,
        LAG(ORDERDATE) OVER (PARTITION BY CUSTOMERNAME ORDER BY ORDERDATE) AS prev_order_date
     FROM [Femi].[dbo].[sales_data]
)
SELECT
   CUSTOMERNAME,
   PRODUCTLINE,
   COUNT(*) AS count_of_orders,
    AVG(DATEDIFF(day, prev_order_date, ORDERDATE)) AS avg_days_between_orders
FROM
    OrderTimes
GROUP BY
   CUSTOMERNAME,PRODUCTLINE
   HAVING COUNT(*) > 1 
ORDER BY 
count_of_orders DESC, avg_days_between_orders DESC


```

```

-- What was the distribution of deal sizes (Small, Medium, Large)?
-- Medium Sizes generated $6,087,432.24 with an order of 1384
-- Small Sizes generated $2,643,077.35 with an order of 1282
-- Large Sizes generated $1,302,119.26 with an order of 157

SELECT 
    DEALSIZE,
    COUNT(*) AS Order_count, 
   FORMAT(SUM(SALES),'C','en-US') AS Revenue
FROM 
  [Femi].[dbo].[sales_data]
GROUP BY 
    DEALSIZE
ORDER BY 
  Order_count DESC,Revenue DESC 

```

```

-- Which geographic regions generate the most sales?
UPDATE [Femi].[dbo].[sales_data]-- Populate the STATE column with the proper state context to make it more readable and easy to grasp
SET STATE = CASE	
			WHEN STATE LIKE 'CA' THEN 'California'
			WHEN STATE LIKE 'PA' THEN 'Pennsyvania'
			WHEN STATE LIKE	'NV' THEN 'Nevada'
			WHEN STATE LIKE	'NY' THEN 'New York'
			WHEN STATE LIKE	'CT' THEN 'Connecticut'
			WHEN STATE LIKE	'NJ' THEN 'New Jersey'
			WHEN STATE LIKE	'NH' THEN 'New Hampshire'
			WHEN STATE LIKE	'MA' THEN 'Massachusetts'
			WHEN STATE LIKE 'BC' THEN 'British Columbia'
			WHEN STATE LIKE 'NSW' THEN 'New South Wales'
			ELSE NULL
			END 

WITH geographic_regions as
(
SELECT  COUNTRY,STATE,CITY,
		FORMAT(SUM(SALES),'C','en-US') AS Revenue,
		COUNT(QUANTITYORDERED) AS Order_Count
		FROM [Femi].[dbo].[sales_data]
		WHERE STATE IS NOT NULL
		GROUP BY COUNTRY,STATE,CITY
) 
		SELECT *,
	    DENSE_RANK() OVER(PARTITION BY COUNTRY ORDER BY Revenue DESC) as Region_Revenue_Rank,
		DENSE_RANK() OVER(PARTITION BY COUNTRY ORDER BY Order_Count DESC) as Region_Order_Count
		FROM geographic_regions

```

```

-- How does sales vary by territory?

-- EMEA(Europe Middle East Africa) generated $4,979,272.41 EMEA and order of 1407.
-- APAC(Asia Pacific) generated $746,121.83 EMEA and order of 1407.
-- Japan generated $455,173.22 with on order of 121.
-- NA(North America) generated $3,852,061.39 with an order of 1074..
SELECT  CASE
			TERRITORY
			WHEN 'EMEA' THEN 'Europe Middle East Africa'
			WHEN 'APAC' THEN 'Asia Pacific'
			WHEN 'NA' THEN  'North America'
			WHEN 'Japan' Then  'Japan'
			END as TERRITORY,
            FORMAT(SUM(SALES),'C','en-US') AS Revenue,
			COUNT(QUANTITYORDERED) AS Order_Count
			FROM  [Femi].[dbo].[sales_data]
			GROUP BY TERRITORY

```

  ```

-- What were the sales trends by quarter?

-- 1st quarter generated $2,350,817.73 with an order of 665
-- 2nd quarter generated $2,048,120.30 with an order of 561
-- 3rd quarter generated $1,758,910.81 with an order of 503
-- 4th quarter generated the highest revenue of $3,874,780.01 with an order of 1094

SELECT QTR_ID,
		FORMAT(SUM(SALES),'C','en-US') AS Revenue,
		COUNT(QUANTITYORDERED) AS Order_count
		FROM [Femi].[dbo].[sales_data]
	    GROUP BY QTR_ID
		ORDER BY Revenue DESC, Order_count DESC

```
		
 ```

-- What is the order status (Shipped, Cancelled, On Hold) sales trends? 

-- Shipped ordered generated $9,291,501.08 with an order of 2617
-- Disputed ordered generated $72,212.86 with an order of  14
-- Cancelled ordered generated $194,487.48 with an order of 60
-- on Hold ordered generated $178,979.19 with an order of 44
-- Resolved ordered generated $150,718.28 with an order of 47
-- In Process ordered generated $144,729.96 with an order of 41
SELECT STATUS,
	   FORMAT(SUM(SALES),'C','en-US') AS Revenue,
	   COUNT(ORDERNUMBER) AS Order_count
	   FROM [Femi].[dbo].[sales_data]	 
	   GROUP BY STATUS
	   ORDER BY  Revenue DESC,Order_count DESC

```

```

--- Which products generate the most revenue?

-- Classic Cars generated the highest revenue of $3,919,615.66 with an order of 967 and Average_Price $87.34
-- Vintage cars generated revenue of $1,903,150.84 with an order of 607 and Average price of $78.15
-- Motorcycles generated revenue of $1,166,388.34  with an order of 331 and Average price of $83.00
-- Planes generated revenue of $975,003.57  with an order of 306 and Average price of $81.74
-- Trucks and Buses generated revenue of $1,127,789.84 with an order of 301 and Average price of $87.53
-- Ships generated revenue of $714,437.13 with an order of 234 and Average price of $83.86
-- Trains generated revenue of $226,243.47 with an order of 77 and Average price of $75.65

SELECT PRODUCTLINE,
       FORMAT(SUM(SALES),'C','en-US') AS Revenue,
	   FORMAT(AVG(PRICEEACH),'C','en-US') AS Average_Price,
	   COUNT(QUANTITYORDERED) AS order_count
	   FROM [Femi].[dbo].[sales_data]
	   GROUP BY PRODUCTLINE
	   ORDER BY Revenue DESC ,Average_Price DESC,order_count DESC

```

```

--- Who were the top 10 customers with highest percentage of sales?
-- Euro Shopping Channel had the highest percentage_revenue of 40.95% and other of 106
WITH top_ten_customers AS
(
SELECT TOP 10 CUSTOMERNAME,PRODUCTLINE,
       ROUND(SUM(SALES) / 100 * 0.01,2) AS revenue_percentage,
	   COUNT(QUANTITYORDERED) AS number_of_order ,
	   MIN(ORDERDATE) AS first_date,
	   MAX(ORDERDATE) AS last_order
	   FROM [Femi].[dbo].[sales_data]
	   GROUP BY CUSTOMERNAME,PRODUCTLINE
	   ORDER BY  revenue_percentage DESC,number_of_order DESC
	   )
	   SELECT *
	   FROM top_ten_customers

```

 ```

--  Find customers with repeating order.What product are they purchasing

WITH repeating_order as 
(SELECT 
		CUSTOMERNAME,
		PRODUCTLINE,
		COUNT(ORDERNUMBER) AS order_count,
		ROUND(SUM(SALES),2) AS revenue,
		MIN(ORDERDATE) AS first_order_purchase,
		MAX(ORDERDATE) AS last_order_purchase
		FROM [Femi].[dbo].[sales_data]
		GROUP BY CUSTOMERNAME,PRODUCTLINE
		HAVING COUNT(*) > 1)		
SELECT  *,
		DENSE_RANK() OVER(PARTITION BY CUSTOMERNAME ORDER BY order_count DESC) AS Order_rank,
		RANK() OVER(PARTITION BY CUSTOMERNAME ORDER BY revenue DESC) AS Revenue_rank
		FROM repeating_order

		
```






### Results/Findings



### Limitations



### Recommendations


