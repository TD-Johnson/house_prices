## A project in which I load data onto a database and perform some analysis of that data using SQL

The data is from a UK online only retailer and contains all the transactions occuring from 01/12/2010 and 09/12/2011.

# Variables Table

| Variable | Name | Role | Type | Description | Units | Missing Values |
| --- | --- | --- | --- | --- | --- | --- | 
InvoiceNo | ID |	Categorical |	a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'c', it indicates a cancellation |     |	no |
StockCode | ID |	Categorical |	a 5-digit integral number uniquely assigned to each distinct product |  | no |
Description |	Feature |	Categorical	| product name |    | 	no |
Quantity |	Feature |	Integer |	the quantities of each product (item) per transaction | | no |
InvoiceDate |	Feature |	 Date |	the day and time when each transaction was generated |  | no |
UnitPrice |	Feature |	Continuous |	product price per unit | sterling |	no |
CustomerID |	Feature |	Categorical |	a 5-digit integral number uniquely assigned to each customer |  | no |
Country |	Feature |	Categorical |	the name of the country where each customer resides |	| no |


The data can be found at: https://archive.ics.uci.edu/dataset/352/online+retail

Credit to Daqing Chen (chend@lsbu.ac.uk), school of engineering, London South Bank University for supplying the data



# Some questions to guide the analysis.

## Maybe lets start with some simple questions.

### 1. How many orders did the company receive in this time period?

SELECT COUNT(DISTINCT(invoice_no)) FROM retail;

|count| 
|---|
 |25900|

The company received 25'900 orders during this period.

### 2. What was the total turnover in this time period?


SELECT ROUND(SUM(quantity * unit_price)::numeric, 2) AS turnover FROM retail;

   |turnover| 
|-------------|
 | 9747747.93 |

The total turnover was 9'747'747.93 pounds(GBP).
But was all of this incoming? How much was outgoing.

#### 2.1 How much money was spent during this period?

SELECT SUM(quantity * unit_price) AS money_lost FROM retail
    WHERE unit_price < 0 OR quantity < 0;

| money_lost |
| ------------ |
| -918936.61 |


9186936 pounds were spent. Almost 1/10th of the total turnover was outgoings. It is worth checking what accounts this is for.

#### 2.2 What customers/things were responsible for the outgoings, and how much was spent on each? Display the top 10.

SELECT description,  ROUND(SUM(quantity * unit_price)::numeric, 2) AS money_lost FROM retail
    WHERE unit_price < 0 OR quantity < 0 
    GROUP BY description
    ORDER BY money_lost ASC
    LIMIT (10);

|            description             | money_lost | 
|------------------------------------|------------ |
| AMAZON FEE                         | -235281.59 |
| PAPER CRAFT , LITTLE BIRDIE        | -168469.60 |
| Manual                             | -146784.46|
| MEDIUM CERAMIC TOP STORAGE JAR     |  -77479.64|
| Adjust bad debt                    |  -22124.12|
| POSTAGE                            |  -11871.24|
| REGENCY CAKESTAND 3 TIER           |   -9722.55|
| CRUK Commission                    |   -7933.43|
| Bank Charges                       |   -7340.64|
| WHITE HANGING HEART T-LIGHT HOLDER |   -6624.30|


So, amazon fees were by far the most expensive.


#### 2.3 On the contrary, what customers were resonsible for the most income?

SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
    WHERE quantity > 0 AND unit_price > 0
    GROUP BY customer_id
    ORDER BY income DESC
    LIMIT(10);

| customer_id |   income |
|-----------|------------|
       |   ?   | 1755276.64|
       |14646 |  280206.02|
       |18102 |  259657.30|
       |17450 |  194550.79|
       |16446 |  168472.50|
       |14911 |  143825.06|
       |12415 |  124914.53|
       |14156 |  117379.63|
       |17511 |   91062.38|
       |16029 |   81024.84|



Interestingly, the main source of income does not have a customer id. It would be useful to find out who, or what, this is.

SELECT description, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
    WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NULL
    GROUP BY description
    ORDER BY income DESC
    LIMIT(10);

 |           description            |  income  | 
|-----------------------------------|-----------|
| DOTCOM POSTAGE                    | 194342.41|
| REGENCY CAKESTAND 3 TIER          |  31891.79|
| PARTY BUNTING                     |  30660.00|
| Manual                            |  24332.89|
| PAPER CHAIN KIT 50'S CHRISTMAS    |  22291.46|
| CHARLOTTE BAG SUKI DESIGN         |  22139.30|
| HOT WATER BOTTLE TEA AND SYMPATHY |  17587.37|
| RABBIT NIGHT LIGHT                |  15618.79|
| AMAZON FEE                        |  13761.09|
| RED RETROSPOT CHARLOTTE BAG       |  11650.94|



