# SQL Examples

The following are some basic SQL Examples and notes taken during my Masters working in the MySQL dialect. Since, to me, much of the meaning of any give SQL syntax can be interpretted from the code itself, I'm not attempting to explain as much as provide self-interpretable examples, giving notes when necessary. I also don't intend to hit every function on its own but rather eventually cover most important examples along the way. lso this generally assume basic knowledge of terms and more serves as a set of syntax examples.

To practice the queries, I have been working with a db-fiddle page:
https://www.db-fiddle.com/f/7wyqDZguJojJBUNp3kWbEt/6

This page has all of the below database creation schema already included, so one can copy and paste the queries there to run. The examples provided below are from the context of this schema representing customers checking out different books. The data inserted is random, and the tables are not meant to be great examples. I simply added what I needed to learn the lanugage.

## Basic Table Schema <a name="table_schema"></a>

### Table Creation <a name="table_creation"></a>

Creating a table looks like:

```sql
CREATE TABLE Customers (
  CustomerID INT NOT NULL AUTO_INCREMENT,
  LastName VARCHAR(255) NOT NULL,
  FirstName VARCHAR(255) NOT NULL,
  Email VARCHAR(255),
  City VARCHAR(255),
  State VARCHAR(2),
  IsCurrent INT, # Will change this below
  PRIMARY KEY (CustomerID)
);
```

Some notes on the above table creation:
* `AUTO_INCREMENT` does as expected - adding a new auto-incremented value for us at each new record insertion. This means we do not have to set the value ourselves.
  * Setting this = some number means it will start at that number in incrementing.
  * Other dialects use different language here, for instance `IDENTITY()` is used in SQL server.
* We could also name a primary key from multiple columns (assuming they are unique and not null). This would look like:
`CONSTRAINT PK_Customer PRIMARY KEY (CustomerID, LastName)` where `PK_CUstomer` is the name of the created composite key.
* We also want `CustomerID` to be unique values. There is a `UNIQUE` constraint that can be applied just liked `NOT NULL`, but this automatically added to the field when we make it a primary key.

### Data Types <a name="data_types"></a>

