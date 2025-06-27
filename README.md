# Restaurant_order_analysis_P3  
## Project Overview  
**Project title** : Restaurant_Order_Analysis  
**Author** : LABHALLA Halima  
**Level** : Beginner    
**database** : `zomato_db`  
**DBMS USED** : PostgreSQL  
**Number of tables** : 2  
   * `menu_items`   
   * `order_details`



![Restaurant](https://cdn.prod.website-files.com/65743fc81ff97582739ae904/658c3ae3e29d9227cf204003_629bc4c5-3147-420c-bac4-f117586aa455_Guide_Image.jpeg)  

**Situation** : i've just been hired as a data Analyst for 'the taste of the world café' . a restaurant that has  divers menu offerings and serves generous portions.  
**Assignment** : the taste of the world café debuted a new menu at the start of the year, i've been asked to dig into the customer data to see which menu items are doing well / not well and what the customers seem to like best.  
**Objectives** : 
1. explore the menu items_table to get an idea of what's on the menu.
2. explore order_details table to get an idea of the data that's been collected.
3. Use both tables to understand how customers are reacting to the new menu.

**Data base creation** : created a data_base named zomato_db.  
**Table Creation** : created tables for : order_details, menu_items.  

**- table structure for table 'order_details**

```sql
--table structure for table 'order_details' 

CREATE TABLE order_details(
order_details_id SMALLINT NOT NULL,
order_id SMALLINT NOT NULL,
order_date DATE,
order_time TIME,
item_id SMALLINT, 
PRIMARY KEY (order_details_id)

);
--table structure for table 'menu_items'

CREATE TABLE menu_items(
menu_item_id SMALLINT ,
item_name VARCHAR(45),
category VARCHAR(45),
price DECIMAL(5,2), 
PRIMARY KEY(menu_item_id)
);
```  
`Objective 1 : explore the menu items_table to get an idea of what's on the menu`     

**- Task1: write a querry to find the total number of items on the menu**:  

```sql
SELECT 
COUNT(*) AS total_items -- coun(*) compte le nombre total de lignes 
FROM menu_items;
```
**-Task 2:  write a querry to find the least expensive item on the menu**:    
```sql
SELECT item_name, price
FROM menu_items
ORDER BY price ASC 
LIMIT 1;
```  
**- Task3: write a querry to find the most expensive item on the menu**:  
```sql
SELECT item_name, price 
FROM menu_items 
ORDER BY price DESC 
LIMIT 1;
```
**- Task4: write a querry to find how many italien dishes are on the manu**:   
```sql
SELECT category, 
COUNT(category)
FROM menu_items
GROUP BY 1 
--4 
SELECT COUNT(*) AS italian_dish_count
FROM menu_items 
WHERE category = 'Italian'
```
**- Task5: write a querry to find the least expensive italian dish on the menu**  
```sql
SELECT 
item_name,
price
FROM 
menu_items 
WHERE 
category = 'Italian'
ORDER BY price ASC 
LIMIT 1
``` 
**- Task6: write a querry to find the most expensive italian dish on the menu**: 
```sql
SELECT 
item_name, 
price 
FROM 
menu_items 
WHERE 
category = 'Italian' 
ORDER BY price DESC
LIMIT 1
``` 
**- Task7: How many dishes are in each category**:  
```sql
SELECT category, 
COUNT(*) AS number_of_dishes
FROM
menu_items
GROUP BY 1;
```
**- Task8: what is the average dish price in each category**:  
SELECT 
category, 
AVG(price) AS avg_price
FROM 
menu_items 
GROUP BY 1  

`OBJECTIVE 2 : EXPLORE THE ORDERS TABLE`    

**- Task1: write a querry to find the date range of the table order_details**:     
```sql  
SELECT 
MIN(order_date) AS  earliest_date, 
MAX(order_date) AS latest_date
FROM 
order_details
``` 
**- Tas2: How many orders were made within this date range**:  
```sql
SELECT COUNT(DISTINCT order_id) AS total_orders 
FROM 
order_details  
WHERE 
order_date BETWEEN 
(SELECT MIN(order_date) FROM order_details) AND 
(SELECT MAX(order_date) FROM order_details);
``` 
**- Task3: How many unique items were ordered within this date range**:   
```sql
SELECT COUNT( DISTINCT item_id) AS unique_items_ordered
FROM 
order_details 
WHERE order_date BETWEEN
(SELECT MIN(order_date) FROM order_details) AND 
( SELECT MAX(order_date) FROM order_details); 
-- How many items were ordered whithin this date range 
SELECT COUNT(*) AS total_items_ordered 
FROM order_details
```
**- Task4: Which order had the most number of items**:   
```sql
SELECT order_id, 
COUNT(item_id) AS total_items
FROM order_details 
GROUP BY order_id 
ORDER BY total_items DESC 
LIMIT 1;
```
**-Task5: How many orders had more than 12 items**:   
```sql
SELECT COUNT(*) AS orders_with_more_than_12_items
FROM(
SELECT order_id, 
COUNT(item_id) 
FROM 
order_details 
GROUP BY 
order_id 
HAVING COUNT(item_id) > 12 
) AS subquery;  
SELECT*FROM order_details;
```
`Objective 3: ANALYZE CUSTOMER BEHAVIOR` 

**- Task1: combine the menu_items table and order_details table into a single table**:  
```sql
SELECT 
o.*, 
m.*
FROM order_details o 
LEFT JOIN menu_items m 
ON m.menu_item_id = o.item_id
```
**- Task2: What were the least ordered items (we should not keep the chicken tacos on our menu)**:  
```sql
SELECT*FROM menu_items
SELECT  
m.item_name,
m.category,
COUNT( o.order_details_id) AS most_ordered_items
FROM order_details o 
JOIN menu_items m 
ON m.menu_item_id = o.item_id
GROUP BY 1, 2
ORDER BY most_ordered_items ASC
LIMIT 2
```
**- Task3: what were the most ordered items (we should keep the hamburger on our menu)**: 
```sql
SELECT  
m.item_name,
m.category,
COUNT( o.order_details_id) AS most_ordered_items
FROM order_details o 
JOIN menu_items m 
ON m.menu_item_id = o.item_id
GROUP BY 1,2
ORDER BY most_ordered_items DESC
LIMIT 2
```
**- Task4: what were the top 5 orders that spent the most money**:    
```sql
-- on va rassembler order id qui sont les memes et leurs item id 
SELECT
  o.order_id,
  SUM(m.price) AS total_spend
FROM order_details o
JOIN menu_items m 
  ON o.item_id = m.menu_item_id 
GROUP BY o.order_id
ORDER BY total_spend DESC 
LIMIT 5;
```

**- Task5: View the details of the highest spend order (we see that the highest spend order it bought a lot of italian items and not so many of the rest that's interesting because even if italian items weren't the most popular thing on the menu, it seems like this highest spend order tends to really like italian food)**: 
```sql
SELECT category, COUNT(item_id)
FROM order_details o
LEFT JOIN menu_items m 
  ON o.item_id = m.menu_item_id 
  where o.order_id = '440'
  GROUP BY category
``` 
**- Task6: view the details of the top  5 highest spend orders**: 
```sql
SELECT order_id, category, COUNT(item_id)
FROM order_details o
LEFT JOIN menu_items m 
ON o.item_id = m.menu_item_id 
where o.order_id IN ( 440,2075,1957,330,2675)
GROUP BY category
```
**Conclusion**  
the insight that we've gathered here is that we should keep these expensive italian dishes on our menu
because people seemed to be ordering them a lot. 
  */

*📌 Main Conclusion*

Through this data exploration project for 'The Taste of the World Café', I was able to analyze customer behavior and the performance of menu items following the launch of a new menu.  

📋 Menu Overview:  
  * 📋 Menu Overview: The menu offers a wide variety of dishes, including multiple categories and Italian cuisine stands out in terms of both variety and pricing.
  * 📈 Ordering Behavior:

**The most ordered item is the hamburger, while chicken tacos were among the least popular**.

A small number of orders (top 5) accounted for the highest spending, with a clear preference for Italian dishes.

*💡 Strategic Recommendations:*

Continue offering and potentially expanding the Italian category, as it plays a significant role in high-value orders.

Consider removing low-performing items like chicken tacos to optimize menu efficiency.

This analysis provides a solid foundation for making data-driven decisions that align the menu with customer preferences and maximize revenue potential.





 
