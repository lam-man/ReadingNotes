# MySQL Learning Note 2 Advanced Query

[TOC]

## Chapter 8. Using Wildcard Filtering 

### 8.1 Using the `LIKE` Operator

The previously learned query technics **has the common denominator is that the values used in the filtering are known. ** But filtering data that way does not always work. For example, how could you search for all products that contained the text anvil within the product name? That cannot be done with simple comparison operators; that’s a job for wildcard searching. 

The wildcards themselves are actually characters that have special meanings within SQL `WHERE` clauses, and SQL supports several wildcard types.

To use wildcards in search clauses, the `LIKE` operator must be used. `LIKE` instructs MySQL that the following search pattern is to be compared using a wildcard match rather than a straight equality match.

> **NOTE**
>
> **Predicates**. When is an operator not an operator? When it is a predicate. Technically, LIKE is a predicate, not an operator. The end result is the same; just be aware of this term in case you run across it in the MySQL documentation.



### 8.2 The Percent Sign (%) Wildcard 

The most frequently used wildcard is the percent sign (%). Within a search string, % means match any number of occurrences of any character. 

#### 8.2.1 `%` at the end of text

For example, to find all products that start with the word jet, you can issue the following SELECT statement:

- Input 

  ```mysql
  SELECT prod_id, prod_name 
  FROM products 
  WHERE prod_name LIKE 'jet%';
  ```

- Analysis

  This example uses a search pattern of `jet%`. When this clause is evaluated, any value that starts with jet is retrieved. The % tells MySQL to accept any characters after the word jet, regardless of how many characters there are.

  > **NOTE**
  >
  > Case-sensitivity. Depending on how MySQL is configured, searches might be case-sensitive, in which case `jet%` would not match JetPack 1000. 

#### 8.2.2 `%` at the beginning and the end 

Wildcards can be used anywhere within the search pattern, and multiple wildcards can be used as well. The following example uses two wildcards, one at either end of the pattern:

- Input 

  ```mysql
  SELECT prod_id, prod_name
  FROM products
  WHERE prod_name LIKE '%anvil%';
  ```

- Analysis 

  The search pattern `%anvil%` means **match any value that contains the text anvil anywhere within it**, regardless of any characters before or after that text.

#### 8.2.3 `%` at the middle 

Wildcards can also be used in the middle of a search pattern, although that is rarely useful. The following example finds all products that begin with an `s` and end with an `e`:

- Input 

  ```mysql
  SELECT prod_name 
  FROM products 
  WHERE prod_name LIKE 's%e';
  ```



#### 8.2.4 `%` can be used to match nothing 

It is important to note that, in addition to matching one or more characters,  `%` **also matches zero characters.** `%` represents zero, one, or more characters at the specified location in the search pattern.



> **NOTE**
>
> NoteWatch for Trailing Spaces. Trailing spaces can interfere with wildcard matching. For example, if any of the anvils had been saved with one or more spaces after the word anvil, the clause WHERE prod_name LIKE '%anvil' would not have matched them as there would have been additional characters after the final l. One simple solution to this problem is to always append a final % to the search pattern. A better solution is to trim the spaces using functions, as is discussed in Chapter 11, “Using Data Manipulation Functions.”

> **CAUTION**
>
> **Watch for NULL.** Although it might seem that the `%` wildcard matches anything, there is one exception—`NULL`. Not even the clause `WHERE prod_name LIKE '%'` will match a row with the value `NULL` as the product name.



### 8.3. The Undersocre `_` wildcard 

Another useful wildcard is the underscore (_). The underscore is used just like %, **but instead of matching multiple characters, the underscore matches just a single character.**

- Input 

  ```mysql
  SELECT prod_id, prod_name
  FROM products
  WHERE prod_name LIKE '_ ton anvil';
  ```

- Output 

  | prod_id | prod_name   |
  | ------- | ----------- |
  | ANV02   | 1 ton anvil |
  | ANV03   | 2 ton anvil |

- Analysis 

  The search pattern used in this `WHERE` clause specifies a wildcard followed by literal text. The results shown are the only rows that match the search pattern: The underscore matches `1` in the first row and `2` in the second row. The `.5 ton anvil` product did not match because the search pattern matched a single character, not two. By contrast, the following `SELECT`statement uses the `%` wildcard and returns three matching products:



### 8.4 Tips for Using Wildcards
As you can see, MySQL’s wildcards are extremely powerful. But that power comes with a price: **Wildcard searches typically take far longer to process than any other search types discussed previously.** Here are some tips to keep in mind when using wildcards:

