###  Group by
1. Number of contacts in each customer  

		SELECT c1.cust_id, c1.cust_name, count(c1.cust_contact)
		FROM customers c1    
		GROUP BY c1.cust_name;
 

2. Find the customers that have more than one contact  
(! Order of queries: SELECT-FROM-(WHERE)-GROUP BY-HAVING)

		SELECT c1.cust_id, c1.cust_name, count(c1.cust_contact) contact_number
		FROM customers c1
		GROUP BY c1.cust_name
		HAVING contact_number >1;

###  Self-join and sub-query  
  I think in some situations these two approaches have the same results.  
  
3. Find the customers that have more than one contact and list the contact name  
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

4. The quer below means **return 1 row from the "first" row**  

		SELECT prod_name
		FROM products
		LIMIT 1 OFFSET 0;

5. The quer below means **return 1 row from the second row**  

		SELECT prod_name
		FROM products
		LIMIT 1 OFFSET 1;

### ORDER BY
6. Return all culumns and rank the order and item price in descending order.  
(! the position of DESC)  

		SELECT *
		FROM orderitems
		ORDER BY order_num DESC, item_price DESC;

### WHERE  
7. Return product name and price which price is between 3 and 5.  
(! BETWEEN 3 AND 5 = [3,5])  

		SELECT prod_name, prod_price
		FROM products
		WHERE prod_price BETWEEN 3 AND 5.99;

8. <> and !=  

		SELECT prod_name, vend_id
		FROM products
		WHERE vend_id <> 'brs01'; 
	
I found in MySQL it's case insensitive! So WHERE vend_id <> 'brs01' = WHERE vend_id <> 'BRS01'.  
But, what if there are BRS01 and brs01 at the same time and I want to only exclude brs01 and keep BRS01?  
Here is one of the solutions!~ Using **BINARY** Thanks for the Internet and search engine:)  

		SELECT prod_name, vend_id
		FROM products
		WHERE BINARY vend_id <> 'brs01'; 
	
9. IS NULL and IS NOT NULL  

		SELECT *
		FROM vendors
		WHERE vend_state IS NOT NULL;
	
#### About the date  
10. Get the year/month/day from the date column: year(), month(), day()  

		SELECT *
		FROM orders
		WHERE day(order_date) BETWEEN 1 AND 5; -- from Monday to Friday

11. Return the orders which were created on workday  

		SELECT *
		FROM orders
		WHERE dayname(order_date) BETWEEN 1 AND 5;

12. Return the order that order date is 2012-05-01  
(the date format does not have to be the same with table, but the order is the same)  

		SELECT *
		FROM orders
		WHERE date(order_date) = '2012/5/1';
	
13. yearweek = year+week; weekofyear = week of year  

		SELECT *
		FROM orders
		WHERE yearweek(order_date) >201204; -- NOT 20124!!!

		SELECT *
		FROM orders
		WHERE weekofyear(order_date) >4;

#### AND, OR, IN, NOT   
14. Priority of and & or: and > or  

		SELECT *
		FROM products
		WHERE vend_id = 'DLL01' OR vend_id = 'BRS01' AND prod_price > 4;

		SELECT *
		FROM products
		WHERE (vend_id = 'DLL01' OR vend_id = 'BRS01' )AND prod_price > 4;

15. IN for string and BETWEEN for number  

		SELECT *
		FROM products
		WHERE vend_id IN ('DLL01','BRS01') AND prod_price > 4;
	
16. NOT: just not the query followed the 'NOT'

		SELECT *
		FROM products
		WHERE NOT vend_id = 'DLL01' AND prod_price > 4;

		SELECT *
		FROM products
		WHERE NOT (vend_id = 'DLL01' AND prod_price > 4);
### Wildcard (always use with LIKE/REGEXP)
Please find more detail [here]('https://dev.mysql.com/doc/refman/5.6/en/regexp.html#operator_regexp') Documentation is useful!  
**%**: any number of any character, it can also match space

17. Find all bear toy

		SELECT prod_id, prod_name
		FROM products
		WHERE prod_name LIKE '%bear%';  
		
**\_**: only one character (use more \_ if you want to find a specified number of characters, e.g. \_\_\_ means 3 characters)  
N.B. It's **?** in Access :)  

18. Find all xxxx bean bag toy  

		SELECT prod_id, prod_name
		FROM products
		WHERE prod_name LIKE '____ bean bag toy'  
		