There are a number of different data types that can be used when setting up fields, and they may differ in different SQL dialects. A handy list can be found on this [W3Schools link](https://www.w3schools.com/sql/sql_datatypes.asp). A few examples below:

* String types:
  * VARCHAR(size) - String of variable length where size in characters can be 0 to 65535
  * TEXT(size) - Holds a string where size in bytes can be up to 65,535 bytes
  * ENUM(val1, val2, ...) - A string object that can only have one values chosen from a list of possible values. If a value is inserted that's not in the list, then a blank is inserted.
  * SET(val2, val2, ...) - Like ENUM but allows for multiple values.
* Numeric types:
  * BOOL or BOOLEAN - Zero is considered false, nonzero values considered true. Note that MySQL currently converts this to an equivalent datatype on use for posterity reasons.
  * Different integer types - There are different integer types which various ranges acceptable. If you add a (size) parameter to any of these, this specifies the maximum display width (which is 255).
    * TINYINT - A very small integer with range -128 to 127.
    * INT - A medium sized integer (range -2147483648 to 2147483647).
    * BIGINT - Like INT but with range -9223372036854775808 to 9223372036854775807.
  * DOUBLE(size, d) - A floating point number with size as the number of digits and d as the number of digits after the decimal point.
  * DECIMAL(size, d) - Like double but an exact fixed-point number instead of a float.
* Date/Time types:
  * DATE - Format of YYYY-MM-DD
  * DATETIME - Format of YYYY-MM-DD hh:mm:ss
  * TIMESTAMP - Same format as DATETIME but different use case
  * TIME - Format hh:mm:ss
  * YEAR - Format YYYY

### Altering Tables <a name="altering_tables"></a>

We can then alter an existing table like so:

```sql
ALTER TABLE Customers
ADD Address VARCHAR(255),
DROP COLUMN Email,
MODIFY COLUMN IsCurrent BOOL;
```

Note in this case there is no parentheses. Where we used `MODIFY COLUMN` here, it was done to change the datatype of the `IsCurrent` field to boolean. You can also use `ALTER TABLE` to add or drop various constraints on fields.

### Check Constraint Example <a name="CHECK_constraint"></a>

You can also create table fields that only allow for values that pass a logical check. We see this in making our `Books` table below.

```sql
CREATE TABLE Books (
  BookID INT NOT NULL AUTO_INCREMENT,
  Name VARCHAR(255) NOT NULL,
  Type VARCHAR(10),
  SequelID INT, # Self-refferential, ID of book in this table that is listed book's sequel, if any
  PRIMARY KEY (BookID),
  CHECK (Type="fiction" OR Type="nonfiction")
); 
```

In other dialects, the `CHECK` constraint might be listed right after the variable. This effectively only allows books to be entered as type "fiction" or "nonfiction". It could, however, be handled via an `ENUM` datatype. One extra advantage of `CHECK` is that you can also add in extra columns into the logical check if needed.

### Foreign Keys and Default Values <a name="foreign_keys"></a>

We'll also create a table for different book checkouts. Given the relation algebra of this table, it will require multiple foreign keys.

```sql
CREATE TABLE Checkouts (
  CheckoutID INT NOT NULL AUTO_INCREMENT,
  BookID INT NOT NULL,
  CustomerID INT NOT NULL,
  CheckoutTime DATETIME DEFAULT NOW(), # Default value is current time
  DueDate DATE,
  Returned BOOL DEFAULT 0,
  PRIMARY KEY (CheckoutID),
  FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
  FOREIGN KEY (BookID) REFERENCES Books(BookID)
);
```

A few notes on the table above:
* Note that functions can be used within table constraints like `DEFAULT` above.
* To change the default value of a column, add use `SET DEFAULT` in `ALTER TABLE`. Defaults can also be dropped.
* We'll see how to make a composite key below.

### Naming Constraints or Keys w/ Mult Columns <a name="naming_constraints"></a>

To name keys or constraints, and/or to add ones with multiple columns, you can use the following syntax (example below of a composite foreign key):

```sql
CONTSTRAINT FK_CustomerCheckout FOREIGN KEY (CustomerID, other columns...) REFERENCES Customers(CustomerID, othercolums...)
```

`REFERENCES` is a pointer to the columns the listed foreign keys should associate with in another table. Of course in the case of other constraints we would use their appropriate commands rather than `FORIEGN KEY` or `REFERENCES`. For instance, a named `CHECK` constraint on multiple columns might look like. Assumed we have an `AquiredDate` for our books and occasionally check their status as `LastStatusCheck`:

```sql
CONSTRAINT Check_StatusDate CHECK (LastStatusCheck > AquiredDate)
```

This would ensure we could not add in status check dates that happened before the date we got the book.

### Creating Indexes <a name="creating_indexes"></a>

Indexes can also be used to increase lookup peformance (in the case of a regular index) or to help enforce unique values on columns (in the case of a unique index).

```sql
CREATE INDEX Idx_LastName
ON Customers (LastName);
```

Multiple columns could have been added to the above, and it could have been converted to a unique index as `UNIQUE INDEX`. Indexes can be dropped.

### Inserting Values <a name="inserting_values"></a>

Inserting values follows two general syntaxes:

1) Where the columns are explicitly named:
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```
2) Where the columns are not named (values for each column are provided):
```sql
INSERT INTO table_name
VALUES (value1, value2, value3, ...);
```
Additional rows can be added by adding another parentheses line.

The second case is often not used with tables having an `AUTO INCREMENT` column as we typically let the database handle that for us. As such, to use the second case we would need to be providing the values for that column directly since this method assumes we provide values for all columns.

For the data actually added to our fiddle schema, see the "Schema for Fiddle" section below.

### Other Noteworthy Table/Database Commands <a name="other_table_database"></a>

Some other useful table commands:

```sql
DROP TABLE table_name # deletes the table
TRUNCATE TABLE table_name # deletes the data in the table
SHOW TABLES # Shows all tables within the database
DESCRIBE table_name # shows information on all columns of a table
```

And some useful database commands:

```sql
# Creating a database looks like
CREATE DATABASE databasename;

# Show the databases on the server
SHOW DATABASES;

# Change to a different database
USE databasename;