- Don’t overuse wildcards. If another search operator will do, use it instead.

- When you do use wildcards, try to not use them at the beginning of the search pattern unless absolutely necessary. Search patterns that begin with wildcards are the slowest to process.

- Pay careful attention to the placement of the wildcard symbols. If they are misplaced, you might not return the data you intended.Having said that, wildcards are an important and useful search tool, and one that you will use frequently.



## Chapter 9. Searching Using Regular Expressions 

Regular  expressions are special strings (sets of characters) that are used to match text.

### 9.1 Basic Character Matching 

- Retrieve all rows where column `prod_name` contains the text 1000: 

- ```mysql
  SELECT prod_name
  FROM products
  WHERE prod_name REGEXP '1000'
  ORDER BY prod_name;
  ```

- Single character `REGEXP`

  ```mysql
  SELECT prod_name 
  FROM products 
  WHERE prod_name REGEXP '.000'
  ORDER BY prod_name;
  ```



### 9.2 Difference between `LIKE` and `REGEXP`

Code example: 

- `LIKE` example

```mysql 
SELECT prod_name
FROM products
WHERE prod_name LIKE '1000'
ORDER BY prod_name;
```

- `REGEXP` example

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000'
ORDER BY prod_name;
```

`LIKE` will return 0 rows; `REGEXP` will return 1 row. 

Why? 

- `LIKE` matches an entire column. If the text to be matched existed in the middle of a column value, `LIKE` would not find it and the row would not be returned (unless wildcard characters were used). 
- `REGEXP`, on the other hand, looks for matches within column values, `REGEXP` would find it and the row would be returned. 



> ### TIP
>
> **Matches Are Not Case-Sensitive.** Regular expression matching in MySQL (as of version 3.23.4) are not case-sensitive (either case will be matched). To force case-sensitivity, you can use the `BINARY` keyword, as in
>
> ```mysql
> WHERE prod_name REGEXP BINARY 'JetPack .000'
> ```



### 9.3 Performing `OR` matches 

To search for one of two strings (either one or the other), use `|` as seen here: 

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP `1000|2000`
ORDER BY prod_name;
```



> ### TIP
>
> **More Than Two OR Conditions.** More than two `OR` conditions may be specified. For example, `'1000|2000|3000'` would match `1000` or `2000` or `3000`.



### 9.4 Matching One of Several Characters 

`.` matches any single character. But what if you wanted to match only specific characters? You can do this by specifying a set of characters enclosed within `[`and`]`.

- Example

```mysql
SELECT prod_name
FROM products 
WHERE prod_name REGEXP '[123] Ton'
ORDER BY prod_name;
```

- Result 

```
+-------------+
| prod_name   |
+-------------+
| 1 ton anvil |
| 2 ton anvil |
+-------------+
```

As you have just seen, `[]` is another form of `OR` statement. In fact, the regular expression `[123] Ton` is shorthand for `[1|2|3] Ton`, which also would have worked. But the `[]` characters are needed to define what the `OR` statement is looking for. 

- Example using `OR`

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1|2|3 Ton'
ORDER BY prod_name;
```

- Result 

```
+---------------+
| prod_name     |
+---------------+
| 1 ton anvil   |
| 2 ton anvil   |
| JetPack 1000  |
| JetPack 2000  |
| TNT (1 stick) |
+---------------+
```

- Compare and Analysis: 

Well, that did not work. The two required rows were retrieved, but so were three others. This happened because MySQL assumed that you meant *‘1’ or ‘2’ or ‘3 ton’*. The `|` character applies to the entire string unless it is enclosed with a set.

#### 9.4.1 Negate a set using `^`

Sets of characters can also be negated. That is, they’ll match anything *but* the specified characters. To negate a character set, place a `^` at the start of the set. So, whereas `[123]` matches characters `1`, `2`, or `3`, `[^123]` matches anything but those characters.



### 9.5 Matching Ranges 

Sets can be used to define one or more characters to be matched. For example, the following will match digits `0` through `9`:

`[0123456789]` is the same as `[0-9]`. Ranges are not limited to complete sets—`[1-3]` and `[6-9]` are valid ranges, too. In addition, ranges need not be numeric, and so `[a-z]` will match any alphabetical character.

- Example: 

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[1-5] Ton'
ORDER BY prod_name;
```

- Result

```
+--------------+
| prod_name    |
+--------------+
| .5 ton anvil |
| 1 ton anvil  |
| 2 ton anvil  |
+--------------+
```

