# MySQL Learning Note 3 Advanced DB Tech

[TOC]

## 1. Chapter 15. Joining Tables

**Foreign Key:** A column in one table that contains the primary key values from another table, thus defining the relationships between tables.

**Benefits of FK:**

- **Efficient Storage:**Vendor information is never repeated, and so time and space are not wasted.
- **Easier Manipulation:**If vendor information changes, you can update a single record in the `vendors` table. Data in related tables does not change.
- **Greater Scalability:**As no data is repeated, the data used is obviously consistent, making data reporting and manipulation much simpler.

**If data is stored in multiple tables, how can you retrieve that data with a single SELECT statement?**

_The answer is to use a join. Simply put, a join is a mechanism used to associate tables within a SELECT statement (and thus the name join). Using a special syntax, multiple tables can be joined so a single set of output is returned, and the join associates the correct rows in each table on-the-fly._

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

The join you have been using so far is called an _equijoin_—a join based on the testing of equality between two tables. This kind of join is also called an _inner join_. In fact, you may use a slightly different syntax for these joins, specifying the type of join explicitly. The following `SELECT` statement returns the exact same data as the preceding example:

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

Whenever tables are joined, at least one column appears in more than one table (the columns being joined). Standard joins (the inner joins you learned about in the previous chapter) return all data, even multiple occurrences of the same column. A _natural join_ simply eliminates those multiple occurrences so only one of each column is returned.

**The truth is, every inner join you have created thus far is actually a natural join, and you will probably never even need an inner join that is not a natural join.**

#### 2.2.3 Outer Joins

Most joins relate rows in one table with rows in another. But occasionally, you want to include rows that have no related rows. For example, you might use joins to accomplish the following tasks:

- Count how many orders each customer placed, including customers who have yet to place an order
- List all products with order quantities, including products not ordered by anyone
- Calculate average sale sizes, taking into account customers who have not yet placed an order

In each of these examples, the join includes table rows that have no associated rows in the related table. This type of join is called an _outer join_.

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

## 4. Chapter 19. Inserting Data

`INSERT` is used to insert (add) rows to a database table. `INSERT` can be used in several ways:

- To insert a single complete row
- To insert a single partial row
- To insert multiple rows
- To insert the results of a query

### 4.1 Inserting Complete Rows

- Example

```mysql
INSERT INTO Customers
VALUES(NULL,
   'Pep E. LaPew',
   '100 Main Street',
   'Los Angeles',
   'CA',
   '90046',
   'USA',
   NULL,
   NULL);
```

> The first column, `cust_id`, is also `NULL`. This is because that column is automatically incremented by MySQL each time a row is inserted. You’d not want to specify a value (that is MySQL’s job), and nor could you omit the column (as already stated, every column must be listed), and so a `NULL` value is specified (it is ignored by MySQL, which inserts the next available `cust_id` value in its place).

- To make the insert not depend on the order of columns, you need to specify the column name:

```mysql
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country,
   cust_contact,
   cust_email)
VALUES('Pep E. LaPew',
   '100 Main Street',
   'Los Angeles',
   'CA',
   '90046',
   'USA',
   NULL,
   NULL);
```

> ### TIP
>
> **Always Use a Columns List.** As a rule, never use `INSERT` without explicitly specifying the column list. This will greatly increase the probability that your SQL will continue to function in the event that table changes occur.

> ### CAUTION
>
> **Omitting Columns.** You may omit columns from an `INSERT` operation if the tabledefinition so allows. One of the following conditions must exist:
>
> - The column is defined as allowing `NULL` values (no value at all).
> - A default value is specified in the table definition. This means the default value will be used if no value is specified.
>
> If you omit a value from a table that does not allow `NULL` values and does not have a default, MySQL generates an error message, and the row is not inserted.

> ### TIP
>
> **Improving Overall Performance.** Databases are frequently accessed by multiple clients, and it is MySQL’s job to manage which requests are processed and in which order. `INSERT` operations can be time consuming (especially if there are many indexes to be updated), and this can hurt the performance of `SELECT` statements that are waiting to be processed.
>
> If data retrieval is of utmost importance (as it usually is), you can instruct MySQL to lower the priority of your `INSERT` statement by adding the keyword `LOW_PRIORITY` in between `INSERT` and `INTO`, like this:
>
> ```
> INSERT LOW_PRIORITY INTO
> ```
>
> Incidentally, this also applies to the `UPDATE` and `DELETE` statements that you’ll learn about in the next chapter.

### 4.2 Inserting Multiple Rows

- Insert with multiple `INSERT` statements and separate with `;`.

```mysql
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES('Pep E. LaPew',
   '100 Main Street',
   'Los Angeles',
   'CA',
   '90046',
   'USA');
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES('M. Martian',
   '42 Galaxy Way',
   'New York',
   'NY',
   '11213',
   'USA');
```

