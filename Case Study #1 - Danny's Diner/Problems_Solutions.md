# Questions and Answers

#### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT 
  S.CUSTOMER_ID, 
  SUM(M.PRICE) AS TOTAL_SPENT
FROM SALES S INNER JOIN MENU M 
ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY S.CUSTOMER_ID
ORDER BY TOTAL_SPENT DESC
````

CUSTOMER_ID|TOTAL_SPENT|
-----------|-----------|
A          |         76|
B          |         74|
C          |         36|

#### 2. How many days has each customer visited the restaurant?

````sql
SELECT CUSTOMER_ID, COUNT(DISTINCT ORDER_DATE) AS VISITS
FROM SALES
GROUP BY CUSTOMER_ID
````

CUSTOMER_ID|VISITS|
-----------|------|
A          |     4|
B          |     6|
C          |     2|

#### 3. What was the first item from the menu purchased by each customer?
*CAN ALSO USE RANK() TO GET FIRST ITEM IF MORE THAN 1 ITEM PURCHASED*

````sql
WITH FIRST_ITEM AS (
	SELECT
		S.CUSTOMER_ID, 
		S.ORDER_DATE, 
		M.PRODUCT_NAME, 
		ROW_NUMBER() OVER (PARTITION BY S.CUSTOMER_ID ORDER BY ORDER_DATE) AS ORDER_RANK
	FROM SALES S
	INNER JOIN MENU M
	ON S.PRODUCT_ID = M.PRODUCT_ID
)

SELECT
	CUSTOMER_ID, 
	PRODUCT_NAME
FROM FIRST_ITEM
WHERE ORDER_RANK = 1
````

CUSTOMER_ID|PRODUCT_NAME|
-----------|------------|
A          |       curry|
B          |       curry|
C          |       ramen|

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT
	M.PRODUCT_NAME,
	COUNT(S.PRODUCT_ID) AS N_PURCHASED
FROM MENU M
INNER JOIN SALES S ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY M.PRODUCT_NAME
ORDER BY N_PURCHASED DESC
LIMIT 1
````

PRODUCT_NAME|N_PURCHASED|
------------|-----------|
ramen       |          8|

#### 5. Which item was the most popular for each customer?

````sql
WITH MOST_POP AS (
SELECT 
	S.CUSTOMER_ID, 
	M.PRODUCT_NAME,
	RANK() OVER (PARTITION BY S.CUSTOMER_ID ORDER BY COUNT(M.PRODUCT_ID) DESC) AS RMOST_POP
FROM SALES S
INNER JOIN MENU M ON S.PRODUCT_ID = M.PRODUCT_ID
GROUP BY S.CUSTOMER_ID, M.PRODUCT_NAME
)

SELECT * FROM MOST_POP 
WHERE RMOST_POP = 1
````

*Customer ID: B, order all 3 menu items the same amount of times*
CUSTOMER_ID|PRODUCT_NAME|RMOST_POP
-----------|------------|---------
A          |       ramen|1
B          |       sushi|1
B          |       curry|1
B          |       ramen|1
C          |       ramen|1

#### 6: Which item was purchased first by the customer after they became a member?

````sql
WITH FIRST_MEM_ITEM AS (
SELECT
	M.CUSTOMER_ID,
	ME.PRODUCT_NAME,
	S.ORDER_DATE,
	RANK() OVER (PARTITION BY M.CUSTOMER_ID ORDER BY S.ORDER_DATE) AS RNK
FROM MEMBERS M 
INNER JOIN SALES S ON S.CUSTOMER_ID = M.CUSTOMER_ID
INNER JOIN MENU ME ON ME.PRODUCT_ID = S.PRODUCT_ID
WHERE S.ORDER_DATE >= M.JOIN_DATE
)

SELECT CUSTOMER_ID, PRODUCT_NAME
FROM FIRST_MEM_ITEM
WHERE RNK = 1
````


CUSTOMER_ID|PRODUCT_NAME|
-----------|------------|
A          |       curry|
B          |       sushi|

#### 7: Which item was purchased just before the customer became a member?

````sql
WITH FIRST_NONMEM_ITEM AS (
SELECT
	M.CUSTOMER_ID,
	ME.PRODUCT_NAME,
	S.ORDER_DATE,
	RANK() OVER (PARTITION BY M.CUSTOMER_ID ORDER BY S.ORDER_DATE DESC) AS RNK
FROM MEMBERS M 
INNER JOIN SALES S ON S.CUSTOMER_ID = M.CUSTOMER_ID
INNER JOIN MENU ME ON ME.PRODUCT_ID = S.PRODUCT_ID
WHERE S.ORDER_DATE < M.JOIN_DATE
)

SELECT CUSTOMER_ID, PRODUCT_NAME
FROM FIRST_NONMEM_ITEM
WHERE RNK = 1
````

CUSTOMER_ID|PRODUCT_NAME|
-----------|------------|
A          |       sushi|
A          |       curry|
B          |       sushi|

#### 8: What is the total items and amount spent for each member before they became a member?

````sql
SELECT 
	S.CUSTOMER_ID, 
	COUNT(ME.PRODUCT_ID) AS TOTAL_ITEMS, 
	SUM(ME.PRICE) AS AMT_SPENT
FROM MEMBERS M 
INNER JOIN SALES S ON S.CUSTOMER_ID = M.CUSTOMER_ID
INNER JOIN MENU ME ON ME.PRODUCT_ID = S.PRODUCT_ID
WHERE S.ORDER_DATE < M.JOIN_DATE
GROUP BY S.CUSTOMER_ID
````

CUSTOMER_ID|TOTAL_ITEMS|AMT_SPENT
-----------|-----------|---------
B          |          3|       40
A          |          2|       25

#### 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

````sql
SELECT 
	S.CUSTOMER_ID,
	SUM(CASE 
		WHEN S.PRODUCT_ID = 1 THEN PRICE * 20
		ELSE PRICE * 10
	END) AS TOTAL_POINTS
FROM MEMBERS M
INNER JOIN SALES S ON S.CUSTOMER_ID = M.CUSTOMER_ID
INNER JOIN MENU ME ON ME.PRODUCT_ID = S.PRODUCT_ID
GROUP BY S.CUSTOMER_ID
ORDER BY TOTAL_POINTS DESC
````

CUSTOMER_ID|TOTAL_POINTS|
-----------|------------|
B          |        940 |
A          |        860 |

#### 10: In the first week after a customer joins the program 
#### (including their join date) they earn 2x points on all items, not just sushi 
#### how many points do customer A and B have at the end of January?

```sql
SELECT 
	S.CUSTOMER_ID,
	SUM(CASE WHEN S.ORDER_DATE < M.JOIN_DATE
			THEN 
				CASE WHEN S.PRODUCT_ID = 1 THEN PRICE * 20
	   				ELSE PRICE * 10
		END
	    WHEN S.ORDER_DATE > (M.JOIN_DATE + 6)
			THEN 
				CASE WHEN S.PRODUCT_ID = 1 THEN PRICE * 20
	   				ELSE PRICE * 10
				END 
		ELSE PRICE * 20
		END)
		AS POINTS
FROM MEMBERS M 
INNER JOIN SALES S ON S.CUSTOMER_ID = M.CUSTOMER_ID
INNER JOIN MENU ME ON ME.PRODUCT_ID = S.PRODUCT_ID
WHERE S.ORDER_DATE <= '2021-01-31'
GROUP BY S.CUSTOMER_ID
````

CUSTOMER_ID|POINTS|
-----------|------|
A          |  1370|
B          |   820|