# To backup a database
BACKUP DATABASE databasename
TO DISK = 'filepath';
```

From the [MySQL docs](https://dev.mysql.com/doc/refman/5.7/en/grant.html#grant-overview), the syntax for creating a user and granting privileges:

```sql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
GRANT SELECT ON db2.invoice TO 'jeffrey'@'localhost';
ALTER USER 'jeffrey'@'localhost' WITH MAX_QUERIES_PER_HOUR 90;
```
Some notes on the above:
* `%` Could be used instead of `localhost` to identify all valid hosts.
* The location of permission syntax is `database_name.table_name` so `db1.*` refers to all tables within `db1`.
* MySQL doesn't support wildcard for user names, so to grant to all users, leave the username section blank.
* Can add multiple privileges via commas, like `GRANT SELECT, INSERT ON ...`.

### Schema for Fiddle <a name="fiddle_schema"></a>

The full database schema used in the fiddle, presrved here, can be found below:

```sql
CREATE TABLE Customers (
  CustomerID INT NOT NULL AUTO_INCREMENT, # Automatically adds new value at record insertion
  LastName VARCHAR(255) NOT NULL,
  FirstName VARCHAR(255) NOT NULL,
  Email VARCHAR(255),
  City VARCHAR(255),
  State VARCHAR(2),
  IsCurrent INT, # Will change this below
  PRIMARY KEY (CustomerID)
);

ALTER TABLE Customers
	ADD Address VARCHAR(255),
	DROP COLUMN Email,
	MODIFY COLUMN IsCurrent BOOL;

CREATE TABLE Books (
  BookID INT NOT NULL AUTO_INCREMENT,
  Name VARCHAR(255) NOT NULL,
  Type VARCHAR(10),
  SequelID INT, # Self-refferential, ID of book in this table that is listed book's sequel, if any
  PRIMARY KEY (BookID),
  CHECK (Type="fiction" OR Type="nonfiction")
);  
    
CREATE TABLE Checkouts (
  CheckoutID INT NOT NULL AUTO_INCREMENT,
  BookID INT NOT NULL,
  CustomerID INT NOT NULL,
  CheckoutTime DATETIME DEFAULT NOW(), # Default value is current time
  DueDate DATE,
  Returned BOOL DEFAULT 0,
  PRIMARY KEY (CheckoutID),
  FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
  FOREIGN KEY (BookID) REFERENCES Books(BookID)
);

CREATE INDEX Idx_LastName
	ON Customers (LastName);

INSERT INTO Customers (LastName, FirstName, City, State, IsCurrent, Address)
VALUES ("Smith", "John", "Smallville", "AL", 0, "150 Street"),
	("Smith", "Elane", "Jordantown", "OR", 0, "23rd street"),
        ("Crook", "John", "Jordantown", "OR", 1, "15th avenue"),
        ("Smith", "Leslie", "Smallville", "AL", 1, "24 Cross Creek Rd"),
        ("Russell", "Ben", "Newcut", "NM", 1, "11 Street"),
        ("Constantine", "Bert", "Mooresville", "TX", 0, "23 Johnsons Road"),
        ("Collins", "Susie", "Longrow", "AL", 1, "1200 12th Avenue"),
        ("Logan", "Rebecca", NULL, "TN", 0, "21 Circle St"),
        ("Smith", "John", "Smallville", "AL", 0, "150 Street");
        
INSERT INTO Books (Name, Type, SequelID)
VALUES ("How to bake chez", "nonfiction", 2),
	("How to tame a wilderbeast", "fiction", 3),
        ("How to womp a wombat", "fiction", NULL),
        ("Scary things scarring scary things", "fiction", NULL),
        ("The tale of Bob", "nonfiction", NULL),
        ("Happy happy happy", "fiction", NULL),
        ("'Nonsense' the king", "nonfiction", NULL),
        ("Cooking soup without water", "nonfiction", NULL);
        
INSERT INTO Checkouts (BookID, CustomerID, CheckoutTime, DueDate, Returned)
VALUES (1, 3, '2008-11-01 12:00:00', '2008-12-01', 1),
	(4, 1, '2008-01-13 12:00:00', '2008-02-13', 0),
        (5, 2, '2008-12-13 12:00:00', '2009-01-13', 1),
        (2, 4, '2009-08-09 12:00:00', '2009-09-09', 1),
        (8, 4, '2009-07-25 12:00:00', '2009-08-25', 1),
        (8, 4, '2009-11-09 12:00:00', '2009-12-09', 1),
        (4, 9, '2009-11-15 12:00:00', '2009-12-15', 1),
        (2, 2, '2009-08-14 12:00:00', '2009-09-14', 1),
        (4, 2, '2009-12-09 12:00:00', '2010-01-09', 1),
        (4, 4, '2010-01-03 12:00:00', '2010-02-03', 1),
        (3, 3, '2010-02-09 12:00:00', '2010-03-09', 1),
        (7, 8, '2010-11-24 12:00:00', '2010-12-24', 0),
        (2, 6, '2010-07-22 12:00:00', '2010-08-22', 1),
        (3, 2, '2011-07-15 12:00:00', '2011-08-15', 1),
        (8, 1, '2019-08-12 12:00:00', '2019-09-12', 1),
        (7, 2, '2019-08-04 12:00:00', '2019-09-04', 1),
        (1, 9, '2019-09-09 12:00:00', '2019-10-09', 0),
        (5, 2, '2019-09-09 12:00:00', '2019-10-09', 1),
        (7, 3, '2019-10-03 12:00:00', '2019-11-03', 0),
        (7, 2, '2019-10-02 12:00:00', '2019-11-02', 1),
        (3, 1, '2019-10-09 12:00:00', '2019-11-09', 0),
        (2, 2, '2019-10-12 12:00:00', '2019-11-12', 0);;
