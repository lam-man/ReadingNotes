# MySQL Learning Note 4 View, Store Procedure, Cursor and Trigger

[TOC]

## 1. Chapter 22. Using Views

### 1.1 Understanding Views

Views are virtual tables. Unlike tables that contain data, views simply contain queries that dynamically retrieve data when used.

Instead of using the following query every time to get the customers that order a specific kind of product, you can put it into a **Store Procedures**.

```mysql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
  AND orderitems.order_num = orders.order_num
  AND prod_id = 'TNT2';
```

After changing it to store procedure, you will have the following:

```mysql
SELECT cust_name, cust_contact
FROM productcustomers
WHERE prod_id = 'TNT2';
```

### 1.2 Why views

- To reuse SQL statements.
- To simplify complex SQL operations. After the query is written, it can be reused easily, without having to know the details of the underlying query itself.
- To expose parts of a table instead of complete tables.
- To secure data. Users can be given access to specific subsets of tables instead of to entire tables.
- To change data formatting and representation. Views can return data formatted and presented differently from their underlying tables.

For the most part, after views are created, they can be used in the same way as tables. You can perform `SELECT` operations, filter and sort data, join views to other views or tables, and possibly even add and update data. (There are some restrictions on this last item. More on that in a moment.)

> ### CAUTION
>
> **Performance Issues.** Because views contain no data, any retrieval needed to execute a query must be processed every time the view is used. If you create complex views with multiple joins and filters, or if you nest views, you may find that performance is dramatically degraded. Be sure you test execution before deploying applications that use views extensively.

### 1.3 Using Views

- Views are created using the `CREATE VIEW` statement.
- To view the statement used to create a view, use `SHOW CREATE VIEW` _viewname;_.
- To remove a view, the `DROP` statement is used. The syntax is simply `DROP VIEW` _viewname;_
- To update a view you may use the `DROP` statement and then the `CREATE` statement again, or just use `CREATE OR REPLACE VIEW`, which will create it if it does not exist and replace it if it does.

### 1.4 Using Views to Simplify Complex Joins

- Example Input

- ```mysql
  CREATE VIEW productcustomers AS
  SELECT cust_name, cust_contact, prod_id
  FROM customers, orders, orderitems
  WHERE customers.cust_id = orders.cust_id
    AND orderitems.order_num = orders.order_num;
  ```

- Analysis

  This statement creates a view named `productcustomers`, which joins three tables to return a list of all customers who have ordered any product. If you were to `SELECT * FROM productcustomers`, you’d list every customer who ordered anything.

  To retrieve a list of customers who ordered product `TNT2`, you can do the following:

- ```mysql
  SELECT cust_name, cust_contact
  FROM productcustomers
  WHERE prod_id = 'TNT2';
  ```

> ### TIP
>
> **Creating Reusable Views.** It is a good idea to create views that are not tied to specific data. For example, the view created in this example returns customers for all products, not just product `TNT2` (for which the view was first created). Expanding the scope of the view enables it to be reused, making it even more useful. It also eliminates the need for you to create and maintain multiple similar views.

### 1.5 Updating Views

All of the views thus far have been used with `SELECT` statements. But can view data be updated? The answer is that it depends.

As a rule, yes, views are updateable (that is, you can use `INSERT`, `UPDATE`, and `DELETE` on them). Updating a view updates the underlying table (the view, you will recall, has no data of its own); if you add or remove rows from a view you are actually removing them from the underlying table.

But not all views are updateable. Basically, if MySQL is unable to correctly ascertain the underlying data to be updated, updates (this includes inserts and deletes) are not allowed. In practice, this means that if any of the following are used you’ll not be able to update the view:

- Grouping (using `GROUP BY` and `HAVING`)
- Joins
- Subqueries
- Unions
- Aggregate functions (`Min()`, `Count()`, `Sum()`, and so forth)
- `DISTINCT`
- Derived (calculated) columns

> ### TIP
>
> **Use Views for Retrieval.** As a rule, use views for data retrieval (`SELECT` statements) and not for updates (`INSERT`, `UPDATE`, and `DELETE`).

## 2. Working with Stored Procedures

