# MySQL Learning Note 1 Basic Query



[Chapter 1: Understanding SQL](##Chapter-1:-Understanding-SQL) \
[1.1 Primary Key](###1.1.-Primary-Key) \
[1.2 MySQL Primary Key Rules](###1.2.-MySQLPrimary-Key-Rules) \
[1.3. Primary Key Best Practices](###1.3.-Primary-Key-Best-Practices) \


## Chapter 1: Understanding SQL 

### 1.1. Primary Key

Primary Key: A column (or set of columns) whose values uniquely identify every row in a table. 

### 1.2. MySQL Primary Key Rules 

- No two rows can have the same primary key value;
- Every row must have a primary key value (primary key columns may not allow NULL values).

### 1.3. Primary Key Best Practices 

- Don't update vlaues in primary key columns. 
- Don't reuse values in primary key columns. 
- Don't use values that might change in primary key columns. (For example, when you use a name as a primary key to identify a supplier, you would have to change the primary key when the supplier merges and changes its name).



## Chapter 2: Understanding MySQL

### 2.1. DBMS Categories: 

- Shared file-based 
  - Designed for desktop use and are generally not intended for use on higher-end or more critical applications. 
- Client-server: Contains the following parts: 
  - Server: a piece of software that is responsible for all data access and manipulation. This software runs on a computer called the database server. 
  - Client: Requests or changes come from computers running client software. This is the software with which the user interacts. 



## Chpater 3: Working with MySQL

### 3.1. Go to a specific data base

```MySQL
USE <database-name>;
```

You will see `Database changed`, if the command got executed correctly. 

### 3.2. Show all the available DBs

```MySQL
SHOW DATABASES;
```

### 3.3. Show available tables in the DB 

```mysql
SHOW TABLES;
```

### 3.4. Display a tables's columns 

```mysql
SHOW COLUMNS FROM customers;
```

### 3.5. Another way to display a table's columns 

```mysql
DESCRIBE customers;
```

### 3.6. Other useful commands

#### 3.6.1 `SHOW CREATE DATABASE` and `SHOW CREATE TABLE`

Used to display the MySQL statements used to create specified databases or tables respectively.

```mysql
show create table `product_category`;
```



#### 3.6.2 `SHOW GRANTS`

Used to display security rights granted to users (all users or a specific user)

#### 3.6.3 `SHOW ERRORS` and `SHOW WARNINGS`

Used to display server error or warning messages

#### 3.6.4 **Learning More About SHOW.** 

In the `mysql` command-line utility, execute command `HELP SHOW;` to display a list of allowed `SHOW` statements.



## Chapter 4. Retrieving Data 

### 4.1 `SELECT` Statement

The purpose of `SELECT` statement is to retrieve information from one or more tables. To use `select` statement, you need to specify at least the following two conditions: 

- What you want to select;
- From where you want to select;

### 4.2. Use of White Space. 

All extra white space within a SQL statement is ignored when that statement is processed. SQL statements can be specified on one long line or broken up over many lines. Most SQL developers find that breaking up statements over multiple lines makes them easier to read and debug.

### 4.3. Retrieving Multiple Columns

To retrieve multiple columns from a table, the same SELECT statement is used. The only difference is that multiple column names must be specified after the SELECT keyword, and each column must be separated by a comma.

> TIP
>
> Take Care with Commas. When selecting multiple columns, be sure to specify a comma between each column name, but not after the last column name. Doing so will generate an error.

### 4.4. Retrieving Distinct Rows 

As you have seen, SELECT returns all matched rows. But what if you did not want every occurrence of every value? Use `DISTINCT` key word. 

- Input

  ```mysql
  SELECT DISTINCT vend_id FROM products;
  ```

### 4.5. Limiting Results

#### 4.5.1. Select from row 0

SELECT statements return all matched rows, possibly every row in the specified table. To return just the first row or rows, use the LIMIT clause. Here is an example:

- Input

  ```mysql
  SELECT prod_name FROM products LIMIT 5;
  ```

#### 4.5.2. Select from a specific row 

To get the next five rows, specify both where to start and the number of rows to retrieve, like this:

- Sample

  ```mysql
  SELECT prod_name
  FROM products
  LIMIT rowstart, limits;
  ```

  

- Input 

  ```mysql
  SELECT prod_name
  FROM products
  LIMIT 5, 5;
  ```

> **CAUTION**
>
> Row 0. The first row retrieved is row 0, not row 1. As such, LIMIT 1,1 will retrieve the second row, not the first one.

> **TIP**
>
> MySQL 5 LIMIT Syntax. Does LIMIT 3,4 mean 3 rows starting from row 4, or 4 rows starting from row 3? As you just learned, it means 4 rows starting from row 3, but it is a bit ambiguous.For this reason, MySQL 5 supports an alternative syntax for LIMIT. LIMIT 4 OFFSET 3 means to get 4 rows starting from row 3, just like LIMIT 3,4.



## Chapter 5. Sorting Retrieved Data

Relational database design theory states that the sequence of retrieved data cannot be assumed to have significance if ordering was not explicitly specified.

### 5.1. Sort data with `ORDER BY`

- Input

  ```mysql
  SELECT prod_name
  FROM products
  ORDER BY prod_name;
  ```

> **TIP**
>
> Sorting by Nonselected Columns. More often than not, the columns used in an ORDER BY clause are ones that were selected for display. However, this is actually not required, and it is perfectly legal to sort data by a column that is not retrieved.



### 5.2. Sorting by Multiple Columns 

To sort by multiple columns, simply specify the column names separated by commas (just as you do when you are selecting multiple columns).

- Input 

  ```mysql
  SELECT prod_id, prod_price, prod_name
  FROM products
  ORDER BY prod_price, prod_name;
  ```

It is important to understand that when you are sorting by multiple columns, the sort sequence is exactly as specified. In other words, using the output in the previous example, the products are sorted by the prod_name column only when multiple rows have the same prod_price value. If all the values in the prod_price column had been unique, no data would have been sorted by prod_name.

### 5.3. Specifying Sort Direction 

Data sorting is not limited to ascending sort orders (from `A` to `Z`). Although this is the default sort order, the `ORDER BY` clause can also be used to sort in descending order (from `Z` to `A`). To sort by descending order, the keyword `DESC` must be specified.

- Input 

  ```mysql
  SELECT prod_id, prod_price, prod_name
  FROM products
  ORDER BY prod_price DESC;
  ```



Sort by multiple columns in `DESC` order. 

```mysql
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price DESC, prod_name DESC;
```



The opposite of DESC is ASC (for ascending), which may be specified to sort in ascending order. In practice, however, ASC is not usually used because ascending order is the default sequence (and is assumed if neither ASC nor DESC are specified).



### 5.4. Choose the most expensive product

Using a combination of ORDER BY and LIMIT, it is possible to find the highest or lowest value in a column. The following example demonstrates how to find the value of the most expensive item:

```mysql
SELECT prod_price 
FROM products
ORDER BY prod_price DESC
LIMIT 1;
```

> CAUTION
>
> Position of ORDER BY Clause. When specifying an ORDER BY clause, be sure that it is after the FROM clause. If LIMIT is used, it must come after ORDER BY. Using clauses out of order will generate an error message.



## Chapter 6. Filtering Data

Within a `SELECT` statement, data is filtered by specifying search criteria in the `WHERE` clause. The `WHERE`clause is specified right after the table name (the FROM clause) as follows:

> CAUTION
>
> `WHERE` Clause Position. When using both `ORDER BY` and `WHERE` clauses, make sure `ORDER BY` comes after the `WHERE`; otherwise an error will be generated. (See Chapter 5, “Sorting Retrieved Data,” for more information on using `ORDER BY`.)

```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_price < 10 
ORDER BY prod_name;
```



### 6.1 The `WHERE` Clause Operators

The first WHERE clause we looked at tests for equality—determining if a column contains a specific value. MySQL supports a whole range of conditional operators, some of which are listed in Table 6.1.

| Operator  |         Description          |
| :-------: | :--------------------------: |
|    `=`    |           Equality           |
|   `<>`    |         Nonequality          |
|   `!=`    |         Nonequality          |
|    `<`    |          Less than           |
|   `<=`    |    Less than or equal to     |
|    `>`    |         Greater than         |
|   `>=`    |   Greater than or equal to   |
| `BETWEEN` | Between two specified values |



### 6.2 Checking for a Range of Values 

To check for a range of values, you can use the BETWEEN operator. Its syntax is a little different from other WHERE clause operators because it requires two values: the beginning and end of the range. The BETWEEN operator can be used, for example, to check for all products that cost between 5 and 10 or for all dates that fall between specified start and end dates.

The following example demonstrates the use of the `BETWEEN` operator by retrieving all products with a price between `5` and `10`:

- Input 

  ```mysql
  SELECT prod_name, prod_price 
  FROM products
  WHERE prod_price BETWEEN 5 AND 10;
  ```

- Analysis

  As seen in this example, when BETWEEN is used, two values must be specified—the low end and high end of the desired range. The two values must also be separated by the AND keyword. BETWEEN matches all the values in the range, including the specified range start and end values.



### 6.3 Checking for No Value 

When a table is created, the table designer can specify whether individual columns can contain no value. When a column contains no value, it is said to contain a `NULL` value.

- Input

  ```mysql
  SELECT prod_name
  FROM products
  WHERE prod_price IS NULL;
  ```

This statement returns a list of all products that have no price (an empty `prod_price` field, not a price of `0`), and because there are none, no data is returned. 

> CAUTION
>
> NULL and Nonmatches. You might expect that when you filter to select all rows that do not have a particular value, rows with a NULL will be returned. But they will not. Because of the special meaning of unknown, the database does not know whether they match, and so they are not returned when filtering for matches or when filtering for nonmatches.When filtering data, make sure to verify that the rows with a NULL in the filtered column are really present in the returned data.



## Chapter 7. Advanced Data Filtering

For a greater degree of filter control, MySQL allows you to specify multiple WHERE clauses. These clauses may be used in two ways: as `AND` clauses or as `OR` clauses.



### 7.1 Using the `AND` Operator

To filter by more than one column, you use the AND operator to append conditions to your WHERE clause. The following code demonstrates this:

- Input

  ```mysql
  SELECT prod_id, prod_price, prod_name
  FROM products
  WHERE vend_id = 1003 AND prod_price <= 10;
  ```

> `AND`
>
> A keyword used in a WHERE clause to specify that only rows matching all the specified conditions should be retrieved.

The example just used contained a single AND clause and was thus made up of two filter conditions. Additional filter conditions could be used as well, each seperated by an AND keyword.



### 7.2 Using the `OR` Operator

The `OR` operator is exactly the opposite of `AND`. The `OR` operator instructs MySQL to retrieve rows that match either condition.

- Input 

  ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE vend_id = 1002 OR vend_id = 1003;
  ```

> **OR.** 
>
> A keyword used in a `WHERE` clause to specify that any rows matching either of the specified conditions should be retrieved.



### 7.3 Understanding Order of Evaluation

`WHERE` clauses can contain any number of AND and OR operators. Combining the two enables you to perform sophisticated and complex filtering.

**But combining `AND` and `OR` operators presents an interesting problem. **To demonstrate this, look at an example. You need a list of all products costing 10 or more made by vendors 1002 and 1003. The following `SELECT` statement uses a combination of `AND` and `OR` operators to build a `WHERE` clause:

- Input 

  ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;
  ```

- Output 

  | prod_name      | prod_price |
  | -------------- | ---------- |
  | Detonator      | 13.00      |
  | Bird seed      | 10.00      |
  | Fuses          | 3.42       |
  | Oil can        | 8.99       |
  | Safe           | 50.00      |
  | TNT (5 sticks) | 10.00      |

- Analysis

  Look at the previously listed results. Two of the rows returned have prices less than 10—so, obviously, the rows were not filtered as intended. Why did this happen? The answer is the order of evaluation. **SQL (like most languages) processes AND operators before OR operators. **When SQL sees the preceding WHERE clause, it reads products made by vendor 1002 regardless of price, and any products costing 10 or more made by vendor 1003. In other words, because AND ranks higher in the order of evaluation, the wrong operators were joined together.

  **The solution to this problem is to use parentheses to explicitly group related operators. Take a look at the following `SELECT` statement and output:**

- Input

  ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;
  ```

- Output

  | prod_name      | prod_price |
  | -------------- | ---------- |
  | Detonator      | 13.00      |
  | Bird seed      | 10.00      |
  | Safe           | 50.00      |
  | TNT (5 sticks) | 10.00      |

  > **Tip**
  >
  > Using Parentheses in WHERE Clauses. Whenever you write WHERE clauses that use both AND and OR operators, use parentheses to explicitly group operators. Don’t ever rely on the default evaluation order, even if it is exactly what you want. There is no downside to using parentheses, and you are always better off eliminating any ambiguity.



### 7.4 Using the `IN` Operator

Parentheses have another very different use in WHERE clauses. The IN operator is used to specify a range of conditions, any of which can be matched. IN takes a comma-delimited list of valid values, all enclosed within parentheses. The following example demonstrates this:

- Input

  ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE vend_id IN (1002, 1003)
  ORDER BY prod_name;
  ```

- You can get the same result when you use `OR` 

  ```mysql
  SELECt prod_name, prod_price
  FROM products
  WHERE vend_id = 1002 OR vend_id = 1003
  ORDER BY prod_name;
  ```

- Why use the `IN` operator? 

  - When you are working with long lists of valid options, the IN operator syntax is far cleaner and easier to read.
  - The order of evaluation is easier to manage when `IN` is used (as there are fewer operators used).
  - IN operators almost always execute more quickly than lists of OR operators.
  - The biggest advantage of IN is that the IN operator can contain another SELECT statement, enabling you to build highly dynamic WHERE clauses. You’ll look at this in detail in Chapter 14, “Working with Subqueries.”



### 7.5 Using the `NOT` Operator 

> **NOTE**
>
> **NOT**. A keyword used in a `WHERE` clause to negate (make ineffective) a condition 

- Input 

- ```mysql
  SELECT prod_name, prod_price
  FROM products
  WHERE vend_id NOT IN (1002, 1003)
  ORDER BY prod_name;
  ```

- Analysis 

  The NOT here negates the condition that follows it; so instead of matching vend_id to 1002 or 1003, MySQL matches vend_id to anything that is not 1002 or 1003.

  **So why use NOT? **

  Well, for simple WHERE clauses, there really is no advantage to using NOT. NOT is useful in more complex clauses. For example, using NOT in conjunction with an IN operator makes it simple to find all rows that do not match a list of criteria.

> **NOTE**
>
> **NOT** in MySQL. MySQL supports the use of NOT to negate IN, BETWEEN, and EXISTS clauses. This is quite different from most other DBMSs that allow NOT to be used to negate any conditions.