```

## Basic Data Selection <a name="data_selection"></a>

### General Select Syntax <a name="SELECT"></a>

The general syntax for retrieving data can be as simple as `SELECT * FROM table_name` which pulls all (via `*`) columns from the given table. However, a fuller syntax generally takes the form:

```sql
SELECT Column1, Column2... FROM table_name
WHERE some_logical_condition
```

For instance in our fiddle, to select a few columns from our `Customers` table who live in Alabama:

```sql
SELECT CustomerID, FirstName, LastName, City, State FROM Customers
WHERE State='AL';
```

### Only Unique Values <a name="DISTINCT"></a>

To retrieve only the unique values within a column, we can use the append `DISTINCT` within our `SELECT` call. For instance, only the unique states in our `Customers` table:

```sql
SELECT DISTINCT State FROM Customers;
```

When used on multiple columns, this returns only distinct combinations:
```sql
SELECT DISTINCT FirstName, LastName FROM Customers;
```

`DISTINCT` also works with functions (discussed later) like `COUNT()` where it is written right in front of the column:

```sql
SELECT COUNT(DISTINCT State) FROM Customers;
```

However, care is needed when considering unique rows as, if not invalidated by the table schema, rows may be otherwise unique except for an ID column as is the case in our fiddle schema for our customers. In that table, there are two equivalent "John Smith"'s that would not be caught by a `DISTINCT` call on all rows because the CustomerID columns differ. One might try the below:

```sql
SELECT DISTINCT * FROM Customers;
```

Which here would not catch the noted case. Resolving this requires more complicated queries such as using `GROUPBY` and `HAVING` which will be covered later.

### Order By <a name="ORDER_BY"></a>

You can easily order the results of your query as seen below:

```sql
SELECT * FROM Checkouts
ORDER BY DueDate DESC, CheckoutTime ASC;
```

### Limiting Number of Results <a name="limiting_results"></a>
	
For a large table, you might be interested in limiting the number of results shown. This is handled differently in other dialects (such as the use of `TOP` in SQL Server and `ROWNUM` in Oracle), and other dialects may allow for percentages whereas MySQL does not (at least not simply). Nonetheless, a simple count limitation on the number of records returned in MySQL uses `LIMIT` as seen below:

```sql
SELECT * FROM Checkouts
LIMIT 10;
```

### WHERE examples <a name="WHERE"></a>

The `WHERE` clause accepts typical **logical operators (AND, OR, NOT)** as seen below:

```sql
SELECT * FROM Customers
WHERE State='AL' AND NOT City='Smallville';
```

Parentheses can be used to work with **combined operators** (typically when using OR) as needed:

```sql
SELECT * FROM Customers
WHERE State='AL' AND (City='Smallville' OR City='Longrow');
```

We can work with **dates and numeric values** in the expected way:

```sql
SELECT * FROM Checkouts
WHERE DueDate <='2019:09:30';
```
**Note:** Actual numeric values, like integers, do not require the quotes as is typical in many languages.

We can also work with **booleans** in the expected way:

```sql
SELECT * FROM Checkouts
WHERE Returned=True;
```

It also accepts a **between** statement:

```sql
SELECT * FROM Checkouts
WHERE DueDate BETWEEN '2018-10-31' AND '2019-10-31';
```

You can also use an **in** statement to search for values in a list:

```sql
SELECT * FROM Customers
WHERE State IN ('OR', 'TX', 'AL');
```

SQL also accepts `LIKE` for text operations which we'll cover below.

### String Searches via LIKE <a name="LIKE"></a>

You can use `LIKE` within a `WHERE` caluse to search specific patterns in strings similar to regex. Here `%` is a wildcard for any number of characters while `_` is a wildcard for just one character.

For instance, all first names beginning with "B":

```sql
SELECT * FROM Customers
WHERE FirstName LIKE 'B%';
```

Or an example below which will return all "John"'s but using wildcards:

```sql
SELECT * FROM Customers
WHERE FirstName LIKE 'J__n';
```

There are a number of different string search operators that can be used for more complicated queries. [Further details here in the docs](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html).

### Check for Nulls <a name="nulls"></a>

To work with missing values, we can utilize `IS NULL` and `IS NOT NULL` as seen below:

```sql
SELECT * FROM Customers
WHERE City IS NULL;
```

Or...

```sql
SELECT * FROM Customers
WHERE City IS NOT NULL;
```

### Basic Functions <a name="basic_functions"></a>

SQL dialects generally support a number of basic functions like `MIN`, `MAX`, `AVG`, and others that can be applied to columns within a `SELECT` clause. While not necessary, these can be given an "alias" using `AS`, which may be helpful for later references or for providing a more interpretable read in the query results. We'll discuss aliases more later.

The query below provides a set of metrics on our `Checkouts` table using a number of these functions as examples. As this query structure is somewhat more complicated, we begin using indents to clarify its structure:

```sql
SELECT
	MIN(CheckoutTime) AS Earliest_Checkout,
	MAX(DueDate) AS Farthest_Due_Date,
	AVG(Returned) AS Average_Return_Rate,
	SUM(Returned) AS Total_Returned_Checkouts,
	COUNT(DISTINCT CheckoutID) AS Total_Checkouts
