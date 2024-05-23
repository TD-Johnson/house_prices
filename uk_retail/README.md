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

retail=# SELECT COUNT(DISTINCT(invoice_no)) FROM retail;

 count 
-------
 25900

The company received 25'900 orders during this period.

### 2. What was the total turnover in this time period?


retail=# SELECT ROUND(SUM(quantity * unit_price)::numeric, 2) AS turnover FROM retail;

   turnover 
-------------
  9747747.93

The total turnover was 9'747'747.93 pounds(GBP).
But was all of this incoming? How much was outgoing.

#### 2.1 How much money was spent during this period?

retail=# SELECT SUM(quantity * unit_price) AS money_lost FROM retail
retail-# WHERE unit_price < 0 OR quantity < 0;

 money_lost 
------------
 -918936.61


9186936 pounds were spent. Almost 1/10th of the total turnover was outgoings. It is worth checking what accounts this is for.

#### 2.2 What customers/things were responsible for the outgoings, and how much was spent on each? Display the top 10.

retail=# SELECT description,  ROUND(SUM(quantity * unit_price)::numeric, 2) AS money_lost FROM retail
WHERE unit_price < 0 OR quantity < 0 
GROUP BY description
ORDER BY money_lost asc
LIMIT (10);

            description             | money_lost 
------------------------------------+------------
 AMAZON FEE                         | -235281.59
 PAPER CRAFT , LITTLE BIRDIE        | -168469.60
 Manual                             | -146784.46
 MEDIUM CERAMIC TOP STORAGE JAR     |  -77479.64
 Adjust bad debt                    |  -22124.12
 POSTAGE                            |  -11871.24
 REGENCY CAKESTAND 3 TIER           |   -9722.55
 CRUK Commission                    |   -7933.43
 Bank Charges                       |   -7340.64
 WHITE HANGING HEART T-LIGHT HOLDER |   -6624.30


So, amazon fees were by far the most expensive.


#### 2.3 On the contrary, what customers were resonsible for the most income?

retail=# SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
WHERE quantity > 0 AND unit_price > 0
GROUP BY customer_id
ORDER BY income desc
LIMIT(10);

 customer_id |   income 
-------------+------------
             | 1755276.64
       14646 |  280206.02
       18102 |  259657.30
       17450 |  194550.79
       16446 |  168472.50
       14911 |  143825.06
       12415 |  124914.53
       14156 |  117379.63
       17511 |   91062.38
       16029 |   81024.84



Interestingly, the main source of income does not have a customer id. It would be useful to find out who, or what, this is.

retail=# SELECT description, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NULL
GROUP BY description
ORDER BY income desc
LIMIT(10);

            description            |  income   
-----------------------------------+-----------
 DOTCOM POSTAGE                    | 194342.41
 REGENCY CAKESTAND 3 TIER          |  31891.79
 PARTY BUNTING                     |  30660.00
 Manual                            |  24332.89
 PAPER CHAIN KIT 50'S CHRISTMAS    |  22291.46
 CHARLOTTE BAG SUKI DESIGN         |  22139.30
 HOT WATER BOTTLE TEA AND SYMPATHY |  17587.37
 RABBIT NIGHT LIGHT                |  15618.79
 AMAZON FEE                        |  13761.09
 RED RETROSPOT CHARLOTTE BAG       |  11650.94
(10 rows)


Ok, so it looks like there is a mix of individual items, postage, and amazon fees. My guess is that these are refunds. 
Lets check how these refunds affect total income. 

#### 2.4 What was total income from customer sales only.

retail=# SELECT ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NOT NULL;

   income
------------
 8911407.90

Ok, so total turnover was around 9.7 million, some of this was outgoings, some was from refunds, but income from sales was 8'911'407.90 pounds.


#### 2.5 And who were the most valuable customers in total revenue?

retail=# SELECT customer_id, ROUND(SUM(quantity * unit_price)::numeric, 2) AS income FROM retail
WHERE quantity > 0 AND unit_price > 0 AND customer_id IS NOT NULL
GROUP BY customer_id
ORDER BY income desc
LIMIT(10);
 customer_id |  income
-------------+-----------
       14646 | 280206.02
       18102 | 259657.30
       17450 | 194550.79
       16446 | 168472.50
       14911 | 143825.06
       12415 | 124914.53
       14156 | 117379.63
       17511 |  91062.38
       16029 |  81024.84
       12346 |  77183.60


 





