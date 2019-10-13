# SQL Examples

The following are some basic SQL Examples and notes taken during my Masters working in the MySQL dialect. Since, to me, much of the meaning of any give SQL syntax can be interpretted from the code itself, I'm not attempting to explain as much as provide self-interpretable examples, giving notes when necessary.

To practice the queries, I have been working with a db-fiddle page:
https://www.db-fiddle.com/f/7wyqDZguJojJBUNp3kWbEt/2
This page has all of the below database creation schema already included, so one can copy and paste the queries there to run. The examples provided below are from the context of this schema representing customers checking out different books. The data inserted is random, and the tables are not meant to be great examples. I simply added what I needed to learn the lanugage.

## Basic Table Schema

### Table Creation

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
```sql CONSTRAINT PK_Customer PRIMARY KEY (CustomerID, LastName) ``` ...where `PK_CUstomer` is the name of the created composite key.
* We also want `CustomerID` to be unique values. There is a `UNIQUE` constraint that can be applied just liked `NOT NULL`, but this automatically added to the field when we make it a primary key.

### Data Types

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

### Altering Tables

We can then alter an existing table like so:
```sql
ALTER TABLE Customers
  ADD DateOfBirth DATE,
  DROP COLUMN Email,
  MODIFY COLUMN IsCurrent BOOL;
```

```sql
CREATE TABLE Books (
  BookID INT NOT NULL,
  Type VARCHAR(10),
  PRIMARY KEY (BookID),
  CHECK (Type="fiction" OR Type="nonfiction")
); 
```
Constraings like `CHECK` can be given names in the same way way as the constraint in the primary key example above

Now create some checkouts

```sql
CREATE TABLE Checkouts (
  CheckoutID INT NOT NULL,
  BookID INT NOT NULL,
  CustomerID INT NOT NULL,
  CheckoutTime DATETIME DEFAULT NOW(), # Default value is current time
  DueDate DATE,
  PRIMARY KEY (CheckoutID),
  FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
  FOREIGN KEY (BookID) REFERENCES Books(BookID)
);
```
To name foreign keys or to add ones with multiple columns you can use the same technique for Primary key
```sql
CONTSTRAINT FK_CustomerCheckout FOREIGN KEY (CustomerID, other columns...)
REFERENCES Customers(CustomerID, othercolums...)
```
To change the default value of a column, add use `SET DEFAULT` in `ALTER TABLE`. Note that in the table above we can set a default to a function. Defaults can be dropped.


Some other useful table commands
```sql
DROP TABLE table_name # deletes the table
TRUNCATE TABLE table_name # deletes the data in the table
```

Indexes can also be used to increase lookup peformance (in the case of a regular index) or to help enforce unique values on columns (in the case of a unique inded).
```sql
CREATE INDEX Idx_LastName
ON Customers (LastName);
```
Multiple columns could have been added to the above, and it could have been converted to a unique index as `UNIQUE INDEX`. Indexes can be dropped.

### Inserting Values

Can be done as:

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
In either case, additional rows can be added by adding another parenthesize line. At least in MySQL, must use 1 because autoinc primary key

Data added into the fiddle stored here:
```sql

```

## Schema for the Fiddle

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
        ("Logan", "Rebecca", "Smallville", "TN", 0, "21 Circle St"),
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

## Basic Data Selection