Ok, so it looks like there is a mix of individual items, postage, and amazon fees. My guess is that these are refunds. 
Lets check how these refunds affect total income. 

#### 2.4 What was total income from customer sales only.

SELECT ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
    WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NOT NULL;

 |  income|
|------------|
 |8911407.90|

Ok, so total turnover was around 9.7 million, some of this was outgoings, some was from refunds, but income from sales was 8'911'407.90 pounds.


#### 2.5 And who were the most valuable customers in total revenue?

SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
    WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NOT NULL
    GROUP BY customer_id
    ORDER BY income DESC
    LIMIT(10);

| customer_id |  income |
|-----------|-----------|
| 14646 | 280206.02 |
| 18102 | 259657.30 |
| 17450 | 194550.79 |
| 16446 | 168472.50 |
| 14911 | 143825.06 |
| 12415 | 124914.53 |
| 14156 | 117379.63 |
| 17511 |  91062.38 |
| 16029 |  81024.84 |
| 12346 |  77183.60 |

So how many separate orders have these customers placed?

SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income, COUNT(DISTINCT(invoice_no)) AS number_of_orders 
    FROM retail
    WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NOT NULL
    GROUP BY customer_id
    ORDER BY income DESC
    LIMIT(10);

| customer_id |  income   | number_of_orders| 
|-------------|-----------|------------------|
|       14646 | 280206.02 |               73|
|       18102 | 259657.30 |               60|
|       17450 | 194550.79 |               46|
|       16446 | 168472.50 |                2|
|       14911 | 143825.06 |              201|
|       12415 | 124914.53 |               21|
|       14156 | 117379.63 |               55|
|       17511 |  91062.38 |               31|
|       16029 |  81024.84 |               63|
|       12346 |  77183.60 |                1|


So we can see that those that have spent the most money in total, have not necessarily done so due to ordering multiple times. 
In fact, one customer '16446' ordered on two separate occasions, but spent 168'472 pounds in total. Lets have a look at these two orders and see how many things were ordered in each.

SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income, invoice_no, COUNT(description) AS number_of_items
    FROM retail
    WHERE quantity > 0 AND unit_price > 0 AND customer_id = 16446 
    GROUP BY invoice_no, customer_id
    ORDER BY income DESC;

| customer_id |  income   | invoice_no | number_of_items |
|-------------|-----------|------------|-----------------|
|       16446 | 168469.60 | 581483     |               1|
|       16446 |      2.90 | 553573     |               2|



Interesting. Before we move on I should just check that none of this was refunded.

SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income, invoice_no, COUNT(description) AS number_of_items
    FROM retail
    WHERE customer_id = 16446 
    GROUP BY invoice_no, customer_id
    ORDER BY income DESC;

| customer_id |   income   | invoice_no | number_of_items |
|-------------|------------|------------|-----------------|
|       16446 |  168469.60 | 581483     |               1|
|       16446 |       2.90 | 553573     |               2|
|       16446 | -168469.60 | C581484    |               1|

 
Yes, the big order was refunded. Perhaps the order was a mistake. 

### 3. What product generated the most income?

SELECT description, unit_price, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income, COUNT(invoice_no) AS number_of_orders FROM retail 
    WHERE unit_price > 0
    GROUP BY description, unit_price
    ORDER BY income DESC
    LIMIT(10);

|            description             | unit_price |  income  | number_of_orders |
|------------------------------------|------------|----------|------------------|
| REGENCY CAKESTAND 3 TIER           |      10.95 | 85913.70 |              387|
| REGENCY CAKESTAND 3 TIER           |      12.75 | 48399.00 |             1550|
| WHITE HANGING HEART T-LIGHT HOLDER |       2.55 | 44742.30 |              373|
| PICNIC BASKET WICKER 60 PIECES     |      649.5 | 39619.50 |                2|
| ASSORTED COLOUR BIRD ORNAMENT      |       1.69 | 37945.57 |             1395|
| POSTAGE                            |         18 | 35838.00 |              720|
| RABBIT NIGHT LIGHT                 |       1.79 | 35715.87 |              259|
| BLACK RECORD COVER FRAME           |       3.39 | 35150.91 |              122|
| PARTY BUNTING                      |       4.95 | 33759.00 |             1239|
| JUMBO BAG RED RETROSPOT            |       1.79 | 33161.54 |              146|


