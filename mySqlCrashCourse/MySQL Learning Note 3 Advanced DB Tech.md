# MySQL Learning Note 3 Advanced DB Tech

[TOC]

## 1. Chapter 15. Joining Tables 

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
  WHERE vendors.vend_id = products.vend_id
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

- Pattern of INNER JOIN

```mysql
SELECT column1, column2, column3 .... 
FROM table1 INNER JOIN table2
ON table1.same_column = table2.same_column;
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



## 2. Chapter 16. Creating Advanced Joins 

### 2.1 Using Table Aliases

- Why table alias:

  - To shorten the SQL syntax
  - To enable multiple uses of the same table within a single SELECT statement. 

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

**The truth is, every inner join you have created thus far is actually a natural join, and you will probably never even need an inner join that is not a natural join.**



#### 2.2.3 Outer Joins 

Most joins relate rows in one table with rows in another. But occasionally, you want to include rows that have no related rows.  For example, you might use joins to accomplish the following tasks:

- Count how many orders each customer placed, including customers who have yet to place an order
- List all products with order quantities, including products not ordered by anyone
- Calculate average sale sizes, taking into account customers who have not yet placed an order

In each of these examples, the join includes table rows that have no associated rows in the related table. This type of join is called an *outer join*.

- Example: Inner Join

```mysql
SELECT customers.cust_id, orders.order_num
FROM customers INNER JOIN orders
 ON customers.cust_id = orders.cust_id;
```

- Inner Join result:

```
+---------+-----------+
| cust_id | order_num |
+---------+-----------+
|   10001 |     20005 |
|   10001 |     20009 |
|   10003 |     20006 |
|   10004 |     20007 |
|   10005 |     20008 |
+---------+-----------+
```

- Example: Left Outter Join

```mysql
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
 ON customers.cust_id = orders.cust_id;
```

- Left Outter Join result: 

```
+---------+-----------+
| cust_id | order_num |
+---------+-----------+
|   10001 |     20005 |
|   10001 |     20009 |
|   10002 |      NULL |
|   10003 |     20006 |
|   10004 |     20007 |
|   10005 |     20008 |
+---------+-----------+
```



### 2.3 Using Joins with Aggregate Functions

You want to retrieve a list of all customers and the number of orders that each has placed. The following code uses the `COUNT()` function to achieve this:

- Code

```mysql
SELECT customers.cust_name,
       customers.cust_id,
       COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders
 ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```

- Result

```
+----------------+---------+---------+
| cust_name      | cust_id | num_ord |
+----------------+---------+---------+
| Coyote Inc.    |   10001 |       2 |
| Mouse House    |   10002 |       0 |
| Wascals        |   10003 |       1 |
| Yosemite Place |   10004 |       1 |
| E Fudd         |   10005 |       1 |
+----------------+---------+---------+
```



## 3. Chapter 17. Combining Queries

### 3.1 Creating Combined Queries 

SQL queries are combined using the `UNION` operator. Using `UNION`, multiple `SELECT` statements can be specified, and their results can be combined into a single result set.

#### 3.1.1 Using `UNION`

Using `UNION` is simple enough. All you do is specify each `SELECT` statement and place the keyword `UNION` between each.

Let’s look at an example. You need a list of all products costing `5` or less. You also want to include all products made by vendors `1001` and `1002`, regardless of price.

- Using two separate SQL select: Select all products that have price `<=` 5

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5;
```

- Uing two separate SQL select: Select all products made by vendor `1001` and `1002`

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```

- Using `UNION`

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```

#### 3.1.2 Using `OR`

As a point of reference, here is the same query using multiple `WHERE` clauses instead of a `UNION`:

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
  OR vend_id IN (1001,1002);
```



### 3.2 `UION` Rules

As you can see, unions are very easy to use. But a few rules govern exactly which can be combined:

- A `UNION` must be comprised of two or more `SELECT` statements, each separated by the keyword `UNION` (so, if combining four `SELECT` statements, three `UNION` keywords would be used).
- Each query in a `UNION` must contain the same columns, expressions, or aggregate functions (although columns need not be listed in the same order).
- Column datatypes must be compatible: They need not be the exact same type, but they must be of a type that MySQL can implicitly convert (for example, different numeric types or different date types).

Aside from these basic rules and restrictions, unions can be used for any data retrieval tasks.



### 3.3 Including or Eliminating Duplicate Rows `UNION ALL`

Go back to the preceding section titled “[Using `UNION`](https://learning.oreilly.com/library/view/mysql-crash-course/0672327120/ch17.html#ch17lev2sec1)” and look at the sample `SELECT` statements used. You’ll notice that when executed individually, the first `SELECT` statement returns four rows, and the second `SELECT` statement returns five rows. However, when the two `SELECT` statements are combined with a `UNION`, only eight rows are returned, not nine.

The `UNION` automatically removes any duplicate rows from the query result set (in other words, it behaves just as multiple `WHERE` clause conditions in a single `SELECT` would). Because vendor `1002` creates a product that costs less than `5`, that row was returned by both `SELECT` statements. When the `UNION` was used, the duplicate row was eliminated.

This is the default behavior of `UNION`, but you can change this if you so desire. If you do, in fact, want all occurrences of all matches returned, you can use `UNION ALL` instead of `UNION`.

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION ALL
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```



### 3.4 Sorting Combined Query Results

`SELECT` statement output is sorted using the `ORDER BY` clause. When combining queries with a `UNION`, only one `ORDER BY` clause may be used, and it must occur after the final `SELECT` statement. There is very little point in sorting part of a result set one way and part another way, and so multiple `ORDER BY` clauses are not allowed.

- Example 

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002)
ORDER BY vend_id, prod_price;
```



