19. Find all names containing exactly 8 characters  

		SELECT cust_contact
		FROM customers
		WHERE cust_contact LIKE '_________';

**REGEXP**: it's similar to regular expression in R :)  You can find a regular expression in my another repository~
20. Find toy that the name begins with B or F  

		SELECT prod_id, prod_name
		FROM products
		WHERE prod_name REGEXP '^(B|F)'; -- Do not need to use % 

### Aggregation
**concat**: concatenate columns and also strings(should be in '').   
This is available in MySQL. As far as I know, in other DBMS it's + or ||. If we use these two syntax, MySQL doesn't give error but the result isn't right.

21. Combine vendor name and country in a new column and country in parentheses, give a new alias.  	
		
		SELECT concat(vend_name,'(',vend_country,')') vend_name_country -- Use '' when add string!
		FROM vendors;

**RTRIM/LTRIM/TRIM**: delete unnecessary space.  

22. Delete unnecessary space in vend_state column.

		SELECT TRIM(vend_state)
		FROM vendors;

### Calculation

23. Calculate each item amount in every order.  

		SELECT order_num,
			order_item,
        		quantity,
        		item_price,
        		item_price*quantity item_amount
		FROM orderitems;
		
I just have one thing want every SQL users remember: PLEASE USE **;** AT THE END OF EVERY QUERY!!!!!  
Once I tried "SELECT now()", after that all my queries can't run correctly, then I found it's because I'd forgot to use ";" @-@  

**UPPER/LOWER**: change text into upper/lower case, it's similar in R.  

24. Upper and lower product name in the product table.

		SELECT prod_name, UPPER(prod_name), LOWER(prod_name)
		FROM products;

**LENGTH**: get the string length.  

25. Get the length of vendor state.  

		SELECT vend_state, LENGTH(vend_state) state_text_length
		FROM vendors;

25. I want to find a contact, but I can't remember his name clearly. Jim or Gin.  
But it really depends on the SOUND!!!

		SELECT cust_contact
		FROM customers
		WHERE SOUNDEX(cust_contact) = SOUNDEX('Gin Jone');

**ABS()/COS()/PI()**
We can get pi by using SELECT

		SELECT PI();

### AGGREGATION: AVG()/SUM()/MAX()/MIN()/COUNT()  

**COUNT** 

+ COUNT: for the whole table, count() will return the total row numbers  
26. How many toy vendors do we have?

		SELECT COUNT(*)
		FROM vendors;

+ COUNT: for single column, NULL value are not included  
27. How many vendors does provide their state?  

		SELECT COUNT(vend_state)  
		FROM vendors;  

**AVG()**

28. Calculate average order price.  
		
		SELECT AVG(item_price)
		FROM orderitems;

But I think above eexample is meaningless, so here is another one.  

29. Calculate every order's item average price  

		SELECT order_num,sum(quantity*item_price)/sum(quantity) avg_order_price
		FROM orderitems
		GROUP BY order_num
		ORDER BY avg_order_price;

**MAX()/MIN()**

30. Find the maximum quantity and maximum item price. This query works well, BUT the result isn't from the same row!

		SELECT MAX(quantity),MAX(item_price)
		FROM orderitems;

MAX/MIN can also use for other datatype, not just int/num

		SELECT MIN(cust_name) -- in alphaetical order
		FROM customers;

31. How many product every vendor can provide?

		SELECT v.vend_name, COUNT(p.prod_id)
		FROM vendors v, products p
		WHERE v.vend_id = p.vend_id
		GROUP BY p.vend_id;

		SELECT v.vend_name, COUNT(p.prod_id)
		FROM vendors v INNER JOIN products p
		  ON v.vend_id = p.vend_id
		GROUP BY p.vend_id;

32. How many product every vendor can provide? Those vendors don't provide products should be included.  
N.B.Pls keep an eye on the table where we get the column used in group by when outer join!!!!!!  

The outcomes of below two queries are different.

		SELECT v.vend_name, COUNT(p.prod_id)
		FROM vendors v LEFT OUTER JOIN products p
		  ON v.vend_id = p.vend_id
		GROUP BY v.vend_id;  -- return all vendors

		SELECT v.vend_name, COUNT(p.prod_id)
		FROM vendors v LEFT OUTER JOIN products p
		  ON v.vend_id = p.vend_id
		GROUP BY p.vend_id;  -- if those vendors without products are at the end of the table, they won't be show in the outcome, because they don't have records in products table!
SELECT PI();