> CAUTION: [Why stored procedure is faster?](https://stackoverflow.com/questions/8559443/why-execute-stored-procedures-is-faster-than-sql-query-from-a-script)

Stored procedures are simply collections of one or more MySQL statements saved for future use. You can think of them as batch files, although in truth they are more than that.

### 2.1 Why Use Stored Procedures

- To simplify complex operations (as seen in the previous example) by encapsulating processes into a single easy-to-use unit.

- To ensure data integrity by not requiring that a series of steps be created over and over. If all developers and applications use the same (tried and tested) stored procedure, the same code will be used by all.

  An extension of this is to prevent errors. The more steps that need to be performed, the more likely it is that errors will be introduced. Preventing errors ensures data consistency.

- To simplify change management. If tables, column names, or business logic (or just about anything) changes, only the stored procedure code needs to be updated, and no one else will need even to be aware that changes were made.

  An extension of this is security. Restricting access to underlying data via stored procedures reduces the chance of data corruption (unintentional or otherwise).

- To improve performance, as stored procedures typically execute quicker than do individual SQL statements.

- There are MySQL language elements and features that are available only within single requests. Stored procedures can use these to write code that is more powerful and flexible. (We’ll see an example of this in the next chapter).

In other words, there are three primary benefits—simplicity, security, and performance. Obviously all are extremely important. Before you run off to turn all your SQL code into stored procedures, here’s the downside:

- Stored procedures tend to be more complex to write than basic SQL statements, and writing them requires a greater degree of skill and experience.
- You might not have the security access needed to create stored procedures. Many database administrators restrict stored procedure creation rights, allowing users to execute them but not necessarily create them.

### 2.2 Creating Stored Procedures

- Example

```mysql
CREATE PROCEDURE productpricing()
BEGIN
   SELECT Avg(prod_price) AS priceaverage
   FROM products;
END;
```

> ### NOTE
>
> **mysql Command-line Client Delimiters.** If you are using the mysql command-line utility, pay careful attention to this note.
>
> The default MySQL statement delimiter is `;` (as you have seen in all of the MySQL statement used thus far). However, the mysql command-line utility also uses `;` as a delimiter. If the command-line utility were to interpret the `;` characters inside of the stored procedure itself, those would not end up becoming part of the stored procedure, and that would make the SQL in the stored procedure syntactically invalid.
>
> The solution is to temporarily change the command-line utility delimiter, as seen here:
>
> ```
> DELIMITER //
>
> CREATE PROCEDURE productpricing()
> BEGIN
>    SELECT Avg(prod_price) AS priceaverage
>    FROM products;
> END //
>
> DELIMITER ;
> ```
>
> Here, `DELIMITER //` tells the command-line utility to use `//` as the new end of statement delimiter, and you will notice that the `END` that closes the stored procedure is defined as `END //` instead of the expected `END;`. This way the `;` within the stored procedure body remains intact and is correctly passed to the database engine. And then, to restore things back to how they were initially, the statement closes with a `DELIMITER ;`.
>
> Any character may be used as the delimiter except for `\`.
>
> If you are using the mysql command-line utility, keep this in mind as you work through this chapter.

- Executing stored procedures

```mysql
CALL productpricing();
```

### 2.3 Using Stored Procedures

#### 2.3.1 Show & Dropping Stored Procedures

- Example of showing

```mysql
SHOW CREATE PROCEDURE productpricing;
```

- Example of dropping

```mysql
DROP PROCEDURE productpricing;
```

- Drop when exist

> ### TIP
>
> **Drop Only If It Exists.** `DROP PROCEDURE` will throw an error if the named procedure does not actually exist. To delete a procedure if it exists (and not throw an error if it does not), use `DROP PROCEDURE IF EXISTS`.

#### 2.3.2 Executing Stored Procedures with Parameters

> ### NEW TERM
>
> **Variable.** A named location in memory, used for temporary storage of data.

- Creating Stored Procedure with variables

```mysql
CREATE PROCEDURE productpricing(
   OUT pl DECIMAL(8,2),
   OUT ph DECIMAL(8,2),
   OUT pa DECIMAL(8,2)
)
BEGIN
   SELECT Min(prod_price)
   INTO pl
   FROM products;
   SELECT Max(prod_price)
   INTO ph
   FROM products;
   SELECT Avg(prod_price)
   INTO pa
   FROM products;
END;
```

MySQL refers to stored procedure execution as _calling_, and so the MySQL statement to execute a stored procedure is simply `CALL`. `CALL` takes the name of the stored procedure and any parameters that need to be passed to it. Take a look at this example:

- **Input**

```mysql
CALL productpricing(@pricelow,
                    @pricehigh,
                    @priceaverage);
```

- **Analysis**

Here a stored procedure named `productpricing` is executed; it calculates and returns the lowest, highest, and average product prices.

Stored procedures might or might not display results, as you will see shortly.

> ### NOTE
>
> **Variable Names.** All MySQL variable names must begin with `@`.

- How to use variables:

```mysql
SELECT @priceaverage;
```

```mysql
SELECT @pricehigh, @pricelow, @priceaverage;
```

#### 2.3.3 Create Stored Procedures with `IN` and `OUT` Parameters

Here is another example, this time using both `IN` and `OUT` parameters. `ordertotal` accepts an order number and returns the total for that order:

- Input

```mysql
CREATE PROCEDURE ordertotal(
   IN onumber INT,
   OUT ototal DECIMAL(8,2)
)
BEGIN
   SELECT Sum(item_price*quantity)
   FROM orderitems
   WHERE order_num = onumber
   INTO ototal;
END;
```

- Call and get result

```mysql
CALL ordertotal(20005, @total);
SELECT @total;
```

### 2.4 Building Intelligent Stored Procedures

Consider this scenario. You need to obtain order totals as before, but also need to add sales tax to the total, but only for some customers (perhaps the ones in your own state). Now you need to do several things:

- Obtain the total (as before).
- Conditionally add tax to the total.
- Return the total (with or without tax).

That’s a perfect job for a stored procedure:

```mysql
-- Name: ordertotal
-- Parameters: onumber = order number
--             taxable = 0 if not taxable, 1 if taxable
--             ototal = order total variable

CREATE PROCEDURE ordertotal(
   IN onumber INT,
   IN taxable BOOLEAN,
   OUT ototal DECIMAL(8,2)
) COMMENT 'Obtain order total, optionally adding tax'
BEGIN

   -- Declare variable for total
   DECLARE total DECIMAL(8,2);
   -- Declare tax percentage
   DECLARE taxrate INT DEFAULT 6;

   -- Get the order total
   SELECT Sum(item_price*quantity)
   FROM orderitems
   WHERE order_num = onumber
   INTO total;

   -- Is this taxable?
   IF taxable THEN
      -- Yes, so add taxrate to the total
      SELECT total+(total/100*taxrate) INTO total;
   END IF;

   -- And finally, save to out variable
   SELECT total INTO ototal;

END;
```

> ### TIP
>
> **The COMMENT Keyword.** The stored procedure for this example included a `COMMENT` value in the `CREATE PROCEDURE` statement. This is not required, but if specified, is displayed in `SHOW PROCEDURE STATUS` results.

## 3. Chapter 24. Using Cursors

Sometimes there is a need to step through rows forward or backward and one or more at a time. This is what cursors are used for. A cursor is a database query stored on the MySQL server—not a `SELECT` statement, but the result set retrieved by that statement. Once the cursor is stored, applications can scroll or browse up and down through the data as needed.

> ### NOTE
>
> **Only in Stored Procedures.** Unlike most DBMSs, MySQL cursors may only be used within stored procedures (and functions).

Using cursors involves several distinct steps:

1. Before a cursor can be used it must be declared (defined). This process does not actually retrieve any data; it merely defines the `SELECT` statement to be used.
2. After it is declared, the cursor must be opened for use. This process actually retrieves the data using the previously defined `SELECT` statement.
3. With the cursor populated with data, individual rows can be fetched (retrieved) as needed.
4. When it is done, the cursor must be closed.

### 3.1 Working with Cursors

### 3.1.1 Creating Cursors

Cursors are created using the `DECLARE` statement. `DECLARE` names the cursor and takes a `SELECT` statement, complete with `WHERE` and other clauses if needed.

- Example

```mysql
CREATE PROCEDURE processorders()
BEGIN
    DECLARE ordernumbers CURSOR
    FOR
    SELECT order_num FROM orders;
END;
```

- Analysis

This stored procedure does not do a whole lot. A `DECLARE` statement is used to define and name the cursor—in this case `ordernumbers`. Nothing is done with the cursor, and as soon as the stored procedure finishes processing it will cease to exist (as it is local to the stored procedure itself).

#### 3.1.2 Opening and Closing Cursors

- Open curosr

```mysql
OPEN ordernumbers;
```

When the `OPEN` statement is processed, the query is executed, and the retrieved data is stored for subsequent browsing and scrolling.

- Close cursor

```mysql
CLOSE ordernumbers;
```

`CLOSE` frees up any internal memory and resources used by the cursor, and so every cursor should be closed when it is no longer needed.

After a cursor is closed, it cannot be reused without being opened again. However, a cursor does not need to be declared again to be used; an `OPEN` statement is sufficient.

```mysql
CREATE PROCEDURE processorders()
BEGIN
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;

	-- Open the cursor
	OPEN ordernumbers;

	-- Close the cursor
	CLOSE ordernumbers;
END;
```

This stored procedure declares, opens, and closes a cursor. However, nothing is done with the retrieved data.

### 3.2 Using Cursor Data

After a cursor is opened, each row can be accessed individually using a `FETCH` statement. `FETCH` specifies what is to be retrieved (the desired columns) and where retrieved data should be stored. It also advances the internal row pointer within the cursor so the next `FETCH` statement will retrieve the next row (and not the same one over and over).

- Simple Example

```mysql
CREATE PROCEDURE processorders()
BEGIN

   -- Declare local variables
   DECLARE o INT;

   -- Declare the cursor
   DECLARE ordernumbers CURSOR
   FOR
   SELECT order_num FROM orders;

   -- Open the cursor
   OPEN ordernumbers;

   -- Get order number
   FETCH ordernumbers INTO o;

   -- Close the cursor
   CLOSE ordernumbers;

END;
```

- Usage Example

```mysql
CREATE PROCEDURE processorders()
BEGIN

   -- Declare local variables
   DECLARE done BOOLEAN DEFAULT 0;
   DECLARE o INT;

   -- Declare the cursor
   DECLARE ordernumbers CURSOR
   FOR
   SELECT order_num FROM orders;

   -- Declare continue handler
   DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;

   -- Open the cursor
   OPEN ordernumbers;

   -- Loop through all rows
   REPEAT

      -- Get order number
      FETCH ordernumbers INTO o;

   -- End of loop
   UNTIL done END REPEAT;

   -- Close the cursor
   CLOSE ordernumbers;

END;
```

Like the previous example, this example uses `FETCH` to retrieve the current `order_num` into a declared variable named `o`. Unlike the previous example, the `FETCH` here is within a `REPEAT` so it is repeated over and over until `done` is true (as specified by `UNTIL done END REPEAT;`). To make this work, variable `done` is defined with a `DEFAULT 0` (false, not done). So how does `done` get set to true when done? The answer is this statement:

```mysql
DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
```

This statement defines a `CONTINUE HANDLER`, code that will be executed when a condition occurs. Here it specifies that when `SQLSTATE '02000'` occurs, then `SET done=1`. And `SQLSTATE '02000'` is a _not found_ condition and so it occurs when `REPEAT` cannot continue because there are no more rows to loop through.

> ### CAUTION
>
> **DECLARE Statement Sequence.** There is specific order in which `DECLARE` statements, if used, must be issued. Local variables defined with `DECLARE` must be defined before any cursors or handlers are defined, and handlers must be defined after any cursors. Failure to follow this sequencing will generate an error message.

> ### NOTE
>
> **REPEAT or LOOP?** In addition to the `REPEAT` statement used here, MySQL also supports a `LOOP` statement that can be used to repeat code until the `LOOP` is manually exited using a `LEAVE` statement. In general, the syntax of the `REPEAT` statement makes it better suited for looping through cursors.

```mysql
CREATE PROCEDURE processorders()
BEGIN

   -- Declare local variables
   DECLARE done BOOLEAN DEFAULT 0;
   DECLARE o INT;
   DECLARE t DECIMAL(8,2);

   -- Declare the cursor
   DECLARE ordernumbers CURSOR
   FOR
   SELECT order_num FROM orders;

   -- Declare continue handler
   DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;

   -- Create a table to store the results
   CREATE TABLE IF NOT EXISTS ordertotals
      (order_num INT, total DECIMAL(8,2));

   -- Open the cursor
   OPEN ordernumbers;

   -- Loop through all rows
   REPEAT

      -- Get order number
      FETCH ordernumbers INTO o;

      -- Get the total for this order
      CALL ordertotal(o, 1, t);

      -- Insert order and total into ordertotals
      INSERT INTO ordertotals(order_num, total)
      VALUES(o, t);

   -- End of loop
   UNTIL done END REPEAT;

   -- Close the cursor
   CLOSE ordernumbers;

END;
```

This stored procedure returns no data, but it does create and populate another table that can then be viewed using a simple `SELECT` statement.

## 4. Chapter 25. Using Triggers

A _trigger_ is a MySQL statement (or a group of statements enclosed within `BEGIN` and `END` statements) that are automatically executed by MySQL in response to any of these statements:

- `DELETE`
- `INSERT`
- `UPDATE`

No other MySQL statements support triggers.

### 4.1 Creating Triggers

When creating a trigger you need to specify four pieces of information:

- The unique trigger name
- The table to which the trigger is to be associated
- The action that the trigger should respond to (`DELETE`, `INSERT`, or `UPDATE`)
- When the trigger should be executed (before or after processing)

Triggers are created using the `CREATE TRIGGER` statement.

```mysql
CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'Product added' AS '';
```

This chapter is buggy, please see the following link for more about Trigger.

https://dev.mysql.com/doc/refman/5.5/en/trigger-syntax.html
