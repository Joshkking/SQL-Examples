# SQL Examples

The following are some basic SQL Examples and notes taken during my Masters working in the MySQL dialect.

## Basic Table Creation

```sql
  CustomerID INT NOT NULL AUTO_INCREMENT, # Automatically adds alue at new record insertion
  LastName VARCHAR(255) NOT NULL,
  FirstName VARCHAR(255) NOT NULL,
  Gender VARCHAR(10),
  Address VARCHAR(255),
  City VARCHAR(255),
  State VARCHAR(2),
  Email VARCHAR(255),
  IsCurrent INT, # Will change this below
  PRIMARY KEY (CustomerID)
);
```
`AUTO_INCREMENT` does as expected - adding a new auto-incremented value for us at each new record insertion. This means we do not have to set the value ourselves.
  * Setting this = some number means it will start at that number in incrementing.
  * Other dialects use different language here, for instance `IDENTITY()` is used in SQL server.


Could also name a primary key from multiple columns (assuming they are unique and not null). This would look like:
```sql
CONSTRAINT PK_Customer PRIMARY KEY (CustomerID, LastName)
```
...where `PK_CUstomer` is the name of the composite key.

Handy list of SQL Datatypes(https://www.w3schools.com/sql/sql_datatypes.asp)

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
CONTSTRAINT FK_CustomerCheckout FOREIGN KEY (CustomerID, other columns...) REFERENCES Customers(CustomerID, othercolums...)
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

## Basic Data Selection