### 9.6 Matching Special Characters 

To match special characters they must be preceded by `\\`. So, `\\-` means find `–` and `\\.`means find `.`:

- Example 

```mysql
SELECT vend_name
FROM vendors
WHERE vend_name REGEXP '\\.'
ORDER BY vend_name;
```

| Metacharacter | Description     |
| :------------ | :-------------- |
| `\\f`         | Form feed       |
| `\\n`         | Line feed       |
| `\\r`         | Carriage return |
| `\\t`         | Tab             |
| `\\v`         | Vertical tab    |

> ### NOTE
>
> **\ or \\?** Most regular expression implementation use a single backslash to escape special characters to be able to use them as literals. MySQL, however, requires two backslashes (MySQL itself interprets one and the regular expression library interprets the other).



### 9.7 Matching Multiple Instances 

All of the regular expressions used thus far attempt to match a single occurrence. If there is a match, the row is retrieved and if not, nothing is retrieved. But sometimes you’ll require greater control over the number of matches. For example, you might want to locate all numbers regardless of how many digits the number contains, or you might want to locate a word but also be able to accommodate a trailing `s` if one exists, and so on.

**Repetition Metacharacters**

| Metacharacter | Description                                |
| :------------ | :----------------------------------------- |
| `*`           | 0 or more matches                          |
| `+`           | 1 or more matches (equivalent to `{1,}`)   |
| `?`           | 0 or 1 match (equivalent to `{0,1}`)       |
| `{n}`         | Specific number of matches                 |
| `{n,}`        | No less than a specified number of matches |
| `{n,m}`       | Range of matches (`m` not to exceed 255)   |



## Chapter 10. Creating Calculated Fields 

This is where calculated fields come in. Unlike all the columns we retrieved in the chapters thus far, calculated fields don’t actually exist in database tables. Rather, a calculated field is created on-the-fly within a SQL `SELECT` statement. 





### 10.1 Concatenating Fields 

#### 10.1.1 Using `Concat()`

> **Concatenate.** Joining values together (by appending them to each other) to form a single long value.

When you can only return one column and the data actually contains more than one column, you should use `Concat()` function.

```mysql
SELECT Concat(vend_name, ' (', vend_country, ')')
FROM vendors
ORDER BY vend_name;
```

Result:

```
+--------------------------------------------+
| Concat(vend_name, ' (', vend_country, ')') |
+--------------------------------------------+
| ACME (USA)                                 |
| Anvils R Us (USA)                          |
| Furball Inc. (USA)                         |
| Jet Set (England)                          |
| Jouets Et Ours (France)                    |
| LT Supplies (USA)                          |
+--------------------------------------------+
```

• **Analysis**

`Concat()` concatenates strings, appending them to each other to create one bigger string. `Concat()`requires one or more values to be specified, each separated by commas. The previous `SELECT` statements concatenate four elements:

- The name stored in the `vend_name` column
- A string containing a space and an open parenthesis
- The state stored in the `vend_country` column
- A string containing the close parenthesis

#### 10.1.2 Using `RTrim()`

```mysql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')')
FROM vendors
ORDER BY vend_name;
```

 The `RTrim()` function trims all spaces from the right of a value. By using `RTrim()`, the individual columns are all trimmed properly.

#### 10.1.3 Using Aliases in `Concat()`

- Input

```mysql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS
vend_title
FROM vendors
ORDER BY vend_name;
```

- Output

```
+-------------------------+
| vend_title              |
+-------------------------+
| ACME (USA)              |
| Anvils R Us (USA)       |
| Furball Inc. (USA)      |
| Jet Set (England)       |
| Jouets Et Ours (France) |
| LT Supplies (USA)       |
+-------------------------+
```



### 10.2 Performing Mathematical Calculations 

- Example input 

```mysql
SELECT prod_id, quantity, item_price
FROM orderitems
WHERE order_num = 20005;
```

- Example output 

```
+---------+----------+------------+
| prod_id | quantity | item_price |
+---------+----------+------------+
| ANV01   |       10 |       5.99 |
| ANV02   |        3 |       9.99 |
| TNT2    |        5 |      10.00 |
| FB      |        1 |      10.00 |
+---------+----------+------------+
```



- Example with mathematial Calculation

```mysql
SELECT prod_id,
			 quantity,
			 item_price,
			 quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```

- Example results