FROM Checkouts;
```
The result is a new, single-row table-like result of the requested values.

Many other functions exist that can be utilized in use cases beyond this. This W3schools link provides a [general function list](https://www.w3schools.com/sql/sql_ref_mysql.asp).

### Aliases <a name="aliases"></a>

Temporary names, or "aliases," can be given to columns or entire tables. This may be helpful in changing the way the results of a query are presented (i.e. to make them more human-readable) or for later referential use. In either case, we use the `AS` command.

Basic column aliases have already been addressed when discussing functions, but showing an example with spaces:
```sql
SELECT
	BookID AS 'Book Number',
    	DueDate AS 'Due Date'
FROM Checkouts;
```
Note that in the above we use quote marks to for aliases with spaces. Other dialects might use \[square brackets].

However, it may also be helpful to use aliases when we create a derived value. One example of this is creating an "Address" column as below:
```sql
SELECT
	FirstName,
    	LastName,
    	CONCAT(Address,", ",City,", ",State) AS 'Full Address'
FROM Customers;
```

Table aliases work within the query, assigning the alias in `FROM` and then utilizing the alias within `SELECT`, `FROM`, and other elements of the query. For instance, we could use the following to get all of the book checkouts made by "John Smith". This would use two tables, so using aliases here might make the reference shorter (either way we do it for example below).

```sql
SELECT
	cu.FirstName,
    	cu.LastName,
    	ch.BookID
FROM
	Customers AS cu,
    	Checkouts AS ch
WHERE
	cu.FirstName='John' AND
    	cu.LastName='Smith' AND
    	cu.CustomerID=ch.CustomerID;
```
**However** the above use of querrying multiple tables creeps into the typical use case of joins. The style of querrying above is being depreciated in some systems, and for a number of reasons, joins are generally preferrable. As such, it would be more common to see table aliases in joins (discussed later) such as to shorten the code, or simply seeing column aliases as we noted above.

## Joins <a name="joins"><a/>

Joins are where a relational database really gets to take advantage of the relational algebra underlying its structure, combining records from multiple tables based on shared columns between them (often various keys). There are four "main" join types, though others exist and will be discussed later:
* **Inner Join** - Returns only records that have matches in both tables (analogous to the intersection of a venn diagram). In this case, one would expect few or minimal null values returned.
* **Left (Outer) Join** - Returns *all* records from the "left" table, regardless of whether there is a corresponding match on the right, as well as values from the "right" table where there is a match on the left (analogous to the left circle on a venn diagram including the part intersecting with the right). It should be noted that "left" and "right" are rather arbitrary here and only defined by as such by their relative location in the code syntax. In this case, one might expect a number of null values returned for records in the "left" table that do not have corresponding values in the "right" table.
* **Right (Outer) Join** - Like a Left Join, but just with swapped sides.
* **Full (Outer) Join** - Effectively, a merger of both tables, including all records from each that match the query. One would expect the most possible null values from this type of join. This join type is rarely used.

The general SQL algorithm is to `SELECT` columns from both databases simultaneously (noting the table where each comes from), include the first (or "Left") table in the `FROM` clause, provide the appropriate table to merge it with via `JOIN`, and finally specify the condition designating which columns should match values via the keyword `ON` (for instance the primary key in one table with the corresponding foreign key in another). Joins may also be coordinated with more than 2 tables which will also be noted below.

### Inner Join <a name="inner_join"></a>

The general structure of an inner join is:

```sql
SELECT table1.column_name, table2.column_name, ...
FROM table1
INNER JOIN table2
ON table1.column_name = table2.column_name;
```

For instance, if we wanted a list of book names (from our `Books` table) for every checkout in the `Checkouts` table, we could use an inner join like so:

```sql
SELECT Checkouts.CheckoutID, Checkouts.CheckoutTime, Books.Name
FROM Checkouts
INNER JOIN Books
ON Checkouts.BookID = Books.BookID;
```

If we also wanted the customer name for each checkout, then this requires us to join three tables together. The syntax generally remains the same:

```sql
SELECT
	Checkouts.CheckoutID,
    	Checkouts.CheckoutTime,
    	Books.Name,
    	Customers.FirstName,
    	Customers.LastName
