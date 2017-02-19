###  Self-join, group by and sub-query

1 Number of contacts in each customer  
		`SELECT c1.cust_id, c1.cust_name, count(c1.cust_contact)`    
		`FROM customers c1`    
		`GROUP BY c1.cust_name;`
 

2 Find the customers that have more than one contact  
		`SELECT c1.cust_id, c1.cust_name, count(c1.cust_contact) contact_number`  
		`FROM customers c1`  
		`GROUP BY c1.cust_name`  
    	`HAVING contact_number >1;`  

3 Find the customers that have more than one contact and list the contact name  
**Solution 1**: using sub-query
    	`SELECT c1.cust_id, c1.cust_name, c1.cust_contact`    
    	`FROM customers c1`  
    	`WHERE c1.cust_name = ( SELECT c1.cust_name`  
                          	   `FROM customers c1`  
                           	   `GROUP BY c1.cust_name`  
					       	   `HAVING count(c1.cust_contact) > 1);`  

**Solution 2**: using self-join  
    	`SELECT c1.cust_id, c1.cust_name, c1.cust_contact`  
		`FROM customers c1, customers c2`  
    	`WHERE c1.cust_name = c2.cust_name and c1.cust_contact != c2.cust_contact;`  