```
+---------+----------+------------+----------------+
| prod_id | quantity | item_price | expanded_price |
+---------+----------+------------+----------------+
| ANV01   |       10 |       5.99 |          59.90 |
| ANV02   |        3 |       9.99 |          29.97 |
| TNT2    |        5 |      10.00 |          50.00 |
| FB      |        1 |      10.00 |          10.00 |
+---------+----------+------------+----------------+
```



> ### TIP
>
> **How to Test Calculations.** `SELECT` provides a great way to test and experiment with functions and calculations. Although `SELECT` is usually used to retrieve data from a table, the `FROM` clause may be omitted to simply access and work with expressions. For example, `SELECT 3 * 2`; would return `6`, `SELECT Trim(' abc ');` would return `abc`, and `SELECT Now()` uses the `Now()` function to return the current date and time. You get the idea—use `SELECT` to experiment as needed.



## Chapter 11. Using Data Manipulation Functions 

> ### NOTE
>
> **Functions Are Less Portable Than SQL.** If you do decide to use functions, make sure you comment your code well, so that at a later date you (or another developer) will know exactly to which SQL implementation you were writing.



### 11.1 Text Manipulation Functions 

- `Upper()` 

```mysql 
SELECT vend_name, UPPER(vend_name) AS vend_name_upcase
FROM vendors
ORDER BY vend_name;
```

- Supported functions 

**Table 11.1. Commonly Used Text-Manipulation Functions**

| Function      | Description                             |
| :------------ | :-------------------------------------- |
| `Left()`      | Returns characters from left of string  |
| `Length()`    | Returns the length of a string          |
| `Locate()`    | Finds a substring within a string       |
| `Lower()`     | Converts string to lowercase            |
| `LTrim()`     | Trims white space from left of string   |
| `Right()`     | Returns characters from right of string |
| `RTrim()`     | Trims white space from right of string  |
| `Soundex()`   | Returns a string’s SOUNDEX value        |
| `SubString()` | Returns characters from within a string |
| `Upper()`     | Converts string to uppercase            |



### 11.2 Date and Time Manipulation Functions 

**Table 11.2. Commonly Used Date and Time Manipulation Functions**

| Function        | Description                                 |
| :-------------- | :------------------------------------------ |
| `AddDate()`     | Add to a date (days, weeks, and so on)      |
| `AddTime()`     | Add to a time (hours, minutes, and so on)   |
| `CurDate()`     | Returns the current date                    |
| `CurTime()`     | Returns the current time                    |
| `Date()`        | Returns the date portion of a date time     |
| `DateDiff()`    | Calculates the difference between two dates |
| `Date_Add()`    | Highly flexible date arithmetic function    |
| `Date_Format()` | Returns a formatted date or time string     |
| `Day()`         | Returns the day portion of a date           |
| `DayOfWeek()`   | Returns the day of week for a date          |
| `Hour()`        | Returns the hour portion of a time          |
| `Minute()`      | Returns the minute portion of a time        |
| `Month()`       | Returns the month portion of a date         |
| `Now()`         | Returns the current date and time           |
| `Second()`      | Returns the second portion of a time        |
| `Time()`        | Returns the time portion of a date time     |
| `Year()`        | Returns the year portion of a date          |



- Example Input

```mysql
SELECT cust_id, order_num
FROM orders
WHERE order_date = '2005-09-01';
```

- Output 

```mysql
+---------+-----------+
| cust_id | order_num |
+---------+-----------+
|   10001 |     20005 |
+---------+-----------+
```

- **Analysis**

That `SELECT` statement worked; it retrieved a single order record, one with an `order_date`of `2005-09-01`.

But is using `WHERE order_date = '2005-09-01'` safe? `order_date` has a datatype of *datetime*. This type stores dates along with time values. The values in our example tables all have times of `00:00:00`, but that might not always be the case. What if order dates were stored using the current date and time (so you’d not only know the order date but also the time of day that the order was placed)? Then `WHERE order_date = '2005-09-01'` fails if, for example, the stored `order_date` value is `2005-09-01 11:30:05`. Even though a row with that date is present, it is not retrieved because the `WHERE` match failed.

The solution is to instruct MySQL to only compare the specified date to the date portion of the column instead of using the entire column value. To do this you must use the `Date()`function. `Date(order_date)` instructs MySQL to extract just the date part of the column, and so a safer `SELECT` statement is

- Example Input

```mysql
SELECT cust_id, order_num
FROM orders 
WHERE Date(order_date) = '2005-09-01';
```