FROM Checkouts
INNER JOIN Books ON Checkouts.BookID = Books.BookID
INNER JOIN Customers ON Checkouts.CustomerID = Customers.CustomerID;
```

However, in some dialects (like MS Access) parentheses are required to group each join with the original table in a sequential order, as below:

```sql
SELECT
	Checkouts.CheckoutID,
    	Checkouts.CheckoutTime,
    	Books.Name,
    	Customers.FirstName,
    	Customers.LastName
FROM ((Checkouts
INNER JOIN Books ON Checkouts.BookID = Books.BookID)
INNER JOIN Customers ON Checkouts.CustomerID = Customers.CustomerID);
```
It should be noted that the use of parentheses also provides extra utility, allowing us to force the code evaluator to implement the joins in the order setup by the parentheticals. This isn't particularly helpful here but might be needed in a case where inner joins were mixed with outer ones.

### Left and Right (Outer) Joins <a name="side_joins"></a>

Since a right join is effectively the same as a left join just with flipped sides, many common use cases can be addressed via a left join. As such (and given the similar syntax), we'll just address left joins here. In either case, the first table mentioned in the `FROM` clause becomes the "Left" table in the join.

For instance, let's say we wanted a list of every book name with every checkout, but we wanted to keep all of the books listed even if they had not yet been checked out, we could do:

```sql
SELECT Books.BookID, Books.Name, Checkouts.CheckoutID
FROM Books
LEFT JOIN Checkouts
ON Books.BookID = Checkouts.BookID;
```

This would return at least one record for *all* books with their corresponding Checkout ID's (if they exist) and multiple records for each book in the case they had been checked out multiple times. Running the above we can see that one book returns a `null` value for `CheckoutID` as it has not yet been checked out.

### Full (Outer) Joins and Unions <a name="full_joins_unions"></a>

While in MySQL, a full outer join is not supported, in other dialects it often looks similar to:

```sql
# NOTE: NOT SUPPORTED IN MYSQL
SELECT table1.column_name, table2.column_name, ...
FROM table1
FULL OUTER JOIN table2
ON table1.column_name = table2.column_name
```

Since this is not supported in MySQL, we might take advantage of the `UNION` operator instead. `UNION` allows us to append values from similar columns together (stacked vertically) from separate tables. When doing so:
* Each `SELECT` clause from each of the tables must ask for the same number of columns (since they will be stacked vertically).
* The corresponding selected columns should be in the same order in each `SELECT` clause.
* The columns should have similar data types (otherwise, even if the querry succeeds, you're not building a sensibly interpretable set).
When using `UNION`, the command automatically defaults to only returning *distinct* values. You can use `UNION ALL` to return non-distinct values as well.

Unions have the general form:
```sql
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```
So you might think of unions as *"run query 1 and query 2, and stack the results on top of each other."*

For instance, let's say we wanted a report of all things in our book checkouts setup we might should check back in on-- this might be customers who are no longer current (given by `IsCurrent` in our customers table) or books that weren't returned (given by `Returned` in our checkouts table). We could `UNION` these two sets together as so:

```sql
SELECT
	'Customer Not Current' AS 'Issue Type', 
    	LastName AS 'Problem Case' 
FROM Customers
WHERE IsCurrent=False
UNION ALL
Select 
	'Checkout Not Returned' AS `Issue Type`,
   	 CheckoutID AS 'Problem Case'
