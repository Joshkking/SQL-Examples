# SQL Examples

The following are some basic SQL Examples and notes taken during my Masters working in the MySQL dialect. Since, to me, much of the meaning of any give SQL syntax can be interpretted from the code itself, I'm not attempting to explain as much as provide self-interpretable examples, giving notes when necessary. I also don't intend to hit every function on its own but rather eventually cover most important examples along the way. lso this generally assume basic knowledge of terms and more serves as a set of syntax examples.

To practice the queries, I have been working with a db-fiddle page:
https://www.db-fiddle.com/f/7wyqDZguJojJBUNp3kWbEt/6

This page has all of the below database creation schema already included, so one can copy and paste the queries there to run. The examples provided below are from the context of this schema representing customers checking out different books. The data inserted is random, and the tables are not meant to be great examples. I simply added what I needed to learn the lanugage.

## Table of Contents
* [Basic Table Schema](#basic-table-schema-)
  * Note this section is based, first, on the creation of the fiddle schema and so may not follow an otherwise logical flow.
  * [Table Creation](#table-creation-)
  * [Data Types](#data-types-)
  * [Altering Tables](#altering-tables-)
  * [CHECK Constraint Example](#check-constraint-example-)
  * [Foreign Keys and Default Values](#foreign-keys-and-default-values-)
  * [Naming Constraints and Keys with Multiple Columns](#naming-constraints-or-keys-w-mult-columns-)
  * [Creating Indexes](#creating-indexes-)
  * [Inserting Values](#inserting-values-)
  * [Other Table/Database Commands](#other-noteworthy-tabledatabase-commands-)
* [Fiddle Schema](#schema-for-fiddle-)
  * The full table schema and inserted values for the created fiddle can be found here.
* [Basic Data Selection](#basic-data-selection-)
  * Note that what I consider "basic" and "advanced" data selection is entirely arbitrary (self-perceived) here.
  * [General Select Syntax](#general-select-syntax-)
  * [Only Unique Values](#only-unique-values-)
  * [Order By](#order-by-)
  * [Limiting the Number of Results](#limiting-number-of-results-)
  * [WHERE Examples](#where-examples-)
  * [String Searches via LIKE](#string-searches-via-like-)
  * [Working with Nulls](#working-with-nulls-)
  * [Basic Functions](#basic-functions-)
  * [Aliases](#aliases-)
* [Joins](#joins-)
  * [Inner Joins](#inner-join-)
  * [Left and Right (Outer) Joins](#left-and-right-outer-joins-)
  * [Full Outer Joins and Unions](#full-outer-joins-and-unions-)
  * [Multiple or Logical ON Conditions](#multiple-on-conditions-and-logic-)
  * [Self Joins](#self-joins-)
* [Advanced Data Selection]()
  * [GROUP BY and HAVING](#group-by-and-having-)
  * [EXISTS](#exists-)
  * [ANY, ALL](#any-all-)
* [CASE and Other Control Logic](#case-and-other-control-logic-)
* [Handy Functions](#handy-functions-)
* [Changing Table Data](#changing-data-and-tables-)
* [Catching Duplicates](#catching-duplicates-)
* [Stored Operations (Variables, Functions, etc.)](#stored-operations-variables-functions-etc-)
  * [Variables](#variables-)
  * [Stored Procedures](#stored-procedures-)
  * [Views](#views-)
  * [Triggers](#triggers-)

## Basic Table Schema <a name="table_schema"></a>

Note that the below syntax examples are not presented in perhaps the most oderly logical fashion. This is inentional however, as I'm basically working through the creation of the example fiddle while learning or noting concepts as they come up.

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

## Schema for Fiddle <a name="fiddle_schema"></a>

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

### Working with Nulls <a name="nulls"></a>

We can compare to the null case in logical expressions using `IS NULL` and `IS NOT NULL` as seen below:

```sql
SELECT * FROM Customers
WHERE City IS NULL;
```

Or...

```sql
SELECT * FROM Customers
WHERE City IS NOT NULL;
```

We can also use the `IFNULL()` function to return an alternative value if null. For instance:

```sql
SELECT
	FirstName,
    	IFNULL(City, 'MISSING CITY VALUE!') AS 'City'
FROM Customers;
```

We could also use the `COALESCE()` function which would return the first non-null value in the list/array (though its use cases may be less often).

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

As the `WHERE` clause does not evaluate aggregated column data (though it can still be used for other criteria in a group by), we have to use `HAVING` on aggregated values, though this command takes logical conditions in a similar manner.

For instance, say we wanted the count of customers in each state, but only for states with more than one customer (count > 1):

```sql
SELECT
	COUNT(CustomerID) AS `Number of Customers`,
    	State
FROM Customers
GROUP BY State
HAVING Count(CustomerID)>1;
```
### Exists <a name="EXISTS"></a>

`EXISTS` check whether or not a subquery returns any values at all, returns true if so, and thus prints the corresponding record. The general syntax is:

```sql
SELECT column_name(s)
FROM table_name
WHERE EXISTS
	(SELECT column_name FROM table_name WHERE condition)
```

The syntax here is not as intuitive as other setups. In this case, the `SELECT` from the subquery is effectively ignored as `EXISTS` is only interested if any data is returned or not. This evaluated per record of the column in the outer `SELECT` which is fed to the `WHERE EXISTS` clause for each evaluation.

For instance, say we wanted a list of all customer names that had actually checked out a book. We could use `WHERE EXISTS` to see if their customer ID actually exists in the checkouts table:

```sql
SELECT FirstName, LastName
FROM Customers
WHERE EXISTS (SELECT
              	# The column is abritrary as it is effectively ignored
              	CheckoutID
              FROM Checkouts
              WHERE Customers.CustomerID = Checkouts.CustomerID);
```

In the above, the `CustomerID` for each record in our outer query gets passed to the `WHERE EXISTS` subquery to use in its own `WHERE` statement which either returns true or false to determine whether values for the outer selction query are printed. One may notice the functionality here is similar to that of a sort of filtered join. While may `WHERE EXISTS` cases could be rewritten as joins, there are sometimes benefits for `EXISTS` instead:
 * `EXISTS` stops the moment it finds a match, so in some cases it may be quicker than a join.
 * There are different possible use cases since `EXISTS` returns a boolean.
 * If the join would result in duplicates, you might consider using `EXISTS` which will avoid this and case sometimes has different behavior than say a `SELECT DISTINCT` with a join (compare the results of merging two tables and filtering vs simply evaluating each record in one table as to whether it should be printed or discarded).

### Any, All <a name="having_any_all"></a>

`ANY` and `ALL` are used with `WHERE` or `HAVING` to compare values in a column to those in another array, that array *most often from a sub-query* (though some dialects allow you to write the array/list directly). The comparison will return true and print the values if the comparison is true for any values in the list in the case of `ANY`, or in the case of `ALL` when all the comparison is true for all values in the list.

While this isn't necessarily used often, it has a lot of utility as `ANY` and `ALL` work with any logical operator. Typical use cases might be returning values after a comparison of a column in one table to that of another table, or comparing aggregated values.

The general syntax is:

```sql
SELECT column_names
FROM table_name
WHERE column_name operator ANY # or ALL
	(SELECT column_name
	FROM table_name);
```

Queries constructed with `ANY` or `ALL` often have equivalent ways they can be constructed such as:
* In the case of testing for equivalency using `ANY`, the syntax `WHERE X = ANY(Y)` is equivalent to `WHERE X IN Y`.
* In the case of some other comparison using `ANY`, syntax can be equivalent to using `EXISTS` such as `WHERE X > ANY(Y)` could be written with `WHERE EXISTS(subquery testing X > Y)`.
* In the case of `ALL` this is equivalent to many `AND` statements within the logical comparison, one for each value in the array. For example, `WHERE X > ALL(Z)` is equivalent to `WHERE X > a AND X > b AND X > c...for the length of the array/subquery` assuming \[a, b, c, ...] comprise array/subquery `Z`.

**Some examples of `ANY` and `ALL` as well as their equivalencies (when appropriate) below:**

**`ANY` equivalency example**

Say we wanted all of our customer data for customers that have books that are not returned:

```sql
SELECT *
FROM Customers
WHERE CustomerID = ANY(SELECT CustomerID
			FROM Checkouts
                       	WHERE Returned=0);
```

This is equivalent to using `IN` instead:

```sql
SELECT *
FROM Customers
WHERE CustomerID IN (SELECT CustomerID
			FROM Checkouts
                       	WHERE Returned=0);
```

**`ANY` non-equivalency example**

Say we wanted all of our customer data where the customers are current (`Customers.IsCurrent` is 1), but they have books that aren't returned (`Checkouts.Returned` is 0). This then is true when `Customers.IsCurrent > Checkouts.Returned`, and we can use `ANY` to compare the values between the tables.

```sql
SELECT *
FROM Customers
WHERE IsCurrent > ANY (SELECT Returned
                       FROM Checkouts);
```

This is equivalent to using `EXISTS` as such:

```sql
SELECT *
FROM Customers
WHERE EXISTS (SELECT Returned
             	FROM Checkouts
            	WHERE Customers.IsCurrent > Checkouts.Returned);
```

**`ALL` example**

Say we wanted the longest name in our books table that's a fiction book. While the below is not likely the optimal SQL code for this query, it can be used as a functional example of using `ALL` in our data:

```sql
SELECT Name
FROM Books
WHERE LENGTH(Name) >= ALL(SELECT LENGTH(Name)
				FROM Books
				WHERE Type='fiction');
```

## Case and Other Control Logic <a name="CASE"></a>

SQL supports a number of control logic statements (if-then-else, loops, etc.). For instance, the general syntax for an if-then-else is:

```sql
IF condition THEN
   statements;
ELSEIF elseif-condition THEN
   elseif-statements;
ELSE
   else-statements;
END IF;
```
Note that the above is not the same as the MySQL `IF()` [function](https://www.w3schools.com/sql/func_mysql_if.asp).

These are particularly useful when creating stored procedures (discussed below). However, given the similarity of these control statements to the general structure in other languages, due to the standard presence across dialects of `CASE` and flexibility of use, we'll focus on `CASE` here. For information on other control statements, the [tutorial series starting here](http://www.mysqltutorial.org/mysql-if-statement/) is useful.

`CASE` works like an if-then-else statement, evaluating each record in the returned table and returning results accordingly. It can be used in different commands (such as `SELECT` or `ORDER BY`). Regardless of the use case, the general syntax is:

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE else_result
END;
```

`CASE` can be built into `SELECT` statements to create a new column whose values are contingent on data in the existing columns. For instance, say we wanted a new column that described which region each customer was in:

```sql
SELECT
	FirstName,
    	LastName,
    	State,
    	CASE
    		WHEN State IN ('AL', 'TN', 'FL') THEN 'Southeast'
        	WHEN State IN ('OR', 'WA') THEN 'Northwest'
        	WHEN State in ('NM', 'TX') THEN 'Southwest'
        	ELSE 'Region unknown'
    	END AS 'Customer Region'
FROM Customers;
```

`CASE` can also be used in `ORDER BY` functions (for instance, *if value is null, order by x, if not, order by y*), within functions, and many other uses. Note that regardless of the location it is used, `CASE` directly returns the value after `THEN` or `ELSE` to that keyword or function. Depending on the situation and SQL dialect, parentheses may need be added around the `CASE` clause for correct evaluation or simply for clarity of syntax.

## Handy Functions <a name="handy_functions"></a>

Below is a list of handy functions just for reference. It is by no means exhaustive (see [the docs](https://dev.mysql.com/doc/refman/8.0/en/func-op-summary-ref.html) for a longer list), and I only include explanatory notes here at my own need. The functions below may take param
* COUNT()
* MAX()
* MIN()
* AVG()
* SUM()
* SQRT()
* ABS()
* ROUND()
* STD()
* RAND() - to generate a random number
* CONCAT() - for joining strings together
* CURDATE()
* NOW() - timestamp
* DATEDIFF()
* EXTRACT() - extracts parts of a date, various other functions like HOUR() can be used as shortcuts
* LENGTH() - functions on strings
* LCASE() - convert to lower case (same as LOWER())
* UCASE() - convert to upper case (same as UPPER())

Many other helpful numeric, string, and date functions also exist.

## Changing Data and Tables <a name="changing_data"></a>

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
NOTE:  It's important to note the `WHERE` condition in the above. This determines which values get changed, so one must be used you're selecting exactly what you want before updating. Note that not all the column values must be provided in `SET`, only the ones you want to update.

We can **copy data from a table into a *new one*** using `SELECT INTO`:

```sql
SELECT column1, column2, column3, ...
INTO new_table # can add 'IN some_external_database' to this line if needed
FROM copied_table
WHERE condition;
```

And we can **copy data from one table into *another table*** using `INSERT INTO` a table, followed by a `SELECT`:

```sql
INSERT INTO table2 (column1, column2, ...)
SELECT column1, column2, ...
FROM table1
WHERE condition;
```
NOTE:  The `WHERE` clause determines which values to copy, not where to put them.

We can also **delete** existing data as such:

```sql
DELETE FROM table_name WHERE condition;
```
Just like in the update case, one should be very careful the `WHERE` clause resolves as intended.

## Catching Duplicates <a name="duplicates"></a>

As noted when discussing `SELECT DISTINCT`, `DISTINCT` is not sufficient to catch duplicates in a table where one column (such as an auto-incremented ID key) is different between the otherwise duplicate records. Generally, this would only occur in the case of user error or a table with poor validations. To catch such duplicates, we need a more complex query.

One method in MySQL (other methods may exist in different dialects) to look for duplicates is to group each record according to a column or columns that you believe should be distinct, select their aggregate counts, and only return those with aggregated counts higher than 1. A general syntax template for this follows:

```sql
SELECT 
	column1, COUNT(column1),
	column2, COUNT(column2)
	...other as needed...
FROM table_name
GROUP BY column1, column2 ...others as needed...
HAVING
	COUNT(column1) > 1 AND 
	COUNT(column2) > 1 AND 
	...others as needed...
```

For instance, we can look at the first name, last name, and address fields of our customers database as no customer should share all of these values (in our case):

```sql
SELECT
	FirstName, COUNT(FirstName),
    	LastName, COUNT(LastName),
    	Address, COUNT(Address)
FROM Customers
GROUP BY FirstName, LastName, Address
HAVING
	COUNT(FirstName) > 1 AND
    	COUNT(LastName) > 1 AND
    	COUNT(Address) > 1;
```

We could then review the count aggregates to make a decision on whether the records are truly different or not (and to see how many times they are duplicated). In this case, we would see there does indeed exist a duplicate in our table. We could then run another query to inspect the entries and finally remove one or more of the records as needed.

## Stored Operations (variables, functions, etc.) <a name="stored_operations"></a>

### Variables <a name="variables"></a>

MySQL does support the utilization of variables, of which there are 3 main types (generally not case-sensitive):
* **[User-defined "session" variables](https://dev.mysql.com/doc/refman/8.0/en/user-variables.html)** (prefix is `@`) - These variables are session and client-specific. That is, they are not "stored" on the server and only accessible by that user for the duration of the session.
  * Can be created as so:  `SET @var_name = ...`.
  * Session variables are not declared and so can be set to a value the moment they are initialized.
  * Variables are accessed as `@var_name` (that is, you still include the `@` to read them)
  * Can only take a limited set of datatypes: integer, decimal, floating-point, binary or nonbinary string, or NULL value.
  * You may see the `:=` operator on these which is needed to set their values inside of other commands (like `SELECT`)
* **[Local variables](https://dev.mysql.com/doc/refman/8.0/en/declare-local-variable.html)** (no prefix) - These variables are used within stored programs (for instance, a stored procedure or function) and are local to that program.
  * Can be created as so: `DECLARE var_name var_type DEFAULT value default_value`
  * The scope of local variables is the `BEGIN ... END` block they reside in.
* **[System variables](https://dev.mysql.com/doc/refman/5.5/en/using-system-variables.html)** (prefix is `@@`) - These are variables created and stored on the server itself and can be global or dependent on the session.
  * These are accessed as `@@var_name`
  * To see the current list of system variable values running on the server, you can use `SHOW VARIABLES;`
 
 ### Stored Procedures <a name="stored_procedures"></a>
 
Stored procedures are stored SQL code that can be saved for later use by calling it's name. The syntax for stored procedures can vary significantly between different dialects, but the basic MySQL syntax looks like:

```sql
DELIMITER //

CREATE PROCEDURE procedure_name
BEGIN
	...sql code goes here...
END//

DELIMITER ;
```
In the above, the general syntax is mostly intuitive, but the use of `DELIMITER` requires some explanation. In the `...sql code goes here...` section, one might input various commands using the standard delimiter `;`. In order to prevent these delimiters from running as normal within the `CREATE PROCEDURE` command, the delimiter is temporarily changed via `DELIMITER` to the value `//`. The delimeter is then changed back to the standard delimiter after the procedure has been created. The delimiter used above `//` is arbirtary and could be other values such as `$$` or `///`.

The stored procedure can then be executed using `CALL` (often "EXECUTE" in other dialects):

```sql
CALL procedure_name();
```
Note the parentheses in the above. As we'll see later, stored procedures can be passed parameters just like a traditional coding function.

If a procedure already exists, it can be modified in the same way it was created but using the `CREATE OR REPLACE PROCEDURE` command instead. `SHOW PROCEDURE STATUS;` can be used to get a list of procedures on the server and can take a number of [search cases](http://www.mysqltutorial.org/listing-stored-procedures-in-mysql-database.aspx).

#### Basic Example

Say we wanted a stored procedure that ran the concatenated first and last name of all of our customers. This could be created and called as:

```sql
DELIMITER //
CREATE PROCEDURE All_Customer_Names()
BEGIN
	SELECT
    		CONCAT(FirstName,' ',LastName) AS Name
	FROM Customers;
END //
DELIMITER ;

CALL All_Customer_Names();
```

#### Procedures with Parameters

Stored procedures can take parameter values from the calling program as well as return output values. As such, the parameter types for a stored procedure are:
* `IN` - A parameter taken in by the stored procedure
* `OUT` - A parameter returned by the stored procedure to the calling program
* `INOUT` - A combination of the two above. The stored procedure takes in a paremter, can modify the value of that parameter, and then returns the result
Parameters are defined within the parenthetical of a stored procedure as `IN/OUT/INOUT parameter_name parameter_datatype`. These only need to be used ad hoc to the situation at hand. The created parameters are referred to by their name only (with no preceding symbols like variable symbols).

For an `IN` example, say we wanted a stored function that return the cusomters table information but filtered down by state:

```sql
DELIMITER //

CREATE PROCEDURE Customers_From_State(
	IN state_name VARCHAR(2)
)
BEGIN
	SELECT *
	FROM Customers
    WHERE State=state_name;
END//

DELIMITER ;

CALL Customers_From_State('AL');
```

For an `OUT` example, say we wanted to ammend the above example to tell us the total in the given state:

```sql
DELIMITER //

CREATE PROCEDURE Total_From_State(
	IN state_name VARCHAR(2),
  	OUT total_count INT
)
BEGIN
	SELECT
    	COUNT(CustomerID)
		INTO total_count
	FROM Customers
    WHERE State=state_name;
END //

DELIMITER ;

CALL Total_From_State('AL',@total);
SELECT(@total);
```
Note that in the above, getting the return of the `OUT` parameter requires some steps. We need to give the `OUT` parameter a container to return to, so we note a session variable `@total` in the parameter list (since this is a session variable it can be initialized within the parameter list). We then `SELECT` that parameter to view the value. If we wanted to build the out value into the stored function results directly, we might consider a local variable (discussed below).

For an `INOUT` parameter example, we could imagine creating a procedure to add two numbers. This example is, of course, not useful but serves a basic example of the syntax:

```sql
DELIMITER //

CREATE PROCEDURE Add_Numbers(
  INOUT number INT,
  IN number_to_add INT
)
BEGIN
  SET number = number + number_to_add;
END//

DELIMITER ;

# Add 1 + 1, returns 2
SET @number = 1;
CALL Add_Numbers(@number, 1);
SELECT @number;
```

#### Procedures with Local Variables

Procedures can also declare and modify variables local to the procedure itself. These are used within the `BEGIN ... END` section of the procedure and are scoped to that section. Local procedure variables must be declared before they can be set, such as `DECLARE var_name var_type DEFAULT default_value;`. These variables can then be changed via the `SET` command, or one can select values into them as `SELECT ... INTO var_name`.

For instance, say we wanted a stored procedure to return the current customer total directly. In the below example, the local variable is not actually necessary (we could just run the query in the stored procedure to get the same result), but it serves as a simple syntax example. Declaring local variables would be useful or necessary for more complex querries.

```sql
DELIMITER //

CREATE PROCEDURE Current_Customer_Count()
BEGIN
    DECLARE current_count INT DEFAULT 0;
    
    SELECT COUNT(CustomerID) INTO current_count
    FROM Customers;
    
    SELECT current_count AS 'Current Customer Total';
END//

DELIMITER ;

CALL Current_Customer_Count();
```
Note that in the above, we `SELECT` the local variable as part of the stored procedure itself. So running this will return the result directly.

### Views <a name="views"></a>

Creating a "view" allows you to store a query with a given name, effectively creating a "virtual" table for future use. These have an advantage over stored procedures in that they can function as building blocks of other queries. For instance, on could `SELECT` from a view as, though "virtual", it functions just like a table. The results, however, are live (since it is technically the query logic, rather than the data, which is being stored). One use-case example might be creating a view that is the result of a join between two tables. Views might also be particularly helpful in simplifying the read of complex queries.

The general syntax is simply:

```sql
CREATE VIEW view_name AS
...query commands go here... ;
```

The created view can then be called from a simple query:

```sql
SELECT * FROM view_name;
```

Views can be dropped, renamed, and in some cases (though not discussed here) manually updated. To see all the views in a database, we can query the list of all tables, but filter for views;

```sql
SHOW FULL TABLES 
WHERE table_type = 'VIEW';
```

### Triggers <a name="triggers"></a>

A trigger is a stored program that is automatically run due to the onset of an event (such as changing data). For MySQL, triggers are supported for the `INSERT`, `UPDATE`, and `DELETE` events. The trigger can be "row-level" (it activates for every row the event operates on) or "statement-level" (operates only once per event, i.e. on the "statement"). However, MySQL currently only supports row-level triggers. Triggers are particularly handy for maintaining the integrity of your table as well as running automated tasks.

Most basic validations should be done via constraints at the table creation step (for instance `NOT NULL`); however, triggers allows for extended checks. They may increase response time on the server though, so one should be cautious in implementing them.

The general syntax for a trigger is:

```sql
CREATE TRIGGER trigger_name
BEFORE/AFTER INSERT/UPDATE/DELETE
ON table_name FOR EACH ROW
trigger_body;
```
Where `trigger_body` is the action to implement. The table columns are then know to the trigger body as `NEW` and `OLD` versions of the table when appropriate (i.e. the `DELETE` event would not have a `NEW` values for a row). For instance, the column would need to be referenced as `NEW.FirstName` and `OLD.FirstName`.

As an example, say we had a column in a table `LastUpdated`. We could use a trigger on the `UPDATE` event to ensure that the `LastUpdated` column is given the update timestamp when someone changes a value in that row via placing `SET LastUpdated = NOW();` within our trigger body. This could be useful as someone might change a separate field but forget to change the `LastUpdated` column. 