We can see that the cakestand generated the most income. Oddly it was valued at two different prices, perhaps due to a discount, but either way it was still ordered a lot. 


### 4. What are the monthly sales?

SELECT DATE_TRUNC('month', invoice_date) AS month, ROUND(SUM(unit_price * quantity)::numeric, 2) AS turnover
    FROM retail
    WHERE customer_id IS NOT NULL
    GROUP BY month
    ORDER BY month;


|        month        |  turnover  |
|---------------------|------------|
| 2010-01-01 00:00:00 |   46051.26|
| 2010-02-01 00:00:00 |   45775.43|
| 2010-03-01 00:00:00 |   22598.46|
| 2010-05-01 00:00:00 |   31380.60|
| 2010-06-01 00:00:00 |   30465.08|
| 2010-07-01 00:00:00 |   53125.99|
| 2010-08-01 00:00:00 |   38048.68|
| 2010-09-01 00:00:00 |   37177.85|
| 2010-10-01 00:00:00 |   32005.35|
| 2010-12-01 00:00:00 |  217975.32|
| 2011-01-01 00:00:00 |  510308.10|
| 2011-02-01 00:00:00 |  479089.93|
| 2011-03-01 00:00:00 |  617490.30|
| 2011-04-01 00:00:00 |  556556.96|
| 2011-05-01 00:00:00 |  704359.40|
| 2011-06-01 00:00:00 |  663888.11|
| 2011-07-01 00:00:00 |  706117.57|
| 2011-08-01 00:00:00 |  583211.86|
| 2011-09-01 00:00:00 |  901380.35|
| 2011-10-01 00:00:00 |  820252.47|
| 2011-11-01 00:00:00 | 1004792.27|
| 2011-12-01 00:00:00 |  198014.47|

This is quite hard to tell whats going on. Perhaps we should see the month by month comparison. AKA is the price for the current month better or worse than the previous month.
One thing to notice is that there appears to be no data for november 2010. Perhaps the retailer was closed.

#### 4.1 How does the current months turnover compare to the previous month?

SELECT month, turnover, ROUND((turnover - LAG(turnover) OVER (ORDER BY month))/LAG(turnover) OVER (ORDER BY month) * 100::numeric, 2) AS growth_rate 
FROM (
    SELECT DATE_TRUNC('month', invoice_date) AS month, ROUND(SUM(unit_price * quantity)::numeric, 2) AS turnover
    FROM retail
    WHERE customer_id IS NOT NULL
    GROUP BY month) AS monthly_sales;


|        month        |  turnover  | growth_rate |
|---------------------|------------|-------------|
| 2010-01-01 00:00:00 |   46051.26 | 
| 2010-02-01 00:00:00 |   45775.43 |       -0.60|
| 2010-03-01 00:00:00 |   22598.46 |      -50.63|
| 2010-05-01 00:00:00 |   31380.60 |       38.86|
| 2010-06-01 00:00:00 |   30465.08 |       -2.92|
| 2010-07-01 00:00:00 |   53125.99 |       74.38|
| 2010-08-01 00:00:00 |   38048.68 |      -28.38|
| 2010-09-01 00:00:00 |   37177.85 |       -2.29|
| 2010-10-01 00:00:00 |   32005.35 |      -13.91|
| 2010-12-01 00:00:00 |  217975.32 |      581.06|
| 2011-01-01 00:00:00 |  510308.10 |      134.11|
| 2011-02-01 00:00:00 |  479089.93 |       -6.12|
| 2011-03-01 00:00:00 |  617490.30 |       28.89|
| 2011-04-01 00:00:00 |  556556.96 |       -9.87|
| 2011-05-01 00:00:00 |  704359.40 |       26.56|
| 2011-06-01 00:00:00 |  663888.11 |       -5.75|
| 2011-07-01 00:00:00 |  706117.57 |        6.36|
| 2011-08-01 00:00:00 |  583211.86 |      -17.41|
| 2011-09-01 00:00:00 |  901380.35 |       54.55|
| 2011-10-01 00:00:00 |  820252.47 |       -9.00|
| 2011-11-01 00:00:00 | 1004792.27 |       22.50|
| 2011-12-01 00:00:00 |  198014.47 |      -80.29|


So we can see that there is a bit of growth from october to november, although there is no data for 2010. But what is strikingly obvious is the increase in turnover in december 2010 and then onto january 2011. It then stabilises around 2011, but there is a rather large decrease in december 2011. The obvious reason for this is that we only have 9 days worth of data in december 2011.



