# MySQL Learning Note 3 Advanced DB Tech

[TOC]

## 1. Chapter 15. Joining Tablies 

**Foreign Key:** A column in one table that contains the primary key values from another table, thus defining the relationships between tables. 

**Benefits of FK:** 

- **Efficient Storage:**Vendor information is never repeated, and so time and space are not wasted.
- **Easier Manipulation:**If vendor information changes, you can update a single record in the `vendors` table. Data in related tables does not change.
- **Greater Scalability:**As no data is repeated, the data used is obviously consistent, making data reporting and manipulation much simpler.

**If data is stored in multiple tables, how can you retrieve that data with a single SELECT statement?**

*The answer is to use a join. Simply put, a join is a mechanism used to associate tables within a SELECT statement (and thus the name join). Using a special syntax, multiple tables can be joined so a single set of output is returned, and the join associates the correct rows in each table on-the-fly.*



### 1.1 Creating a Join 

Creating a join is very simple. You must specify all the tables to be included and how they are related to each other. Look at the following example:



- Input 

  ```mysql
  SELECT vend_name, prod_name, prod_price
  FROM vendors, products
  WHERE venders.vend_id = products.vend_id
  ORDER BY vend_name, prod_name
  ```



### 1.2 Inner Joins

The join you have been using so far is called an *equijoin*—a join based on the testing of equality between two tables. This kind of join is also called an *inner join*. In fact, you may use a slightly different syntax for these joins, specifying the type of join explicitly. The following `SELECT` statement returns the exact same data as the preceding example:

- Input 

  ```mysql
  SELECT vend_name, prod_name, prod_price
  FROM vendors INNER JOIN products
   ON vendors.vend_id = products.vend_id;
  ```



### 1.3 Joining Multiple Tables

SQL imposes no limit to the number of tables that may be joined in a `SELECT` statement. The basic rules for creating the join remain the same. First list all the tables, and then define the relationship between each. Here is an example:

- Input

  ```mysql
  SELECT prod_name, vend_name, prod_price, quantity
  FROM orderitems, products, vendors
  WHERE products.vend_id = vendors.vend_id  
  			AND orderitems.prod_id = products.prod_id  
  			AND order_num = 20005;
  ```

> **Caution**
>
> **Performance Considerations**. MySQL processes joins at run-time, relating each table as specified. This process can become very resource intensive, so be careful not to join tables unnecessarily. The more tables you join, the more performance degrades.



## Chapter 16. Creating Advanced Joins 

### 2.1 Using Table Aliases

- Why table alias:

  - To shorten the SQL syntax
  - To enable multiple uses of hte same table within a single SELECT statement. 

- Example: 

  - ```mysql
    SELECT cust_name, cust_contact
    FROM customers AS c, orders AS o, orderitems AS oi
    WHERE c.cust_id = o.cust_id
      AND oi.order_num = o.order_num
      AND prod_id = 'TNT2';
    ```



### 2.2 Using Diofferent Join Types 

#### 2.2.1 Self Joins

Suppose that a problem was found with a product (item id `DTNTR`), and you therefore wanted to know all of the products made by the same vendor so as to determine if the problem applied to them, too. This query requires that you first find out which vendor creates item `DTNTR`, and next find which other products are made by the same vendor. The following is one way to approach this problem:

- Solve with a self join

  - ```mysql
    SELECT p1.prod_id, p1.prod_name
    FROM products AS p1, products AS p2
    WHERE p1.vend_id = p2.vend_id
    AND p2.vend_id = 'DTNTR';
    ```

- > ### TIP
  >
  > **Self Joins Instead of Subqueries.** Self joins are often used to replace statements using subqueries that retrieve data from the same table as the outer statement. Although the end result is the same, sometimes these joins execute far more quickly than they do subqueries. It is usually worth experimenting with both to determine which performs better.



#### 2.2.2 Natural Joins 

Whenever tables are joined, at least one column appears in more than one table (the columns being joined). Standard joins (the inner joins you learned about in the previous chapter) return all data, even multiple occurrences of the same column. A *natural join* simply eliminates those multiple occurrences so only one of each column is returned.



#### 2.2.3 Outer Joins 

Most joins relate rows in one table with rows in another. But occasionally, you want to include rows that have no related rows. 











































