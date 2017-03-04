## JOIN
I think join is one of the most powderful function in SQL. So here I synthesized these different **JOIN**.  
Paying attention to the details in every query is important!!!  

### INNER JOIN
Find the parts that exist in different table.
* Sum of order of each customer, and the customer state?
If we don't need to return, we can search only in table orders,and join is not necessary in that case

		SELECT c.cust_id, c.cust_state, COUNT(o.order_num) sum_order
		FROM customers c INNER JOIN orders o
			ON c.cust_id = o.cust_id
		GROUP BY c.cust_id;

### SELF-JOIN
I think **SELF-JOIN** is "copy a table to join", not join another one, when there are **more than one queries in one table.**
To copy a table, we can use alias.

#### Accumulation: use >= or <= to join  

In this case, at first I couldn't get the result I want, after a sleep, I found a solution!! :)  
That is using sub-query to change the "copied" table a little.  

* Accumulated distribution of product price, e.g. there are 4 products under $4.99, 5 under $5.99.  
At first my queries as below:
        
        	SELECT DISTINCT p1.prod_price, COUNT(p2.prod_price) prod_categories
       		FROM products p1 JOIN products p2
          		ON p1.prod_price >= p2.prod_price
        	GROUP BY p1.prod_price;

Here is the outcome:
It seems that every row was calculated, DISTINCT doesn't work. :(  

![accumulation_original_result] (https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/result_accumulated_distribution.JPG)

I checked the **execution plan** and found that DISTINCT was executed in the last step, I wanted it to be executed at first.    

![accumulation_original_plan] (https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/accumulated%20distribution_original.png)

Now I found the reason why the outcome was like that. Finally, here is the solution: DISTINCT in a subquery before SELECT.

        	SELECT p1.prod_price, COUNT(p2.prod_price) prod_categories
        	FROM products p2, (SELECT DISTINCT(prod_price) FROM products) p1
        	WHERE p1.prod_price >= p2.prod_price
        	GROUP BY p1.prod_price;

Here is the outcome I expect.

![accumulation_after_result] (https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/result_add_sub_query.JPG)

And I checked the execution plan, it's good!~~

![accumulation_after_plan] (https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/accumulated%20distribution_add_sub_query.png)

Execution plan is really helpful this time!!!

#### Different A with the same label B: use != to join  

In this case, for example, different contact from the same company.  
If I don't use != to add a condition, there will be duplicated result.  

* Find the customers that have more than one contact and list the contact name  

**Solution 1**: using self-join  

        	SELECT c1.cust_id, c1.cust_name, c1.cust_contact
        	FROM customers c1, customers c2
        	WHERE c1.cust_name = c2.cust_name and c1.cust_contact != c2.cust_contact;
        
In some situations, there are two or more approaches can be used.  
**Solution 2**: using sub-query

        	SELECT c1.cust_id, c1.cust_name, c1.cust_contact  
        	FROM customers c1
        	WHERE c1.cust_name = 
            	( SELECT c1.cust_name
            	FROM customers c1
            	GROUP BY c1.cust_name
            	HAVING count(c1.cust_contact) > 1);
            
#### Rank : use > or < to join
As far as I know, there is a function called **rank** in some DBSMs like SQL Server, but there isn't in MySQL :(  
So how to rank(not just order, add rank label)?  

I was inspired by this <answer>(http://stackoverflow.com/questions/3333665/rank-function-in-mysql), thanks

* Rank the order item by quantity:
Rank with duplicate:
        
        	SELECT a.order_item,a.prod_id,a.quantity,
		        count(b.quantity)+1 rank
	        FROM orderitems a left join orderitems b
		        on a.quantity > b.quantity
        	GROUP BY a.order_item,a.prod_id,a.quantity
        	ORDER BY rank;

![rank_result_non_distinct](https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/rank_with_duplicated(non-distinct).JPG)

Rank without duplicate
        
	        SELECT a.order_item,a.prod_id,a.quantity,
		        count(distinct b.quantity)+1 rank
	        FROM orderitems a left join orderitems b
		        on a.quantity > b.quantity
        	GROUP BY a.order_item,a.prod_id,a.quantity
        	ORDER BY rank;

![rank_result_distinct] (https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/rank_without_duplicated(distinct).JPG)

* Return the order_item and prod_id in No.3

        	SELECT a.order_item,a.prod_id,a.quantity,
		        count(distinct b.quantity)+1 rank
	        FROM orderitems a left join orderitems b
		        on a.quantity > b.quantity
        	GROUP BY a.order_item,a.prod_id,a.quantity
        	HAVING rank = 3;

### OUTER JOIN
I drew a simple diagram which shows how INNER JOIN, LEFT OUTER JOIN and RIGHT OUTER JOIN work.

![diagram](https://github.com/YutongLiu/SQL-Practice-and-Learning/blob/master/Images/THREE%20TYPES%20OF%20JOIN.JPG)

#### JOIN TWO TABLE
* List all customer id, name, state and their order number.  

		SELECT c.cust_id, c.cust_name, c.cust_state,
			COUNT(o.order_num)
		FROM customers c LEFT JOIN orders o
			ON c.cust_id = o.cust_id
		GROUP BY c.cust_id;
		
#### JOIN MORE THAN TWO TABLES
* List all customer id, name, state and their order number and amount.
Actually it takes a few minutes to answer this question I raised by myself... Anyway I solved this question. Here are two key points:  
  1.**DISTINCT** is necessary in this case, otherwise it returns customer's order# * order_item.
  2.() is necessary and it must be parenthesize INNER JOIN.  

		SELECT c.cust_id, c.cust_name, c.cust_state,
		  COUNT(DISTINCT o.order_num),SUM(oi.quantity*oi.item_price)
		FROM customers c LEFT JOIN (orders o INNER JOIN orderitems oi)
		  ON o.order_num = oi.order_num and c.cust_id = o.cust_id
		GROUP BY c.cust_id;
		