FROM Checkouts
WHERE Returned=False;
```
Some notes on the above:
* This will return a table with two columns- "Issue Type" and "Problem Case" with the corresponding results from both tables stacked vertically.
* The use of quotes for `Customer Not Current` and `Checkout Not Returned` are providing manual values to be returned in the query. For every new record given by the first `SELECT` there will be a value "Customer Not Current" in the `Issue Type` column to distinguish. The corrollary happens with the "Checkout Not Returned" values. This helps us differentiate the results from the two tables.
* The set of `AS` statements and aliases in the second `SELECT` are not necessary and might even cause issues if not written correctly. Since our values stack vertically, there are only two column names, and MySQL will take the colum name values from the aliases in the first `SELECT` just fine.

One can emulate a `FULL OUTER JOIN` in MySQL using `UNION ALL`. The manner is slightly tedious but effectively ammounts to a union of a left join and a right join with a `WHERE` clause to handle duplicates correctly. A template is given below:
```sql
SELECT * FROM t1
LEFT JOIN t2 ON t1.id = t2.id
UNION ALL
SELECT * FROM t1
RIGHT JOIN t2 ON t1.id = t2.id
WHERE t1.id IS NULL
```
Simply using `UNION` will not produce the same duplicate handling that `FULL OUTER JOIN` would-- hence our use of the `WHERE` statement to effectively build an exclusionary right join which does handle duplicates correctly. There are a number of posts digging into the logic of this online, so I'll resist dedicating more time here.

### Multiple ON Conditions and Logic <a name="join_multiple_ons"></a>

It is possible to perform joins on multiple columns via the use of `ON` *some condition* `AND` *some other condition*. A likely use case for this is when multiple keys are present. However, it should also be noted that the join-on conditions also accept logical operators. Since the multiple keys case is simply a matter of matching (and not as present in our database schema here), and the logical operators case presents more interesting patterns, we'll focus on the latter.

When using logical operators within `JOIN ON` conditions, two interesting behaviors come up. In the case of inner joins, the logical condition functions similar to a `WHERE` statement effectively filtering out matches that don't meet the `JOIN ON` condition.

For instance, consider a case where we wanted to review our checkouts table for any checkouts that aren't returned, but only when the customer is still active. Since, in our schema `Checkouts.Returned` and `Customers.IsCurrent` have values represented by 0's (false) and 1's (true), this logical condition could be written as `Customers.IsCurrent > Checkouts.Returned` (the customer is current, 1, which is greater than the book not being returned, 0). We could use multiple `JOIN ON` conditions to capture this:

```sql
SELECT
	CONCAT(Customers.FirstName,' ',Customers.LastName) AS Customer,
   	Checkouts.CheckoutID
FROM Customers
INNER JOIN Checkouts
ON
	Checkouts.CustomerID=Customers.CustomerID AND
    	Customers.IsCurrent>Checkouts.Returned;
```

...which filters down to only the records desired. The result is equivalent to the same statement with a `WHERE` clause instead:

```sql
SELECT
	CONCAT(Customers.FirstName,' ',Customers.LastName) AS Customer,
    	Checkouts.CheckoutID
FROM Customers
INNER JOIN Checkouts
ON
	Checkouts.CustomerID=Customers.CustomerID
WHERE
    	Customers.IsCurrent>Checkouts.Returned;
```

**BUT** this behavior is only consisten for inner joins. An outer join, such as a left join with multiple `JOIN ON`'s would still return all customers as seen when running the below:

```sql
SELECT
	CONCAT(Customers.FirstName,' ',Customers.LastName) AS Customer,
   	Checkouts.CheckoutID
FROM Customers
LEFT JOIN Checkouts
ON
	Checkouts.CustomerID=Customers.CustomerID AND
   	Customers.IsCurrent>Checkouts.Returned;
```

...where the returned output gives all customers but only fills in the values from our joined table when they meet both `JOIN ON` conditions. To filter it down to only the rows of interest in our given case, a `WHERE` clause would be needed.

Summing up, multiple `ON` clauses in joins have different behaviors for inner and outer joins. In inner joins, the `ON` clauses behave similar to `WHERE` conditions and filter down the results. In outer joins, the `ON` clauses determine which matches to fill in values
 on from the joined table (and a `WHERE` would be needed to filter down the records themselves). This makes sense as `ON` determines how to match values to fill in, whereas `WHERE` operates last to filter down the records. Inner joins only have similar behavior as they represent the intersection of the two tables at your given condition.

### Self Joins <a name="self_joins"></a>

Self Joins are a unique case join case where a table is joined with itself (via the use of naming the table with two different aliases). A self-join might be used in cases where data in a table references itself (for instance, hierarchical data such as employees and their managers) or when you want to compare values rows within the same table (for instance, grouping individuals from the same city).

For the syntax, there is no "self join" command. Rather existing selection techniques are used (contextual to the situation) using the table with two different aliases as noted above:

```sql
SELECT
	a.column_name,
	b.column_name, ...
