###  Group by
1 Number of contacts in each customer  

		SELECT c1.cust_id, c1.cust_name, count(c1.cust_contact)
		FROM customers c1    
		GROUP BY c1.cust_name;
 

2 Find the customers that have more than one contact  
(! Order of queries: SELECT-FROM-(WHERE)-GROUP BY-HAVING)

		SELECT c1.cust_id, c1.cust_name, count(c1.cust_contact) contact_number
		FROM customers c1
		GROUP BY c1.cust_name
		HAVING contact_number >1;

###  Self-join and sub-query  
  I think in some situations these two approaches have the same results.
3 Find the customers that have more than one contact and list the contact name  
**Solution 1**: using sub-query

    	SELECT c1.cust_id, c1.cust_name, c1.cust_contact  
    	FROM customers c1
    	WHERE c1.cust_name = ( SELECT c1.cust_name
                          	FROM customers c1
                           	GROUP BY c1.cust_name
				HAVING count(c1.cust_contact) > 1); 

**Solution 2**: using self-join  

    	SELECT c1.cust_id, c1.cust_name, c1.cust_contact
	FROM customers c1, customers c2
    	WHERE c1.cust_name = c2.cust_name and c1.cust_contact != c2.cust_contact;

### LIMIT, OFFSET  
These two queries are availale just in **MySQL**! It's TOP in SQL server/Assess.  
4 The quer below means **return 1 row from the "first" row**  

	SELECT prod_name
	FROM products
	LIMIT 1 OFFSET 0;

5 The quer below means **return 1 row from the second row**  

	SELECT prod_name
	FROM products
	LIMIT 1 OFFSET 1;

### ORDER BY
6 Return all culumns and rank the order and item price in descending order.  
(! the position of DESC)  

	SELECT *
	FROM orderitems
	ORDER BY order_num DESC, item_price DESC;

### WHERE  
7 Return product name and price which price is between 3 and 5.  
(! BETWEEN 3 AND 5 = [3,5])  

	SELECT prod_name, prod_price
	FROM products
	WHERE prod_price BETWEEN 3 AND 5.99;

8 <> and !=  

	SELECT prod_name, vend_id
	FROM products
	WHERE vend_id <> 'brs01'; 
	
I found in MySQL it's case insensitive! So WHERE vend_id <> 'brs01' = WHERE vend_id <> 'BRS01'.  
But, what if there are BRS01 and brs01 at the same time and I want to only exclude brs01 and keep BRS01?  
Here is one of the solutions!~ Thanks for the Internet and search engine:)  

	SELECT prod_name, vend_id
	FROM products
	WHERE **BINARY** vend_id <> 'brs01'; 
	
9 IS NULL and IS NOT NULL  

	SELECT *
	FROM vendors
	WHERE vend_state IS NOT NULL;
	
#### About the date  
10 Get the year/month/day from the date column: year(), month(), day()  

	SELECT *
	FROM orders
	WHERE day(order_date) BETWEEN 1 AND 5;` -- from Monday to Friday

11 Return the orders which were created on workday  

	SELECT *
	FROM orders
	WHERE dayname(order_date) BETWEEN 1 AND 5;

12 Return the order that order date is 2012-05-01  
(the date format does not have to be the same with table, but the order is the same)  

	SELECT *
	FROM orders
	WHERE date(order_date) = '2012/5/1';
	
13 yearweek = year+week; weekofyear = week of year  

	SELECT *
	FROM orders
	WHERE yearweek(order_date) >201204; -- NOT 20124!!!

	SELECT *
	FROM orders
	WHERE weekofyear(order_date) >4;

#### AND, OR, IN, NOT   
14 Priority of and & or: and > or  

	SELECT *
	FROM products
	WHERE vend_id = 'DLL01' OR vend_id = 'BRS01' AND prod_price > 4;

	SELECT *
	FROM products
	WHERE (vend_id = 'DLL01' OR vend_id = 'BRS01' )AND prod_price > 4;

15 IN for string and BETWEEN for number  

	SELECT *
	FROM products
	WHERE vend_id IN ('DLL01','BRS01') AND prod_price > 4;
	
16 NOT: just not the query followed the 'NOT'

	SELECT *
	FROM products
	WHERE NOT vend_id = 'DLL01' AND prod_price > 4;

	SELECT *
	FROM products
	WHERE NOT (vend_id = 'DLL01' AND prod_price > 4);