> ### TIP
>
> **If You Mean Date Use Date().** It’s a good practice to use `Date()` if what you want is just the date, even if you know that the column only contains dates. This way, if somehow a date time value ends up in the table in the future, your SQL won’t break. Oh, and yes, there is a `Time()` function, too, and it should be used when you want the time.
>
> Both `Date()` and `Time()` were first introduced in MySQL 4.1.1.



- What if you wanted to retrieve all orders placed in September 2005? A simple equality test does not work as it matches the day of month, too. There are several solutions, one of which follows:

  - Example input 

  ```mysql
  SELECT cust_id, order_num
  FROM orders
  WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';
  ```

  - Example output 

  ```
  +---------+-----------+
  | cust_id | order_num |
  +---------+-----------+
  |   10001 |     20005 |
  |   10003 |     20006 |
  |   10004 |     20007 |
  +---------+-----------+
  ```
  - **Analysis**

    Here a `BETWEEN` operator is used to define `2005-09-01` and `2005-09-30` as the range of dates to match.

  - **Another Solution**

    ```mysql
    SELECT cust_id, order_num
    FROM orders
    WHERE Year(order_date) = 2005 AND Month(order_date) = 9;
    ```

    

### 11.3 Numeric Manipulation Functions 

**Table 11.3. Commonly Used Numeric Manipulation Functions**

| Function  | Description                                            |
| :-------- | :----------------------------------------------------- |
| `Abs()`   | Returns a number’s absolute value                      |
| `Cos()`   | Returns the trigonometric cosine of a specified angle  |
| `Exp()`   | Returns the exponential value of a specific number     |
| `Mod()`   | Returns the remainder of a division operation          |
| `Pi()`    | Returns the value of pi                                |
| `Rand()`  | Returns a random number                                |
| `Sin()`   | Returns the trigonometric sine of a specified angle    |
| `Sqrt()`  | Returns the square root of a specified number          |
| `Tan()`   | Returns the trigonometric tangent of a specified angle |
| `Floor()` | Returns floor value. Ex `SELECT FLOOR(RAND()*10)`      |
| `Ceil()`  | Returns floor value. Ex `SELECT CEIL(RAND()*10)`       |



## Chapter 12. Summarizing Data

 Supported functions 

**Table 12.1. SQL Aggregate Functions**

| Function  | Description                            |
| :-------- | :------------------------------------- |
| `AVG()`   | Returns a column’s average value       |
| `COUNT()` | Returns the number of rows in a column |
| `MAX()`   | Returns a column’s highest value       |
| `MIN()`   | Returns a column’s lowest value        |
| `SUM()`   | Returns the sum of a column’s values   |

### 12.1 The `AVG()` Function 

- Example: average of a specific column. 

```mysql
SELECT AVG(prod_price) AS avg_price
FROM products;
```

- Example: average of specific columns and rows 

```mysql
SELECT AVG(prod_price) AS avg_price 
FROM products
WHERE vend_id = 1003;
```



### 12.2 The `COUNT()` Function 

- Example: count rows in a table 

```mysql
SELECT COUNT(*) AS num_cust
FROM customers;
```

- Example: count rows with restrict: count customers with an email address 

```mysql
SELECT COUNT(cust_email) AS num_cust
FROM customers;
```



### 12.3 The `MAX()`  and `MIN()` Function 

- Returns the highest value in a specified column

```mysql
SELECT MAX(prod_price) AS max_price
FROM products;
```

> ### TIP
>
> **Using MAX() with Non-Numeric Data.** Although `MAX()` is usually used to find the highest numeric or date values, MySQL allows it to be used to return the highest value in any column including textual columns. When used with textual data, `MAX()` returns the row that would be the last if the data were sorted by that column.



- `MIN()` does the exact opposite of `MAX()`. It returns the lowest value in a specified column. 

```mysql 
SELECT MIN(prod_price) AS min_price
FROM products;
```



### 12.4 The `SUM()` Function 

`SUM()` is used to return the sum(total) of the values in a specific column. 

- Example 1: get total quantity of an order

```mysql
SELECT SUM(quantity) AS items_ordered
FROM orderitems
WHERE order_num = 20005;
```

- Example 2: get total price 

```mysql
SELECT SUM(item_price*quantity) AS total_price
FROM orderitems
WHERE order_num = 20005;
```



### 12.5 Combining Aggregate Functions 

- Example 

```mysql
SELECT COUNT(*) AS num_items
       MIN(prod_price) AS price_min,
       MAX(prod_price) AS price_max,
       AVG(prod_price) AS price_avg
FROM products;
```



## Chapter 13. Grouping Data

































