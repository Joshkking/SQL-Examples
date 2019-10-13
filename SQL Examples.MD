# SQL Examples

The following are some basic SQL Examples and notes taken during my Masters working in the MySQL dialect. Since, to me, much of the meaning of any give SQL syntax can be interpretted from the code itself, I'm not attempting to explain as much as provide self-interpretable examples, giving notes when necessary. I also don't intend to hit every function on its own but rather eventually cover most important examples along the way. lso this generally assume basic knowledge of terms and more serves as a set of syntax examples.

To practice the queries, I have been working with a db-fiddle page:
https://www.db-fiddle.com/f/7wyqDZguJojJBUNp3kWbEt/2

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

Note in this case there is no parenthesize. Where we used `MODIFY COLUMN` here, it was done to change the datatype of the `IsCurrent` field to boolean. You can also use `ALTER TABLE` to add or drop various constraints on fields.

### Check Constraint Example <a name="CHECK_constraint"></a>

You can also create table fields that only allow for values that pass a logical check. We see this in making our `Books` table below.

```sql
CREATE TABLE Books (
  BookID INT NOT NULL AUTO_INCREMENT,
  Name VARCHAR(255) NOT NULL,
  Type VARCHAR(10),
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
Additional rows can be added by adding another parenthesize line.

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
        ("Crook", "John", "Albertville", "OR", 0, "15th avenue"),
        ("Smith", "Leslie", "Smallville", "AL", 1, "24 Cross Creek Rd"),
        ("Russell", "Ben", "Newcut", "NM", 1, "11 Street"),
        ("Constantine", "Bert", "Mooresville", "TX", 0, "23 Johnsons Road"),
        ("Collins", "Susie", "Longrow", "AL", 1, "1200 12th Avenue"),
        ("Logan", "Rebecca", NULL, "TN", 0, "21 Circle St"),
        ("Smith", "John", "Smallville", "AL", 0, "150 Street");
        
INSERT INTO Books (Name, Type)
VALUES ("How to bake chez", "nonfiction"),
	("How to tame a wilderbeast", "fiction"),
        ("Where the wombats go", "fiction"),
        ("Scary things scarring scary things", "fiction"),
        ("The tale of Bob", "nonfiction"),
        ("Happy happy happy", "fiction"),
        ("'Nonsense' the king", "nonfiction"),
        ("Cooking soup without water", "nonfiction");
        
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
        (6, 6, '2010-07-22 12:00:00', '2010-08-22', 1),
        (6, 2, '2011-07-15 12:00:00', '2011-08-15', 1),
        (8, 1, '2019-08-12 12:00:00', '2019-09-12', 1),
        (7, 2, '2019-08-04 12:00:00', '2019-09-04', 1),
        (1, 9, '2019-09-09 12:00:00', '2019-10-09', 0),
        (5, 2, '2019-09-09 12:00:00', '2019-10-09', 1),
        (7, 3, '2019-10-03 12:00:00', '2019-11-03', 1),
        (7, 2, '2019-10-02 12:00:00', '2019-11-02', 1),
        (3, 1, '2019-10-09 12:00:00', '2019-11-09', 0),
        (2, 2, '2019-10-12 12:00:00', '2019-11-12', 0);
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

Parenthesize can be used to work with **combined operators** (typically when using OR) as needed:

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

Temporary names, or "aliases," can be given to columns or entire tables. This may be helpful in changing the way the results of a query are presented (i.e. to make them more human-readable) or for later referential use. In either case, we use the `AS` commands.

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
**However** the above use of aliases for tables creeps into the typical use case of joins. This style of querrying is being depreciated in some systems, and for a number of reasons, joins are generally preferrable. The more common alias use case then is for columns.

## Joins <a name="joins"><a/>

asdfasdf

### Advanced Data Selection

asdfsdfa

### Catching Duplicates <a name="duplicates"></a>

Groupby having

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