- Insert with one `INSERT` statement

```mysql
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES(
        'Pep E. LaPew',
        '100 Main Street',
        'Los Angeles',
        'CA',
        '90046',
        'USA'
     ),
      (
        'M. Martian',
        '42 Galaxy Way',
        'New York',
        'NY',
        '11213',
        'USA'
   );
```

> ### TIP
>
> **Improving INSERT Performance.** This technique can improve the performance of your database processing, as MySQL will process multiple insertions in a single `INSERT` faster than it will multiple `INSERT` statements.

### 4.3 Inserting Retrieved Data

Suppose you want to merge a list of customers from another table into your `customers` table. Instead of reading one row at a time and inserting it with `INSERT`, you can do the following:

```mysql
INSERT INTO customers(cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
SELECT cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country
FROM custnew;
```

The `SELECT` statement used in an `INSERT SELECT` can include a `WHERE` clause to filter the data to be inserted.

[Learn more about copy table](https://tableplus.com/blog/2018/11/how-to-duplicate-a-table-in-mysql.html)

## 5. Chapter 20. Updating and Deleting Data

### 5.1 Updating Data

To update (modify) data in a table the `UPDATE` statement is used. `UPDATE` can be used in two ways:

- To update specific rows in a table
- To update all rows in a table

Example to update **one column** in a table:

```mysql
UPDATE customers
SET cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```

Example to update **multiple columns** in a table:

```mysql
UPDATE customers
SET cust_name = 'The Fudds'
    cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```

How to handle errors in updating:

> ### TIP
>
> **The IGNORE Keyword.** If your `UPDATE` statement updates multiple rows and an error occurs while updating one or more of those rows, the entire `UPDATE` operation is cancelled (and any rows updated before the error occurred are restored to their original values). To continue processing updates, even if an error occurs, use the `IGNORE` keyword, like this:
>
> ```mysql
> UPDATE IGNORE customers ...
> ```

Delete a column's value, you can set it to `NULL` (assuming the table is defined to allow `NULL` values). You can do this as follows:

```mysql
UPDATE customers
SET cust_email = NULL
WHERE cust_id = 10005;
```

### 5.2 Deleting Data

> ### CAUTION
>
> **Don’t Omit the WHERE Clause.** Special care must be exercised when using `DELETE` because it is all too easy to mistakenly delete every row from your table.

To delete (remove) data from a table, the `DELETE` statement is used. `DELETE` can be used in two ways:

- To delete specific rows from a table
- To delete all rows from a table

Example to delete a **single row** :

```mysql
DELETE FROM customers
WHERE cust_id = 10006;
```

To delete all rows in a table:

> ### NOTE
>
> **Table Contents, Not Tables.** The `DELETE` statement deletes rows from tables, even all rows from tables. But `DELETE` never deletes the table itself.

> ### TIP
>
> **Faster Deletes.** If you really do want to delete all rows from a table, don’t use `DELETE`. Instead, use the `TRUNCATE TABLE` statement that accomplished the same thing but does it much quicker (`TRUNCATE` actually drops and recreates the table, instead of deleting each row individually).

### 5.3 Guidelines for Updating and Deleting Data

Here are some best practices that many SQL programmers follow:

- Never execute an `UPDATE` or a `DELETE` without a `WHERE` clause unless you really do intend to update and delete every row.
- Make sure every table has a primary key (refer to [Chapter 15](https://learning.oreilly.com/library/view/mysql-crash-course/0672327120/ch15.html), “Joining Tables,” if you have forgotten what this is), and use it as the `WHERE` clause whenever possible. (You may specify individual primary keys, multiple values, or value ranges.)
- Before you use a `WHERE` clause with an `UPDATE` or a `DELETE`, first test it with a `SELECT` to make sure it is filtering the right records—it is far too easy to write incorrect `WHERE`clauses.
- Use database enforced referential integrity (refer to [Chapter 15](https://learning.oreilly.com/library/view/mysql-crash-course/0672327120/ch15.html) for this one, too) so MySQL will not allow the deletion of rows that have data in other tables related to them.

## 6. Chapter 21. Creating and Manipulating Tables

### 6.1 Basic Table Creation

- Example

```mysql
CREATE TABLE customers (
  cust_id					int 			NOT NULL AUTO_INCREMENT,
  cust_name				char(5)		NOT NULL,
  cust_address		char(50)	NULL ,
  cust_city				char(50)	NULL ,
  cust_state   char(5)   NULL ,
  cust_zip     char(10)  NULL ,
  cust_country char(50)  NULL ,
  cust_contact char(50)  NULL ,
  cust_email   char(255) NULL ,
  PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
```

> ### TIP
>
> **Handling Existing Tables.** When you create a new table, the table name specified must not exist or you’ll generate an error. To prevent accidental overwriting, SQL requires that you first manually remove a table (see later sections for details) and then re-create it, rather than just overwriting it.
>
> If you want to create a table only if it does not already exist, specify `IF NOT EXISTS` after the table name. This does not check to see that the schema of the existing table matches the one you are about to create. It simply checks to see if the table name exists, and only proceeds with table creation if it does not.

### 6.2 Working with Primary Key

- Example of a primary key made up of multiple columns:

```mysql
CREATE TABLE orderitems
(
  order_num  int          NOT NULL ,
  order_item int          NOT NULL ,
  prod_id    char(10)     NOT NULL ,
  quantity   int          NOT NULL ,
  item_price decimal(8,2) NOT NULL ,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;
```

### 6.3 Specifying Default Values

MySQL enables you to specify default values to be used if no value is specified when a row is inserted. Default values are specified using the `DEFAULT` keyword in the column definitions in the `CREATE TABLE` statement.

- Example Input

```mysql
CREATE TABLE orderitems
(
  order_num  int          NOT NULL ,
  order_item int          NOT NULL ,
  prod_id    char(10)     NOT NULL ,
  quantity   int          NOT NULL  DEFAULT 1,
  item_price decimal(8,2) NOT NULL ,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;
```

> ### CAUTION
>
> **Functions Are Not Allowed.** Unlike most DBMSs, MySQL does not allow the use of functions as `DEFAULT` values; only constants are supported.

> ### TIP
>
> **Using DEFAULT Instead of NULL Values.** Many database developers use `DEFAULT` values instead of `NULL` columns, especially in columns that will be used in calculations or data groupings.

### 6.4 Engine Types

MySQL has an internal engine that actually manages and manipulates data. When you use the `CREATE TABLE` statement, that engine is used to actually create the tables, and when you use the `SELECT` statement or perform any other database processing, the engine is used internally to process your request. For the most part, the engine is buried within the DBMS and you need not pay much attention to it.

But unlike every other DBMS, MySQL does not come with a single engine. Rather, it ships with several engines, all buried within the MySQL server, and all capable of executing commands such as `CREATE TABLE` and `SELECT`.

So why bother shipping multiple engines? Because they each have different capabilities and features, and being able to pick the right engine for the job gives you unprecedented power and flexibility.

Of course, you are free to totally ignore database engines. If you omit the `ENGINE=` statement, the default engine is used (most likely `MyISAM`), and most of your SQL statements will work as is. But not all of them will, and that is why this is important (and why two engines are used in the sample tables used in this book).

Here are several engines of which to be aware:

- `InnoDB` is a transaction-safe engine (see [Chapter 26](https://learning.oreilly.com/library/view/mysql-crash-course/0672327120/ch26.html), “Managing Transaction Processing”). It does not support full-text searching.
- `MEMORY` is functionally equivalent to `MyISAM`, but as data is stored in memory (instead of on disk) it is extremely fast (and ideally suited for temporary tables).
- `MyISAM` is a very high-performance engine. It supports full-text searching (see [Chapter 18](https://learning.oreilly.com/library/view/mysql-crash-course/0672327120/ch18.html), “Full-Text Searching”), but does not support transactional processing.

> ### CAUTION
>
> **Foreign Keys Can’t Span Engines.** There is one big downside to mixing engine types. Foreign keys (used to enforce referential integrity, as explained in [Chapter 1](https://learning.oreilly.com/library/view/mysql-crash-course/0672327120/ch01.html), “Understanding SQL”) cannot span engines. That is, a table using one engine cannot have a foreign key referring to a table that uses another engine.

### 6.5 Updating Tables

To update table definitions, the `ALTER TABLE` statement is used. But, ideally, tables should never be altered after they contain data. You should spend sufficient time anticipating future needs during the table design process so extensive changes are not required later on.

- Add a column to a table

```mysql
ALTER TABLE vendors
ADD vend_phone CHAR(20);
```

- Remove a column from a table

```mysql
ALTER TABLE vendors
DROP COLUMN vend_phone;
```

#### 6.5.1 Add foreign key into table

One common use for `ALTER TABLE` is to define foreign keys. The following is the code used to define the foreign keys used by the tables in this book:

```mysql
ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);

ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_products FOREIGN KEY (prod_id)
REFERENCES products (prod_id);

ALTER TABLE orders
ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id)
REFERENCES customers (cust_id);

ALTER TABLE products
ADD CONSTRAINT fk_products_vendors
FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);
```

- Deleting Tables

```mysql
DROP TABLE customers2;
```

- Renaming Tables

```mysql
RENAME TABLE customers2 TO customers;
```
