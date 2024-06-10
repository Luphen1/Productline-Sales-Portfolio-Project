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

-  [Recommendations](#Recommendations)

### Project Overview
The provided dataset is a product line sale, containing 2823 records with 25 columns. The data encompasses various aspects of sales transactions, including ORDERNUMBER, QUANTITYNUMBER, PRICEEACH, ORDERLINENUMBER, SALES, ORDERDATE, STATUS, QTR_ID, MONTH_ID, YEAR_ID, PRODUCTLINE, MSRP, PRODUCTCODE, CUSTOMERNAME, PHONE, ADDRESSLINE1, ADDRESSLINE2, CITY, STATE, POSTALCODE, COUNTRY, TERRITORY, CONTACTLASTNAME, CONTACTFIRSTNAME, AND DEALSIZE.
I explored a product sales dataset to generate various analytics and insights, identifying trends and patterns. I progressed from basic SQL queries to addressing complex problems using subqueries, CTEs, aggregate functions, and window functions.

### Data Source



### Tools

Microsoft SQL server management studio was utilized for importing dataset and for checking duplicates records. I also utilized SQL server to Identified and handled null values across the dataset.


### Data Cleaning/Preparation

In the initial data preparation, I performed the following tasks below:

1. Data loading and inspection.
2. Handling for missing values.
3. Checking for duplicate values.
4. Handling Inconsistent data and typos.
5. There were null data in columns like ADDRESSLINE2, STATE, AND POSTALCODE in which I left untouched for data quality.




### Exploration Data Analysis
EDA involved exploring product line sales dataset to answer key questions such as:

1.	What was the total number of orders across the dataset?
2.	What was the best month for sales in a specific year? How much was earned that month?
3.	November seems to be the month, what product do they sell in November?
4.	How frequently does customers place orders?
5.	What was the distribution of deal sizes (Small, Medium, Large)?
6.	Which geographic regions generate the most sales?
7.	How does sales vary by territories?
8.	What were the sales trends by quarter?
9.	How does the order status (Shipped, Cancelled, On Hold) sales trends?
10.	Which products generate the most revenue?
11.	Who were the top ten customers in terms of percentage of sales?
12.	What was the change in product lines sales over time?
	




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

WITH total_sales AS (
    SELECT SUM(SALES) AS total_revenue
    FROM [Femi].[dbo].[sales_data]
),
top_ten_customers AS (
    SELECT TOP 10 CUSTOMERNAME, PRODUCTLINE,
           ROUND(SUM(SALES) / (SELECT total_revenue FROM total_sales) * 100, 2) AS revenue_percentage,
           COUNT(QUANTITYORDERED) AS number_of_order,
           MIN(ORDERDATE) AS first_date,
           MAX(ORDERDATE) AS last_order
    FROM [Femi].[dbo].[sales_data]
    GROUP BY CUSTOMERNAME, PRODUCTLINE
    ORDER BY revenue_percentage DESC, number_of_order DESC
)
SELECT *
FROM top_ten_customers;


```








### Results/Findings

1.	There were 2,823 orders made by customers.
2.	The total revenue generated by all orders was $10,032,628.85.
3.	Sales in 2003 generated revenue of $1,029,837.66 with 158 orders. Sales in 2004 generated revenue of $1,089,048.01 with 301 orders. Sales in 2005 generated revenue of $457,861.06 with 120 orders.
4.	November appears to be the month with the highest revenue. Based on my analysis, Classic Cars were sold the most in November.
5.	Australia, New South Wales, and North Sydney generated the highest revenue of $153,996.13 with 46 orders. Canada, British Columbia, and Vancouver generated revenue of $75,238.92 with 22 orders. The USA, California, and San Rafael generated revenue of $654,858.06 with 180 orders.
6.	EMEA (Europe, Middle East, and Africa) generated revenue of $4,979,272.41 with 1,407 orders. APAC (Asia Pacific) generated revenue of $746,121.83 with 345 orders. Japan generated revenue of $455,173.22 with 121 orders. NA (North America) generated revenue of $3,852,061.39 with 1,074 orders.
7.	The fourth quarter generated the highest revenue of $3,874,780.01 with 1,094 orders. The third quarter generated revenue of $1,758,910.81 with 503 orders. The second quarter generated revenue of $2,048,120.30 with 561 orders. The first quarter generated revenue of $2,350,817.73 with 665 orders.
8.	Shipped orders generated revenue of $9,291,501.08 with 2,617 orders. Disputed orders generated revenue of $72,212.86 with 14 orders. Cancelled orders generated revenue of $194,487.48 with 60 orders. On hold orders generated revenue of $178,979.19 with 44 orders. Resolved orders generated revenue of $150,718.28 with 47 orders. In-process orders generated revenue of $144,729.96 with 41 orders.
9.	Classic Cars generated the highest revenue of $3,919,615.66 with 967 orders and an average price of $87.34. Vintage Cars generated revenue of $1,903,150.84 with 607 orders and an average price of $78.15. Motorcycles generated revenue of $1,166,388.34 with 331 orders and an average price of $83.00. Planes generated revenue of $975,003.57 with 306 orders and an average price of $81.74. Trucks and Buses generated revenue of $1,127,789.84 with 301 orders and an average price of $87.53. Ships generated revenue of $714,437.13 with 234 orders and an average price of $83.86. Trains generated revenue of $226,243.47 with 77 orders and an average price of $75.65.
10.	Salzburg Collectables generated revenue of $149,798.63 with 40 orders. Rovelli Gifts generated revenue of $137,955.72 with 48 orders. Online Diecast Creations Co. generated revenue of $131,685.30 with 34 orders. Corrida Auto Replicas, Limited generated revenue of $120,615.28 with 32 orders. Motor Mint Distributors Inc. generated revenue of $83,682.16 with 23 orders. Tekni Collectables Inc. generated revenue of $83,228.19 with 21 orders. Daedalus Designs Imports generated revenue of $69,052.41 with 20 orders. Iberia Gift Imports, Corp. generated revenue of $54,723.62 with 15 orders. Double Decker Gift Stores, Ltd. generated revenue of $36,019.04 with 12 orders. Atelier graphique generated revenue of $24,179.96 with 7 orders.
11.    Customers with top others are Euro Shopping Channel, Mini Gifts Distributors Ltd., Muscle Machine Inc etc.	






### Recommendations

Based on my comprehensive data analysis of orders and revenue across different customers, regions, and product lines, the following strategic recommendations are proposed to enhance and optimize the business:

1.**Focus on High-Performing Product Lines:**
Classic Cars and Vintage Cars have generated the highest revenues. Prioritize marketing and expanding these product lines, as they have demonstrated strong customer demand and profitability. Consider introducing new models or limited editions to capitalize on their popularity.

2.**Seasonal Sales Optimization:**
November has been identified as the month with the highest revenue, particularly for Classic Cars. Implement targeted marketing campaigns and promotions leading up to and during November to maximize sales during this peak period.

3.**Geographic Expansion and Targeting:**
The USA, particularly California, generated the highest revenue with significant order volume. Consider increasing inventory and marketing efforts in high-performing regions like the USA and Australia to further boost sales.
Explore opportunities to enhance market penetration in underperforming but potential high-revenue regions like Japan and APAC.

4.**Customer Retention and Loyalty Programs:**
Top customers, such as Euro Shopping Channel, Mini Gifts Distributors Ltd., and Muscle Machine Inc., contribute significantly to the revenue. Develop loyalty programs and personalized incentives to retain these key customers and encourage repeat purchases.

5.**Enhance Order Fulfillment Efficiency:**
With a high volume of shipped orders generating substantial revenue, streamline the logistics and supply chain processes to ensure timely and efficient order fulfillment. This will improve customer satisfaction and reduce the number of disputed and cancelled orders.

6.**Quarterly Sales Strategies:**
The fourth quarter generated the highest revenue, suggesting strong year-end sales. Design quarterly sales strategies that build momentum towards a strong finish each year. Consider holiday promotions, bundled deals, and early bird specials to drive fourth-quarter sales.

7.**Product Line Diversification:**
While focusing on high-performing lines, also consider diversifying the product portfolio to include emerging trends in the collectibles market. This could attract new customer segments and reduce dependency on a few product lines.

8.**Data-Driven Decision Making:**
Continuously analyze sales data to identify trends, customer preferences, and potential areas for improvement. Leverage advanced analytics and AI tools to forecast demand, optimize inventory, and personalize marketing efforts.

9.**Enhance Online Presence and E-commerce Capabilities:**
Given the significant revenues from key customers, invest in enhancing the online shopping experience. Improve website usability, introduce virtual showrooms, and implement robust customer support to cater to a global audience.

10.**Sustainability and Ethical Practices:**
With increasing consumer awareness about sustainability, consider adopting eco-friendly practices in production and packaging. Highlight these efforts in marketing campaigns to attract environmentally conscious customers.
By implementing these strategic recommendations, the business can leverage the strengths identified in the data analysis, address areas of improvement, and drive sustainable growth in the product line business.