FROM Table_Name AS a
LEFT/RIGHT/INNER JOIN Table_Name AS b
ON join_condition...
```

For an example of a self join utilizing hierarchical data, we can look at our books table which has the column `SequelID` which is self-referrential to this table. The value is this column links to the `BookID` of the book which is a sequel of the listed record. In this way, it can be used like a foreign key in a self-join to match up books with their sequels:

```sql
SELECT
	a.Name AS Book,
    	b.Name AS Sequel
FROM Books AS a
INNER JOIN Books AS b
ON a.SequelID=b.BookID;
```
In the above, we could change it to a left join to get all books regardless of whether or not they have a sequel, along with the name of their matching sequel.

For the use case of comparing rows to each other, let's assume we wanted to list every possible row combination of customers who live in a location with the same city name (we would need to add in state as well, but we're keeping it simple here). We could run a customers table self-join on the city column:

```sql
SELECT
	CONCAT(a.FirstName,' ',a.LastName) AS Customer1,
    	CONCAT(b.FirstName,' ',b.LastName) AS Customer2,
    	a.City
FROM Customers AS a
INNER JOIN Customers AS b
ON a.City=b.City AND a.CustomerID!=b.CustomerID
ORDER BY a.City, a.LastName, a.FirstName;
```
In the above, we add `A.CustomerID!=B.CustomerID` to remove matching a customer with his or her self. However, this still allows for matches from alias A -> alias B as well as alias B -> alias A, so we get some equivalent pairs. One way to remove this would be to write `A.CustomerID>B.CustomerID` instead which will limit out such cases.

## Advanced Data Selection <a name="advanced_selection"></a>

### Group By and Having <a name="GROUP_BY"></a>

`GROUP BY` allows us to group records together via sum aggregate function (like `COUNT`, `MIN`, `MAX`, etc.) and the values of a the column when group on. Without an aggregate function, the data would not be able to be represented by a simple table (it would require some higher-dimensional structure), so an aggregate function is expected.

For instance, say we wanted a count of how many of our customers were in each state:

```sql
SELECT
	COUNT(LastName) AS `Number in State`, 
    	State
FROM Customers
GROUP BY State;
```
...which returns a State column and the aggregated count of the number of customers in each state in a separate column.

`GROUP BY` also can work with joins. For instance, presume we wanted the number of checkouts by each customer. Since we will group by values on the customers table and are using a left join with it first, we can select multiple columns from it. The records from the checkout table is what is aggregated:

```sql
SELECT
    	Customers.FirstName,
   	Customers.LastName,
	COUNT(Checkouts.CheckoutID) AS `Number of Checkouts`
FROM Customers
LEFT JOIN Checkouts
ON Checkouts.CustomerID=Customers.CustomerID
GROUP BY Customers.CustomerID;
```

As the `WHERE` clause does not work with aggregate functions, we have to use `HAVING`, though the command takes logical conditions in a similar manner.

For instance, say we wanted the count of customers in each state, but only for states with more than one customer (count > 1):

```sql
SELECT
	COUNT(CustomerID) AS `Number of Customers`,
    	State
FROM Customers
GROUP BY State
HAVING Count(CustomerID)>1;
```

### Any, All <a name="having_any_all"></a>

kkkjkjkj

### Catching Duplicates <a name="duplicates"></a>

Groupby having

### Handy Functions <a name="handy_functions"></a>

list of ones I think useful

## Changing Data <a name="changing_data"></a>

We have already seen how to **insert data** above when building our schema like below:

```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

But we can also **update** existing data as so:

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```
It's important to note the `WHERE` condition in the above. This determines which values get changed, so one must be used you're selecting exactly what you want before updating. Note that not all the column values must be provided in `SET`, only the ones you want to update.

We can also **delete** existing data as such:

```sql
DELETE FROM table_name WHERE condition;
```
Just like in the update case, one should be very careful the `WHERE` clause resolves as intended.
