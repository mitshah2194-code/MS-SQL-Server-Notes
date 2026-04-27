# MS-SQL-Server-Notes
/*******************
 * Topic 1 SQL Server — Basics
 * *****************/

/*
 * 1. Connecting to SQL Server using SQL Server Management Studio (SSMS)
 * Open SSMS → in Connect to Server dialog provide:

    Server type: Database Engine

    Server name: . or localhost (local), .\SQLEXPRESS (named instance), or hostname\instance

    Authentication: Windows Authentication (recommended in AD environments) or SQL Server Authentication (username/password)

    Click Connect. In Object Explorer you will see server → databases → security → etc.

    To run queries: New Query → choose database from the dropdown → write T-SQL → Execute (F5).
 */

/*
 * 2. Creating, Altering, Deleting Database in SQL Server
 * 
 * CREATE DATABASE CompanyDB
 *  -- optional file settings:
    -- ON (NAME = 'CompanyDB_Data', FILENAME = 'C:\SQLData\CompanyDB.mdf', SIZE = 20MB, MAXSIZE = 500MB, FILEGROWTH = 10MB)
    -- LOG ON (NAME = 'CompanyDB_Log', FILENAME = 'C:\SQLData\CompanyDB_log.ldf', SIZE = 10MB, FILEGROWTH = 10MB);
    Use / switch to a database
    USE CompanyDB;
    GO

    Alter database

    Change recovery model, max size, set options:

    ALTER DATABASE CompanyDB SET RECOVERY FULL;
    ALTER DATABASE CompanyDB MODIFY NAME = CompanyDB_NewName;


    Drop database

    -- Ensure no connections or put in single user mode:
    ALTER DATABASE CompanyDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE CompanyDB;
 */

/*
 * 3. Creating, Altering, Deleting Tables in SQL Server
 * CREATE TABLE dbo.Department (
    DepartmentId INT NOT NULL PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Location NVARCHAR(100) NULL
);
 */

/*
 * 4. Create table with identity & constraints
 * 
 * CREATE TABLE dbo.Employee (
    EmployeeId INT IDENTITY(1,1) PRIMARY KEY,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    DepartmentId INT NOT NULL,
    Salary DECIMAL(12,2) CONSTRAINT CK_Salary_Positive CHECK (Salary >= 0),
    CONSTRAINT FK_Employee_Department FOREIGN KEY (DepartmentId)
        REFERENCES dbo.Department(DepartmentId)
);
 */

/*
 * 5. Alter table

        Add column:

        ALTER TABLE dbo.Employee ADD Email NVARCHAR(255) NULL;


        Modify column type / nullability:

        ALTER TABLE dbo.Employee ALTER COLUMN Email NVARCHAR(320) NULL;
        -- To change NULL -> NOT NULL you must ensure existing rows have non-null values.


        Add constraint:

        ALTER TABLE dbo.Employee
        ADD CONSTRAINT UQ_Employee_Email UNIQUE (Email);


        Drop column or constraint:

        ALTER TABLE dbo.Employee DROP COLUMN Email;
        ALTER TABLE dbo.Employee DROP CONSTRAINT UQ_Employee_Email;


        Drop table

        DROP TABLE dbo.Employee;

 */

/*
 * 6. SQL Server Data Types

        Common categories and examples:

        Exact numeric: INT, BIGINT, SMALLINT, TINYINT, DECIMAL(p,s), NUMERIC(p,s)

        Approximate numeric: FLOAT, REAL

        Date & time: DATE, TIME, DATETIME2, DATETIMEOFFSET, SMALLDATETIME (legacy), DATETIME (legacy)

        Character: CHAR(n), VARCHAR(n), VARCHAR(MAX)

        Unicode character: NCHAR(n), NVARCHAR(n), NVARCHAR(MAX)

        Binary: BINARY(n), VARBINARY(n), VARBINARY(MAX)

        Others: BIT, UNIQUEIDENTIFIER (GUID), XML, JSON stored in NVARCHAR, SQL_VARIANT, GEOMETRY, GEOGRAPHY

        HierarchyId for hierarchical data (special type).

        Notes:

        Use NVARCHAR for multilingual text (UTF-16). VARCHAR stores single-byte codepages (or UTF-8 in newer SQL Server versions with special collations).

        DECIMAL(p,s) defines precision and scale (e.g., DECIMAL(12,2)).

        Prefer DATETIME2 over old DATETIME for accuracy & range.

        That's a great question about data types in SQL Server! 
        The main differences between VARCHAR, CHAR, and NVARCHAR lie in how they store data, whether they use fixed or variable length, 
        and the type of characters they can hold (single-byte vs. Unicode).Here is a breakdown of the three:
        📏 CHAR (Fixed-Length Non-Unicode)
        Storage Type: Fixed-length.
        Storage Size: $n$ bytes (where $n$ is the defined length). 
        If you define it as CHAR(10) and only store "Hi", it still uses 10 bytes of storage. 
        The remaining space is padded with spaces.
        Character Set: Non-Unicode (typically uses a single-byte encoding like ASCII/Latin-1). 
        This means it can generally store 256 different characters.Maximum Length: 8,000 characters.
        When to Use: When you know the data will always be a specific, fixed length (e.g., US State codes like 'CA', 'NY', or hash values).

        📈 VARCHAR (Variable-Length Non-Unicode)
        Storage Type: Variable-length.
        Storage Size: The actual length of the data plus 2 bytes to store the length information. 
                      If you define it as VARCHAR(10) and store "Hi" (2 characters), it uses 4 bytes (2 for data + 2 for length).
        Character Set: Non-Unicode (single-byte encoding). Same as CHAR.Maximum Length: 8,000 characters.When to Use: When the data's length varies significantly, 
                       and you are only storing standard English or Western European characters. This saves storage space compared to CHAR.Special 
        Note: VARCHAR(MAX) can store up to $2^{31}-1$ bytes (2 GB) of non-Unicode character data.

        🌎 NVARCHAR (Variable-Length Unicode)
        Storage Type: Variable-length.Storage 
        Size: The actual length of the data multiplied by 2 (since each character takes 2 bytes) plus 2 bytes for length information.
        Character Set: Unicode. This is the key difference. It uses 2 bytes per character, allowing it to store characters from almost all languages worldwide, 
                       including Chinese, Japanese, Arabic, etc. (up to 65,536 characters).
        Maximum Length: 4,000 characters (since each character takes 2 bytes, $4000 \times 2 = 8000$ bytes maximum size).
        When to Use: When you need to store global language characters (multilingual data) or any characters outside of the standard ASCII/Latin-1 set.Special 
        Note: NVARCHAR(MAX) can store up to $2^{30}-1$ bytes (1 GB) of Unicode character data.
 */

/*
 * 7. Constraints in SQL Server

        Constraints enforce rules on table columns:

        NOT NULL — value required.

        UNIQUE — unique values across rows.

        PRIMARY KEY — uniquely identifies rows (implies NOT NULL, UNIQUE).

        FOREIGN KEY — referential integrity to parent table.

        CHECK — arbitrary boolean expression (e.g., salary >= 0).

        DEFAULT — default value when not provided.

        IDENTITY — auto-increment behavior (special column property).

        Constraints can be named and added inline or with ALTER TABLE.

            CREATE TABLE dbo.Product (
      ProductId INT IDENTITY(1,1) PRIMARY KEY,
      SKU NVARCHAR(50) NOT NULL UNIQUE,
      Price DECIMAL(10,2) CONSTRAINT CK_Product_Price CHECK (Price >= 0),
      CreatedAt DATETIME2 NOT NULL DEFAULT (SYSUTCDATETIME())
    );

    Primary Key in SQL Server

        Enforces uniqueness and non-nullability.

        Can be clustered (default) or nonclustered.

        Usually created inline with table or via ALTER TABLE.

        -- composite primary key
        CREATE TABLE OrderItem (
          OrderId INT NOT NULL,
          ItemId INT NOT NULL,
          Quantity INT NOT NULL,
          PRIMARY KEY (OrderId, ItemId)  -- composite PK
        );


        Best practice: use a single-column surrogate INT/BIGINT or UNIQUEIDENTIFIER for PKs when convenient, but composite natural keys are fine when meaningful.

    Foreign Key in SQL Server

        Links child table column(s) to parent table primary (or unique) key.

        Enforces referential integrity: child values must exist in parent (unless NULL allowed).

        ALTER TABLE dbo.Employee
        ADD CONSTRAINT FK_Employee_Department
        FOREIGN KEY (DepartmentId) REFERENCES dbo.Department(DepartmentId);


        You can reference composite keys; child must include same columns and types.

    Primary Key and Foreign Key Relationship Between Multiple Tables

        Example scenario: Departments and Employees.

        CREATE TABLE dbo.Department (
          DepartmentId INT PRIMARY KEY,
          Name NVARCHAR(100) NOT NULL
        );

        CREATE TABLE dbo.Employee (
          EmployeeId INT IDENTITY(1,1) PRIMARY KEY,
          FirstName NVARCHAR(50),
          LastName NVARCHAR(50),
          DepartmentId INT NOT NULL,
          CONSTRAINT FK_Employee_Department FOREIGN KEY (DepartmentId)
            REFERENCES dbo.Department(DepartmentId)
        );


        This model allows joining: SELECT e.*, d.Name FROM dbo.Employee e JOIN dbo.Department d ON e.DepartmentId = d.DepartmentId;

        For many-to-many you need a junction table with two FKs:

        CREATE TABLE Project (
          ProjectId INT IDENTITY PRIMARY KEY,
          ProjectName NVARCHAR(200)
        );

        CREATE TABLE EmployeeProject (
          EmployeeId INT NOT NULL,
          ProjectId INT NOT NULL,
          AssignedDate DATE,
          PRIMARY KEY (EmployeeId, ProjectId),
          FOREIGN KEY (EmployeeId) REFERENCES Employee(EmployeeId),
          FOREIGN KEY (ProjectId) REFERENCES Project(ProjectId)
        );

    Cascading Referential Integrity Constraint in SQL Server

        Foreign keys support cascading actions: ON DELETE and ON UPDATE. Options:

        NO ACTION / RESTRICT (default behavior — prevents action if child rows exist)

        CASCADE — deletes/updates child rows automatically

        SET NULL — sets FK column to NULL (FK column must allow NULL)

        SET DEFAULT — sets to column default

        Example:

        CREATE TABLE Department (
          DepartmentId INT PRIMARY KEY,
          Name NVARCHAR(100)
        );

        CREATE TABLE Employee (
          EmployeeId INT IDENTITY PRIMARY KEY,
          DepartmentId INT,
          FirstName NVARCHAR(50),
          FOREIGN KEY (DepartmentId) REFERENCES Department(DepartmentId)
            ON DELETE SET NULL        -- if department deleted, employee.DepartmentId becomes NULL
            ON UPDATE CASCADE         -- if DepartmentId value changes, propagate to Employee
        );


        Important notes:

        ON UPDATE CASCADE is rarely used because surrogate keys normally are immutable.

        Avoid ON DELETE CASCADE unless cascading deletion makes sense (e.g., Order -> OrderItems).

        Large cascades can cause performance issues or accidental data loss; design carefully.

        Identity Column in SQL Server

        IDENTITY(seed, increment) automatically generates integer values at insert.

        CREATE TABLE dbo.Employee (
          EmployeeId INT IDENTITY(1,1) PRIMARY KEY, -- starts at 1, increments by 1
          FirstName NVARCHAR(50),
          LastName NVARCHAR(50)
        );

        INSERT INTO dbo.Employee (FirstName, LastName) VALUES ('Mit', 'Shah');
        SELECT SCOPE_IDENTITY();  -- returns the last identity value generated in current scope


        SCOPE_IDENTITY() is safer than @@IDENTITY (which can be affected by triggers).

        To insert explicit identity values use SET IDENTITY_INSERT dbo.Employee ON (only one table per session at a time) then insert with explicit value, then set OFF.

        Sequence Object in SQL Server

        Sequences are independent objects that generate numeric sequences; introduced in SQL Server 2012.

        CREATE SEQUENCE dbo.OrderSeq
          START WITH 1000
          INCREMENT BY 1
          MINVALUE 1000
          NO CACHE;  -- caching improves perf but may skip numbers on restart

        -- Get next value:
        SELECT NEXT VALUE FOR dbo.OrderSeq;

        -- Use in INSERT:
        INSERT INTO Orders (OrderNumber, ...) VALUES (NEXT VALUE FOR dbo.OrderSeq, ...);


        Sequences are more flexible than IDENTITY: usable across multiple tables, can set min/max, cycle, cache, and get next value in any scope.

        Difference Between Sequence and Identity in SQL Server

        Ownership & Scope

        IDENTITY is a column property bound to a single table.

        SEQUENCE is a server-scoped object independent of tables.

        Usage

        IDENTITY auto-generates only on INSERT to that table.

        SEQUENCE values can be retrieved anywhere via NEXT VALUE FOR and shared across tables.

        Control

        SEQUENCE allows ALTER SEQUENCE to change properties; IDENTITY seed/increment cannot be changed directly (you can reseed via DBCC CHECKIDENT).

        Retrieval of value

        For IDENTITY, use SCOPE_IDENTITY() or OUTPUT clause. For SEQUENCE, use NEXT VALUE FOR.

        Gaps

        Both can have gaps: IDENTITY when a transaction rolls back or deletions happen; SEQUENCE can have gaps when values cached and server restarts.

        When to choose

        Use IDENTITY for simple per-table auto-increment surrogate keys.

        Use SEQUENCE when you need shared sequences across tables, custom generation logic, or transactional control over number generation.

        Select Statement in SQL Server

        SELECT is the primary query statement to retrieve data.

        Basic

        SELECT FirstName, LastName FROM dbo.Employee;


        All columns

        SELECT * FROM dbo.Employee;


        Filtering

        SELECT * FROM dbo.Employee WHERE DepartmentId = 2 AND Salary > 50000;


        Sorting

        SELECT FirstName, LastName, Salary FROM dbo.Employee ORDER BY Salary DESC;


        Joins

        Inner join:

        SELECT e.EmployeeId, e.FirstName, e.LastName, d.Name AS DepartmentName
        FROM dbo.Employee e
        JOIN dbo.Department d ON e.DepartmentId = d.DepartmentId;


        Left join:

        SELECT e.EmployeeId, d.Name
        FROM Employee e
        LEFT JOIN Department d ON e.DepartmentId = d.DepartmentId;


        Aggregates & Grouping

        SELECT DepartmentId, COUNT(*) AS EmployeeCount, AVG(Salary) AS AvgSalary
        FROM dbo.Employee
        GROUP BY DepartmentId
        HAVING AVG(Salary) > 50000;


        Top / Offset-Fetch (pagination)

        -- top N
        SELECT TOP 10 * FROM dbo.Employee ORDER BY Salary DESC;

        -- offset-fetch (SQL Server 2012+)
        SELECT * FROM dbo.Employee ORDER BY EmployeeId
        OFFSET 50 ROWS FETCH NEXT 25 ROWS ONLY;


        Window functions

        SELECT EmployeeId, FirstName, Salary,
               ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RankBySalary,
               AVG(Salary) OVER (PARTITION BY DepartmentId) AS DeptAvgSalary
        FROM dbo.Employee;


        Insert-Select

        INSERT INTO EmployeeArchive (EmployeeId, FirstName, LastName, DepartmentId, Salary, ArchivedDate)
        SELECT EmployeeId, FirstName, LastName, DepartmentId, Salary, SYSUTCDATETIME()
        FROM Employee WHERE Terminated = 1;


        Output clause (capture inserted/updated/deleted rows)

        -- get inserted identity back and show
        INSERT INTO Employee (FirstName, LastName, DepartmentId)
        OUTPUT inserted.EmployeeId, inserted.FirstName
        VALUES ('A', 'B', 1);

        Additional Practical Tips & Best Practices

        Naming conventions: schema-qualified names dbo.TableName; name constraints (easy to manage when dropping).

        Use schema (e.g., dbo) rather than default to avoid ambiguity and improve perf.

        Indexing: create indexes to speed SELECTs; clustered index typically on PK. Avoid over-indexing writes.

        Transactions: use BEGIN TRAN / COMMIT / ROLLBACK when multiple DML statements must be atomic.

        Parameterize queries to avoid SQL injection (use stored procedures or parameterized client queries).

        Statistics & Maintenance: update stats, rebuild/reorganize indexes periodically for perf.

        Backups: configure regular backups (FULL/DIFFERENTIAL/LOG) and test restores.

        Use TRY...CATCH to handle runtime errors in T-SQL.

        BEGIN TRY
          BEGIN TRAN;
          -- statements
          COMMIT TRAN;
        END TRY
        BEGIN CATCH
          ROLLBACK TRAN;
          SELECT ERROR_MESSAGE() AS ErrorMessage;
        END CATCH;


        Use SET XACT_ABORT ON in some contexts to ensure transactions abort on certain errors.

        Short worked example (putting many pieces together)
        -- Create database and use it
        CREATE DATABASE DemoDB;
        USE DemoDB;

        -- Parent table
        CREATE TABLE Department (
          DepartmentId INT PRIMARY KEY,
          Name NVARCHAR(100) NOT NULL
        );

        -- Child table with identity, FK and CHECK
        CREATE TABLE Employee (
          EmployeeId INT IDENTITY(1,1) PRIMARY KEY,
          FirstName NVARCHAR(50) NOT NULL,
          LastName NVARCHAR(50) NOT NULL,
          DepartmentId INT NULL,
          Salary DECIMAL(10,2) CONSTRAINT CK_Employee_Salary CHECK (Salary >= 0),
          CONSTRAINT FK_Employee_Department FOREIGN KEY (DepartmentId)
            REFERENCES Department(DepartmentId) ON DELETE SET NULL
        );

        -- Insert and query
        INSERT INTO Department (DepartmentId, Name) VALUES (1, 'IT'), (2, 'HR');
        INSERT INTO Employee (FirstName, LastName, DepartmentId, Salary)
        VALUES ('Mit','Shah',1,50000), ('Asha','Patel',2,45000);

        SELECT e.EmployeeId, e.FirstName, e.LastName, d.Name AS DeptName
        FROM Employee e
        LEFT JOIN Department d ON e.DepartmentId = d.DepartmentId;

 */

/*
 *  Topic 2 - SQL Server – Clauses
 */

/*
 * 1. WHERE Clause in SQL Server
        ✅ Purpose

        Filters rows before any grouping happens.

        ✅ Syntax
        SELECT columns
        FROM table_name
        WHERE condition;

        ✅ Examples
        -- Get employees with salary > 50000
        SELECT * FROM Employees
        WHERE Salary > 50000;

        -- Multiple conditions
        SELECT * FROM Employees
        WHERE Department = 'IT' AND Salary >= 40000;

        ✅ Operators supported

        Comparison: =, !=, >, <, >=, <=

        Logical: AND, OR, NOT

        Others: IN, BETWEEN, LIKE, IS NULL

        SELECT * FROM Employees
        WHERE Name LIKE 'A%';

        2. ORDER BY Clause in SQL Server
        ✅ Purpose

        Sorts the result set in ascending or descending order.

        ✅ Syntax
        SELECT columns
        FROM table_name
        ORDER BY column_name [ASC | DESC];

        ✅ Examples
        -- Sort by salary ascending (default)
        SELECT * FROM Employees
        ORDER BY Salary;

        -- Sort by salary descending
        SELECT * FROM Employees
        ORDER BY Salary DESC;

        -- Multiple columns
        SELECT * FROM Employees
        ORDER BY Department ASC, Salary DESC;


        ✅ Default sorting is ASC (ascending).

        3. TOP N Clause in SQL Server
        ✅ Purpose

        Returns only the first N rows from the result set.

        ✅ Syntax
        SELECT TOP (N) columns
        FROM table_name;

        ✅ Examples
        -- Top 5 employees
        SELECT TOP 5 * FROM Employees;

        -- Top 5 highest paid employees
        SELECT TOP 5 * FROM Employees
        ORDER BY Salary DESC;

        ✅ With Percentage
        -- Top 10 percentage of rows
        SELECT TOP 10 PERCENT * FROM Employees;


        ✅ Important: Without ORDER BY, TOP returns random rows.

        4. GROUP BY Clause in SQL Server
        ✅ Purpose

        Groups rows that have the same values and allows use of aggregate functions.

        ✅ Common Aggregate Functions
        Function	Description
        COUNT()	Count rows
        SUM()	Total
        AVG()	Average
        MIN()	Smallest value
        MAX()	Largest value
        ✅ Syntax
        SELECT column, AGG_FUNC(column)
        FROM table_name
        GROUP BY column;

        ✅ Example
        -- Count employees per department
        SELECT Department, COUNT(*) AS TotalEmployees
        FROM Employees
        GROUP BY Department;

        5. HAVING Clause in SQL Server
        ✅ Purpose

        Filters groups created by GROUP BY.

        ✅ Syntax
        SELECT column, AGG_FUNC(column)
        FROM table_name
        GROUP BY column
        HAVING condition;

        ✅ Example
        -- Departments having more than 3 employees
        SELECT Department, COUNT(*) AS TotalEmployees
        FROM Employees
        GROUP BY Department
        HAVING COUNT(*) > 3;


        ✅ HAVING is used after GROUP BY.

        6. Difference Between WHERE and HAVING Clause
        Feature	WHERE	HAVING
        Filters	Rows	Groups
        Works with	Normal columns	Aggregate functions
        Executed	Before GROUP BY	After GROUP BY
        Uses aggregate functions	❌ No	✅ Yes
        ✅ Example using both together
        -- Employees with salary > 30000 and departments having more than 2 employees
        SELECT Department, COUNT(*) AS TotalEmployees
        FROM Employees
        WHERE Salary > 30000
        GROUP BY Department
        HAVING COUNT(*) > 2;

        Execution Order of Clauses in SQL Server

        Actual logical execution order:

        FROM

        WHERE

        GROUP BY

        HAVING

        SELECT

        ORDER BY

        TOP

        Quick Interview Notes ✅

        WHERE → filters rows

        GROUP BY → groups rows

        HAVING → filters groups

        ORDER BY → sorts result

        TOP → limits number of rows
 */

/*
 * Topic 3 - SQL Server Operators
 */

/*
 * 1. Assignment Operator in SQL Server
        ✅ Purpose

        Assigns values to variables in T-SQL.

        ✅ Syntax
        DECLARE @Salary INT;
        SET @Salary = 50000;       -- Assignment using SET

        DECLARE @Age INT;
        SELECT @Age = 25;          -- Assignment using SELECT

        ✅ Difference: SET vs SELECT
        Feature	SET	SELECT
        Assign multiple variables	❌	✅
        Return value if no row	NULL	Keeps old value
        Standard SQL compliant	✅	❌
        2. Arithmetic Operators in SQL Server

        Used for mathematical calculations.

        Operator	Description	Example
        +	Addition	Salary + 1000
        -	Subtraction	Salary - 500
        *	Multiplication	Salary * 2
        /	Division	Salary / 2
        %	Modulus (remainder)	Salary % 2
        ✅ Example
        SELECT Salary, Salary * 0.10 AS Bonus
        FROM Employees;

        3. Comparison Operators in SQL Server

        Used to compare values.

        Operator	Meaning
        =	Equal to
        != or <>	Not equal
        >	Greater than
        <	Less than
        >=	Greater than or equal
        <=	Less than or equal
        ✅ Example
        SELECT * 
        FROM Employees
        WHERE Salary >= 40000;

        4. Logical Operators in SQL Server

        Used to combine conditions.

        Operator	Description
        AND	All conditions must be true
        OR	Any condition must be true
        NOT	Negates a condition
        ✅ Example
        SELECT *
        FROM Employees
        WHERE Department = 'IT' AND Salary > 50000;

        5. IN, BETWEEN, and LIKE Operators
        ✅ IN Operator

        Checks if value matches any value in a list.

        SELECT * FROM Employees
        WHERE Department IN ('IT', 'HR');

        ✅ BETWEEN Operator

        Selects values within a range (inclusive).

        SELECT * FROM Employees
        WHERE Salary BETWEEN 30000 AND 60000;

        ✅ LIKE Operator

        Pattern matching with wildcards:

        Wildcard	Meaning
        %	Any number of characters
        _	Single character
        -- Names starting with A
        SELECT * FROM Employees
        WHERE Name LIKE 'A%';

        -- Names having 'mit' anywhere
        SELECT * FROM Employees
        WHERE Name LIKE '%mit%';

        6. ALL Operator in SQL Server
        ✅ Purpose

        Compares a value with all values in a subquery.

        ✅ Rules

        Condition must be true for all returned values.

        ✅ Example
        -- Employees earning more than all employees in HR
        SELECT *
        FROM Employees
        WHERE Salary > ALL (
            SELECT Salary FROM Employees WHERE Department = 'HR'
        );

        7. ANY Operator in SQL Server
        ✅ Purpose

        Returns true if any one value in the subquery satisfies the condition.

        ✅ Example
        -- Employees earning more than any employee in HR
        SELECT *
        FROM Employees
        WHERE Salary > ANY (
            SELECT Salary FROM Employees WHERE Department = 'HR'
        );

        8. SOME Operator in SQL Server
        ✅ Purpose

        SOME is synonym of ANY in SQL Server.

        ✅ Example
        SELECT *
        FROM Employees
        WHERE Salary > SOME (
            SELECT Salary FROM Employees WHERE Department = 'HR'
        );


        ✅ ANY and SOME behave exactly the same.

        9. EXISTS Operator in SQL Server
        ✅ Purpose

        Checks whether subquery returns at least one row.

        ✅ It returns only TRUE or FALSE.

        ✅ Example
        -- Customers who have placed at least one order
        SELECT *
        FROM Customers c
        WHERE EXISTS (
            SELECT 1 FROM Orders o
            WHERE o.CustomerId = c.CustomerId
        );


        ✅ Faster for large datasets because SQL Server stops searching after first match.

        10. UNION and UNION ALL Operators

        Used to combine result sets of two or more SELECT queries.

        ✅ Rules

        Same number of columns

        Compatible data types

        Same column order

        ✅ UNION (removes duplicates)
        SELECT Name FROM Employees
        UNION
        SELECT Name FROM Customers;

        ✅ UNION ALL (keeps duplicates – faster)
        SELECT Name FROM Employees
        UNION ALL
        SELECT Name FROM Customers;

        11. EXCEPT Operator
        ✅ Purpose

        Returns rows from the first query that are not present in the second query.

        ✅ Syntax
        SELECT Name FROM Employees
        EXCEPT
        SELECT Name FROM Customers;


        ✅ Works like: A – B

        12. INTERSECT Operator
        ✅ Purpose

        Returns only common rows from both queries.

        ✅ Syntax
        SELECT Name FROM Employees
        INTERSECT
        SELECT Name FROM Customers;


        ✅ Works like: A ∩ B

        13. Differences Between UNION, EXCEPT, and INTERSECT
        Feature	UNION	EXCEPT	INTERSECT
        Purpose	Combine rows	Compare difference	Find common rows
        Duplicate removal	✅ Yes	✅ Yes	✅ Yes
        Result logic	A + B	A – B	A ∩ B
        Supports duplicates	❌ (use UNION ALL)	❌	❌
        Visual Example
        Table A	Table B
        1	2
        2	3
        3	4
        Operator	Result
        UNION	1,2,3,4
        EXCEPT	1
        INTERSECT	2,3
        Interview Quick Notes ✅

        ALL → compares with all values

        ANY / SOME → compares with any value

        EXISTS → checks row existence

        UNION ALL is faster than UNION

        EXCEPT → returns differences

        INTERSECT → returns common values
 */

/*
 * Topic - 4 - SQL Server Joins
 */

/*
    1. What are JOINS in SQL Server?

        Joins are used to combine rows from two or more tables based on a related column.

        ✅ Why Joins are needed?

        Data normalization puts data in multiple tables.

        Joins help retrieve meaningful combined data.

        Sample Tables for Examples
        Employees
        EmpId	Name	DeptId	ManagerId
        1	Amit	10	NULL
        2	Neha	20	1
        3	Rahul	10	1
        4	Priya	NULL	NULL
        Departments
        DeptId	DeptName
        10	IT
        20	HR
        30	Finance
        2. INNER JOIN
        ✅ Purpose

        Returns only matching rows from both tables.

        ✅ Syntax
        SELECT e.EmpId, e.Name, d.DeptName
        FROM Employees e
        INNER JOIN Departments d
        ON e.DeptId = d.DeptId;

        ✅ Output

        Only employees who belong to a valid department.

        ✅ Interview Point: Shows intersection of both tables.

        3. LEFT OUTER JOIN
        ✅ Purpose

        Returns all rows from left table and matching rows from right table.

        If no match → NULL.

        ✅ Syntax
        SELECT e.EmpId, e.Name, d.DeptName
        FROM Employees e
        LEFT JOIN Departments d
        ON e.DeptId = d.DeptId;

        ✅ Output

        Includes all employees, even those without a department.

        ✅ Keyword OUTER is optional.

        4. RIGHT OUTER JOIN
        ✅ Purpose

        Returns all rows from right table and matching rows from left table.

        ✅ Syntax
        SELECT e.EmpId, e.Name, d.DeptName
        FROM Employees e
        RIGHT JOIN Departments d
        ON e.DeptId = d.DeptId;

        ✅ Output

        Includes all departments, even without employees.

        ✅ Less commonly used than LEFT JOIN.

        5. FULL OUTER JOIN
        ✅ Purpose

        Returns all records from both tables.

        Matched rows are merged.
        Unmatched rows show NULL.

        ✅ Syntax
        SELECT e.EmpId, e.Name, d.DeptName
        FROM Employees e
        FULL OUTER JOIN Departments d
        ON e.DeptId = d.DeptId;


        ✅ Combination of LEFT JOIN and RIGHT JOIN.

        6. SELF JOIN
        ✅ Purpose

        A table joins with itself.

        Used when a table has hierarchical or relational data within itself.

        ✅ Example: Employee–Manager Relationship
        SELECT 
            e1.Name AS Employee,
            e2.Name AS Manager
        FROM Employees e1
        LEFT JOIN Employees e2
            ON e1.ManagerId = e2.EmpId;


        ✅ Useful for organizational charts.

        7. CROSS JOIN
        ✅ Purpose

        Returns Cartesian product of both tables.

        Every row of the first table is combined with every row of the second table.

        ✅ Syntax
        SELECT e.Name, d.DeptName
        FROM Employees e
        CROSS JOIN Departments d;

        ✅ Row Count Formula:
        Rows = (Rows in Employees) × (Rows in Departments)


        ⚠ Use carefully because it can return very large result sets.

        Visual Join Summary
        Join Type	Description
        INNER JOIN	Only matched records
        LEFT JOIN	All left + matched right
        RIGHT JOIN	All right + matched left
        FULL JOIN	All rows from both tables
        SELF JOIN	Same table joined with itself
        CROSS JOIN	Cartesian product
        Interview Quick One-Liners ✅

        INNER JOIN → matching records only

        LEFT JOIN → all left table rows + matched right

        RIGHT JOIN → all right table rows + matched left

        FULL JOIN → all records from both tables

        SELF JOIN → table joins with itself

        CROSS JOIN → Cartesian product
 */

/*
 * Topic - 5 SQL Server Indexes
 */

/*
 *  ✅ What is an Index in SQL Server?

    An index is a database object that improves the speed of data retrieval (SELECT queries) by creating a structured lookup table.
    It works like the index of a book – instead of scanning every page, SQL Server can jump directly to the required data.

    Example (No Index – Slow Query)
    SELECT * FROM Employees WHERE Email = 'test@example.com';


    Without an index, SQL Server performs a table scan (checks every row).

    With Index – Faster Query
    CREATE INDEX IX_Employees_Email
    ON Employees(Email);

    ✅ Clustered Index

    A clustered index defines the physical order of data in a table.

    Key Points:

    Only one clustered index per table

    Data is stored in sorted order

    By default, Primary Key creates a clustered index

    Example:
    CREATE CLUSTERED INDEX CI_Employees_EmpId
    ON Employees(EmpId);

    When it’s fast:

    Range queries (BETWEEN, <, >)

    Sorting (ORDER BY EmpId)

    ✅ Non-Clustered Index

    A non-clustered index does not change physical data order.
    It creates a separate structure that contains:

    Indexed column value

    Pointer to actual data

    Key Points:

    Multiple non-clustered indexes can exist

    Requires extra storage

    Example:
    CREATE NONCLUSTERED INDEX NCI_Employees_Name
    ON Employees(Name);

    ✅ How Index Impacts DML Operations (INSERT, UPDATE, DELETE)

    Indexes speed up SELECT, but they slow down DML operations.

    Impact:
    Operation	Impact
    INSERT	Slower – index must be updated
    UPDATE	Slower if indexed columns are modified
    DELETE	Slower – index entries must be removed
    SELECT	Faster
    Example:

    If a table has 10 indexes, SQL Server must update 10 index structures during every INSERT.

    ✅ Unique Index

    A unique index ensures that no duplicate values exist in a column.

    Example:
    CREATE UNIQUE INDEX UI_Employees_Email
    ON Employees(Email);

    Difference from Primary Key:
    Unique Index	Primary Key
    Allows NULLs (one)	NOT NULL
    Enforces uniqueness	Enforces uniqueness + identity
    ✅ Index in GROUP BY

    Indexes can greatly improve GROUP BY performance, especially when the grouped column is indexed.

    Example:
    CREATE INDEX IX_Employees_Department
    ON Employees(DepartmentId);


    Query:

    SELECT DepartmentId, COUNT(*)
    FROM Employees
    GROUP BY DepartmentId;


    This avoids full table scan and performs index scan/seek.

    ✅ Advantages of Indexes

    ✅ Faster data retrieval
    ✅ Better query performance
    ✅ Improves sorting and grouping
    ✅ Efficient filtering

    ❌ Disadvantages of Indexes

    ❌ Slower INSERT/UPDATE/DELETE
    ❌ Extra storage space
    ❌ Requires maintenance (rebuild/reorganize)
    ❌ Can degrade performance if overused

    ✅ Differences: Clustered vs Non-Clustered Index
    Feature	Clustered	Non-Clustered
    Physical order of data	Yes	No
    Max allowed	1	Multiple
    Storage	Data stored with index	Separate structure
    Speed	Faster for range	Slightly slower
    ✅ Differences: UNION vs EXCEPT vs INTERSECT
    Feature	UNION	EXCEPT	INTERSECT
    Purpose	Combines results	Only rows in first but not second	Only common rows
    Removes duplicates?	Yes	Yes	Yes
    Similar to	OR	Difference	AND
    Example:
    -- UNION
    SELECT Name FROM Table1
    UNION
    SELECT Name FROM Table2;

    -- EXCEPT
    SELECT Name FROM Table1
    EXCEPT
    SELECT Name FROM Table2;

    -- INTERSECT
    SELECT Name FROM Table1
    INTERSECT
    SELECT Name FROM Table2;

    ✅ Quick Interview Tips

    You should remember:

    Clustered index = physical storage order

    Non-clustered = separate lookup structure

    PK → creates clustered index by default

    Unique index prevents duplicates

    Index speeds up SELECT but slows down DML
 */

/*
 * Topic - 6 - SQL Server Built in Functions
 */

/*
 *  ✅ Built-in String Functions in SQL Server

        String functions are used to manipulate text data.

        1. LEN() – Length of String
        SELECT LEN('SQL Server') AS Length;

        2. UPPER() – Convert to Uppercase
        SELECT UPPER('sql server') AS Result;

        3. LOWER() – Convert to Lowercase
        SELECT LOWER('SQL SERVER') AS Result;

        4. LTRIM() – Remove Leading Spaces
        SELECT LTRIM('   SQL') AS Result;

        5. RTRIM() – Remove Trailing Spaces
        SELECT RTRIM('SQL   ') AS Result;

        6. TRIM() – Remove Spaces from Both Sides (2017+)
        SELECT TRIM('   SQL   ') AS Result;

        7. SUBSTRING() – Extract Part of String
        SELECT SUBSTRING('SQL Server', 1, 3) AS Result;
        -- Output: SQL

        8. REPLACE() – Replace Text
        SELECT REPLACE('SQL Server', 'Server', 'Database') AS Result;

        9. CHARINDEX() – Find Position of a Substring
        SELECT CHARINDEX('Server', 'SQL Server') AS Position;

        10. CONCAT() – Join Strings
        SELECT CONCAT('SQL', ' ', 'Server') AS Result;

        ✅ OVER Clause in SQL Server

        The OVER() clause is used with window/analytic functions to perform calculations across rows without grouping them.

        Syntax:
        <function> OVER (PARTITION BY column ORDER BY column)

        Example:
        SELECT 
            EmpId,
            Department,
            Salary,
            AVG(Salary) OVER(PARTITION BY Department) AS AvgDeptSalary
        FROM Employees;


        ✅ OVER() does not reduce rows like GROUP BY.

        ✅ ROW_NUMBER() Function

        ROW_NUMBER() assigns a unique sequential number to each row.

        Syntax:
        ROW_NUMBER() OVER(ORDER BY column)

        Example:
        SELECT 
            ROW_NUMBER() OVER(ORDER BY Salary DESC) AS RowNum,
            EmpName,
            Salary
        FROM Employees;


        Use case:

        Pagination

        Finding duplicates

        ✅ RANK() Function

        RANK() assigns rank to rows but skips numbers when there are ties.

        Example:
        SELECT 
            EmpName,
            Salary,
            RANK() OVER(ORDER BY Salary DESC) AS RankNo
        FROM Employees;

        Example Output:
        Salary	Rank
        90000	1
        90000	1
        80000	3

        (skip rank 2)

        ✅ DENSE_RANK() Function

        DENSE_RANK() is similar to RANK() but does not skip numbers.

        Example:
        SELECT 
            EmpName,
            Salary,
            DENSE_RANK() OVER(ORDER BY Salary DESC) AS DenseRankNo
        FROM Employees;

        Example Output:
        Salary	Dense Rank
        90000	1
        90000	1
        80000	2

        (no skipping)

        ✅ Difference: ROW_NUMBER vs RANK vs DENSE_RANK
        Feature	ROW_NUMBER	RANK	DENSE_RANK
        Unique number	✅ Always	❌ Not always	❌ Not always
        Handles ties	❌ No	✅ Yes	✅ Yes
        Skips numbers	❌ No	✅ Yes	❌ No
        ✅ Practical Interview Query Examples
        1. Get 2nd highest salary
        SELECT *
        FROM (
            SELECT *,
                   DENSE_RANK() OVER(ORDER BY Salary DESC) AS dr
            FROM Employees
        ) X
        WHERE dr = 2;

        2. Remove duplicate records
        WITH CTE AS (
            SELECT *,
                   ROW_NUMBER() OVER(PARTITION BY Email ORDER BY Id) AS rn
            FROM Employees
        )
        DELETE FROM CTE WHERE rn > 1;

        ✅ Quick Interview Points

        OVER() is used with window functions.

        ROW_NUMBER() gives unique sequence.

        RANK() skips rank on ties.

        DENSE_RANK() does not skip.
 *
 */

/*
 *     Topic - 7 - User Defined Functions and Stored Procedure
 */

/*
 *  ✅ Difference: Stored Procedure vs User Defined Function
        Feature	Stored Procedure	Function (UDF)
        Can return multiple result sets	✅ Yes	❌ No
        Can modify database (INSERT, UPDATE, DELETE)	✅ Yes	❌ No
        Can use TRY…CATCH	✅ Yes	❌ No
        Can be used inside SELECT	❌ No	✅ Yes
        Return value	Optional	Mandatory
        ✅ Stored Procedure in SQL Server

        A stored procedure (SP) is a precompiled block of SQL statements stored in the database and executed using EXEC.

        Example: Simple Stored Procedure
        CREATE PROCEDURE spGetEmployees
        AS
        BEGIN
            SELECT * FROM Employees;
        END;

        Execute:
        EXEC spGetEmployees;

        ✅ Stored Procedure with Parameters
        CREATE PROCEDURE spGetEmployeeById
            @EmpId INT
        AS
        BEGIN
            SELECT * FROM Employees WHERE EmpId = @EmpId;
        END;


        Execution:

        EXEC spGetEmployeeById @EmpId = 1;

        ✅ SQL Server Stored Procedure Return Value

        Stored procedures can return an integer status code using RETURN.

        Example:
        CREATE PROCEDURE spCheckEmployee
            @EmpId INT
        AS
        BEGIN
            IF EXISTS (SELECT 1 FROM Employees WHERE EmpId = @EmpId)
                RETURN 1;
            ELSE
                RETURN 0;
        END;


        Calling it and capturing return value:

        DECLARE @Result INT;
        EXEC @Result = spCheckEmployee 5;
        SELECT @Result AS ReturnValue;

        ✅ Temporary Stored Procedure

        Temporary stored procedures exist temporarily in tempdb.

        Types:
        Type	Name	Scope
        Local Temp SP	#spTemp	Current session
        Global Temp SP	##spTemp	All sessions
        Example:
        CREATE PROCEDURE #spTempEmployees
        AS
        BEGIN
            SELECT * FROM Employees;
        END;

        ✅ Stored Procedure with ENCRYPTION and RECOMPILE
        1. WITH ENCRYPTION

        Hides the procedure definition from users.

        CREATE PROCEDURE spSecureData
        WITH ENCRYPTION
        AS
        BEGIN
            SELECT * FROM Employees;
        END;

        2. WITH RECOMPILE

        Forces SQL Server to recompile execution plan every time the SP runs.

        CREATE PROCEDURE spGetDynamicData
        WITH RECOMPILE
        AS
        BEGIN
            SELECT * FROM Employees WHERE Salary > 50000;
        END;


        ✅ Used when query parameters vary significantly (avoids parameter sniffing issues).

        ✅ Scalar Valued Function in SQL Server

        A scalar function returns a single value.

        Example:
        CREATE FUNCTION fnGetYear(@Date DATE)
        RETURNS INT
        AS
        BEGIN
            RETURN YEAR(@Date);
        END;


        Usage:

        SELECT dbo.fnGetYear(GETDATE()) AS CurrentYear;

        ✅ Inline Table-Valued Function (iTVF)

        Returns a table result using a single SELECT.

        Example:
        CREATE FUNCTION fnGetEmployeesByDept(@DeptId INT)
        RETURNS TABLE
        AS
        RETURN
        (
            SELECT EmpId, Name, Salary
            FROM Employees
            WHERE DepartmentId = @DeptId
        );


        Usage:

        SELECT * FROM dbo.fnGetEmployeesByDept(1);


        ✅ This is the fastest type of TVF.

        ✅ Multi-Statement Table-Valued Function (mTVF)

        Returns a table using multiple SQL statements.

        Example:
        CREATE FUNCTION fnEmployeeSummary(@DeptId INT)
        RETURNS @Result TABLE
        (
            EmpId INT,
            Name NVARCHAR(100),
            Salary MONEY
        )
        AS
        BEGIN
            INSERT INTO @Result
            SELECT EmpId, Name, Salary
            FROM Employees
            WHERE DepartmentId = @DeptId;

            RETURN;
        END;


        Usage:

        SELECT * FROM dbo.fnEmployeeSummary(1);

        ✅ Encryption and Schema Binding in Functions
        1. WITH ENCRYPTION

        Hides function definition.

        CREATE FUNCTION fnSecureSalary()
        RETURNS MONEY
        WITH ENCRYPTION
        AS
        BEGIN
            RETURN 50000;
        END;

        2. SCHEMABINDING

        Prevents changes to referenced tables.

        CREATE FUNCTION fnEmployeeCount()
        RETURNS INT
        WITH SCHEMABINDING
        AS
        BEGIN
            RETURN (SELECT COUNT(*) FROM dbo.Employees);
        END;


        ✅ Helps in creating indexed views and improves stability.

        ✅ Deterministic vs Non-Deterministic Functions
        ✅ Deterministic Functions

        Always return same output for same input.

        Examples:

        ABS(-10)   → 10
        ROUND(25.5,1) → 25.5

        ❌ Non-Deterministic Functions

        Return different values even with same input.

        Examples:

        GETDATE()
        NEWID()
        RAND()

        Differences Table
        Feature	Deterministic	Non-Deterministic
        Same output for same input	✅ Yes	❌ No
        Example	ABS(), POWER()	GETDATE(), NEWID()
        ✅ Interview Quick Summary

        You should remember:

        Stored procedures can modify data, functions cannot.

        Scalar functions return single value.

        TVFs return tables.

        Inline TVF is faster than multi-statement TVF.

        WITH ENCRYPTION hides code.

        WITH RECOMPILE rebuilds execution plan.

        Deterministic = same result always

        Non-deterministic = random/time-based
 */

/**
 *      Topic - 8 Exception Handling and Transaction Management
 *      
 *      That's a thorough list covering **Exception Handling and Transaction Management** in **SQL Server**. These topics are vital for maintaining data integrity and building robust database applications.

Here is a detailed breakdown of these concepts:

-----

## ⚙️ Transaction Management in SQL Server

A **Transaction** is a single unit of work that contains one or more SQL statements. The transaction must execute completely and successfully, or it must be rolled back completely.

### ACID Properties in SQL Server

SQL Server transactions strictly adhere to the four **ACID** properties, which guarantee database reliability:

1.  **Atomicity (All or Nothing):** The entire transaction must succeed as a single, indivisible unit. If any part fails, the entire transaction is rolled back, leaving the database unchanged.
2.  **Consistency (Valid State):** A transaction moves the database from one valid state to another. Any data written to the database must comply with all defined rules (constraints, triggers, cascades).
3.  **Isolation (Independent Execution):** Concurrent transactions must not interfere with each other. The result of simultaneous transactions should be the same as if they were executed sequentially. This is managed using **Isolation Levels** (e.g., Read Committed, Serializable).
4.  **Durability (Permanent):** Once a transaction is committed, its changes are permanent and survive system failures (e.g., power loss). This is ensured by logging the committed changes to non-volatile storage (disk).

### Types of Transactions in SQL Server

SQL Server supports three main types of transactions:

1.  **Implicit Transactions:** The first DML statement automatically starts a transaction, but it must be explicitly committed or rolled back. Set using `SET IMPLICIT_TRANSACTIONS ON;`.
2.  **Explicit Transactions (Recommended):** Transactions are explicitly started using `BEGIN TRANSACTION` and ended using `COMMIT TRANSACTION` or `ROLLBACK TRANSACTION`. This provides clear control over the unit of work.
3.  **Autocommit Transactions (Default):** Every individual SQL statement (DML, DDL) is treated as a separate transaction and is automatically committed upon success.

### Nested Transactions in SQL Server

Nested transactions refer to issuing a `BEGIN TRANSACTION` statement when another transaction is already active.

  * **Key Behavior:** SQL Server does **not** truly support nested transactions. When a transaction is already active, subsequent `BEGIN TRANSACTION` calls merely **increment a counter** tracked by the system variable **`@@TRANCOUNT`**.
  * **Commit/Rollback:**
      * **`COMMIT TRANSACTION`:** Decrements `@@TRANCOUNT`. The data changes are only made permanent when `@@TRANCOUNT` reaches **zero**.
      * **`ROLLBACK TRANSACTION`:** Rolls back **all** work associated with the outer transaction, regardless of the `@@TRANCOUNT` value.
  * **Savepoints:** You can create **Savepoints** (`SAVE TRANSACTION SavepointName`) to roll back part of the transaction without affecting the entire transaction.

<!-- end list -->

```sql
BEGIN TRANSACTION -- @@TRANCOUNT = 1
    -- ... SQL statements ...
    
    BEGIN TRANSACTION -- @@TRANCOUNT = 2
        -- ... More SQL ...
        SAVE TRANSACTION MySavepoint;
    COMMIT TRANSACTION -- @@TRANCOUNT = 1
    
    -- ROLLBACK TRANSACTION -- This rolls back everything and sets @@TRANCOUNT to 0.
COMMIT TRANSACTION -- @@TRANCOUNT = 0, changes are now permanent.
```

-----

## 🚨 Exception Handling in SQL Server

### Exception Handling Using Try Catch in SQL Server

Modern SQL Server (since SQL Server 2005) supports structured exception handling using **`TRY...CATCH`** blocks, similar to those in programming languages like C\#.

  * **`TRY` Block:** Contains the SQL statements to be executed. If any error occurs here, control is immediately transferred to the `CATCH` block.
  * **`CATCH` Block:** Contains the code to handle the error (e.g., logging the error, rolling back the transaction, or notifying the user).

**Best Practice for Transactions:** Always include logic in the `CATCH` block to check if a transaction is still active (`IF @@TRANCOUNT > 0`) and, if so, **roll it back**.

```sql
BEGIN TRY
    BEGIN TRANSACTION
    
    UPDATE TableA SET ColumnX = 'Bad Data' WHERE ID = 1; 
    
    COMMIT TRANSACTION
END TRY
BEGIN CATCH
    -- Check if a transaction is still open
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
        
    -- Re-raise the error for the calling application
    THROW;
END CATCH
```

### RaiseError and @@ERROR Function in SQL Server

These are older mechanisms for managing errors, largely superseded by `THROW`.

  * **`@@ERROR`:** A system function that returns the **error number** of the last Transact-SQL statement executed. A value of `0` means the previous statement executed successfully. This value is reset to `0` by the next statement, so it must be captured immediately.
  * **`RAISERROR`:** A statement used to generate an error message and transfer control to the calling application or a `CATCH` block. It allows for custom error messages and severity levels.
      * *Note:* It is kept primarily for backward compatibility and to support custom error messages stored in `sys.messages`.

### How to Raise Errors Explicitly in SQL Server

The modern and preferred way to raise an error explicitly in SQL Server is the **`THROW`** statement.

  * **`THROW` Statement:**

      * Stops the execution of the batch.
      * Transfers control to the nearest `CATCH` block.
      * It is compliant with the behavior of exceptions in the CLR.

  * **Syntax (Custom Error):**

    ```sql
    THROW Error_Number, Message, State;
    ```

      * `Error_Number`: Must be $> 50000$.
      * `Message`: The custom error message (string).
      * `State`: An integer from 0-255 used to denote the source or context of the error.

  * **Syntax (Re-throw):**

    ```sql
    THROW;
    ```

      * Used inside a `CATCH` block **without parameters** to re-raise the original error and preserve the original error information (line number, severity, etc.) for the caller. This is crucial for debugging.
 */

/**
 *  Topic - 9 Views and Triggers in SQL Server
 *  
 *  That's a comprehensive list covering **Views** (virtual tables) and **Triggers** (automated code) in SQL Server, along with user management. These features are fundamental for data security, integrity, and automation.

Here is a detailed breakdown of these concepts:

-----

## 👁️ Views in SQL Server

A **View** is a virtual table whose contents are defined by a query. It does not store data itself; instead, it retrieves data from the underlying tables every time it is queried.

### Advantages and Disadvantages of Views in SQL Server

| Advantage | Disadvantage |
| :--- | :--- |
| **Security:** Restrict user access to specific columns or rows, preventing direct table access. | **Performance:** Views based on complex joins or other views can be slow, as the underlying query must run every time. |
| **Simplicity:** Simplify complex queries (e.g., joins) into a single, easy-to-query entity. | **Update Restrictions:** Many complex views (e.g., those using `GROUP BY` or multiple joins) are non-updatable. |
| **Consistency:** Present a consistent structure to the user, even if the underlying table schema changes (as long as the view definition is updated). | **Hidden Complexity:** Users may treat views like fast tables, unaware of the complex queries running beneath, leading to performance issues. |

### Complex Views in SQL Server

A **Complex View** is generally defined as one that:

  * Involves **multiple tables** through joins.
  * Includes aggregate functions (`SUM`, `AVG`, `COUNT`) with a `GROUP BY` clause.
  * Uses set operators (`UNION`, `INTERSECT`).

Most complex views, especially those using aggregation or `DISTINCT`, are **not updatable**.

### Views with Check Option, Encryption, and Schema Binding in SQL Server

| Clause | Purpose | Behavior |
| :--- | :--- | :--- |
| **`WITH CHECK OPTION`** | Ensures that any data modifications made through the view adhere to the view's selection criteria (`WHERE` clause). | Prevents a row inserted/updated through the view from becoming invisible to the view afterward. Essential for data integrity. |
| **`WITH ENCRYPTION`** | Obfuscates the definition of the view in the system tables. | Prevents users from viewing the source code of the view (the underlying `SELECT` statement). |
| **`WITH SCHEMABINDING`** | Binds the view to the schema of the underlying tables. | **Prevents** the underlying tables from being modified (e.g., column dropped) or dropped entirely unless the view is dropped first. Improves reliability and is required for **Indexed Views**. |

### Indexed View in SQL Server

An **Indexed View** (or materialized view) is a view that has a **unique clustered index** created on it.

  * **Behavior:** When an index is created, SQL Server materializes the view's result set, storing the data on disk.
  * **Benefit:** Improves query performance significantly, as the database engine queries the pre-computed indexed result set rather than running the complex underlying query repeatedly.
  * **Requirement:** An Indexed View **must** be created with `WITH SCHEMABINDING`.

-----

## ⚙️ Triggers in SQL Server

A **Trigger** is a special type of stored procedure that executes automatically in response to certain events on a database or server.

### Types of Triggers in SQL Server

1.  **DML (Data Manipulation Language) Triggers:** Execute in response to `INSERT`, `UPDATE`, or `DELETE` statements on a table or view.
2.  **DDL (Data Definition Language) Triggers:** Execute in response to DDL statements (e.g., `CREATE TABLE`, `ALTER INDEX`, `DROP DATABASE`).
3.  **Logon Triggers:** Execute in response to the `LOGON` event before a user session is fully established.

### Inserted and Deleted Tables in SQL Server

Inside a **DML Trigger**, SQL Server makes two special logical tables available to hold the data being changed:

| Table | DML Operation | Content |
| :--- | :--- | :--- |
| **`Inserted`** | `INSERT`, `UPDATE` | Holds the new row(s) being inserted or the new values of the row(s) being updated. |
| **`Deleted`** | `DELETE`, `UPDATE` | Holds the old row(s) being deleted or the old values of the row(s) being updated. |
| **Note:** | `INSERT` only populates `Inserted`. `DELETE` only populates `Deleted`. `UPDATE` populates **both**. |

### DML Trigger Real-Time Examples in SQL Server

| Scenario | Trigger Type | Implementation Detail |
| :--- | :--- | :--- |
| **Auditing/Logging** | `AFTER INSERT, UPDATE, DELETE` | Capturing the old (`Deleted` table) and new (`Inserted` table) values along with the user, date, and time, and inserting this record into a separate Audit table. |
| **Enforcing Complex Rules** | `AFTER INSERT, UPDATE` | Checking if the value of a column is within a complex range based on other tables, which cannot be enforced by a simple `CHECK` constraint. |
| **Data Synchronization** | `AFTER UPDATE` | When a key product price is updated, automatically updating the calculated total cost in related `Order` tables. |

### Instead Of Trigger in SQL Server

An **`INSTEAD OF`** DML trigger executes **instead of** the triggering action (`INSERT`, `UPDATE`, or `DELETE`), skipping the original DML operation entirely.

  * **Primary Use Case:** Enabling updatability on **non-updatable views**. The trigger intercepts the DML statement aimed at the view and explicitly writes the correct, complex SQL to update the underlying base tables.

### DDL Triggers in SQL Server

**DDL Triggers** respond to Data Definition Language events (e.g., creating, dropping, or altering schema objects).

  * **Scope:** Can be scoped to the **entire server** (e.g., preventing anyone from dropping any database) or to a **specific database** (e.g., preventing anyone from creating tables during certain hours).
  * **Real-Time Example:** Preventing any user from executing `DROP TABLE` or `ALTER DATABASE` unless they belong to a specific administrative role.

### Triggers Execution Order in SQL Server

If you have multiple DML triggers defined on the same table for the same action (e.g., two `AFTER INSERT` triggers), SQL Server allows you to control the order in which they fire.

  * **Functions:** Use `sp_settriggerorder` to designate one trigger as the **`First`** and one as the **`Last`** to execute.
  * **Intermediate Triggers:** Any other triggers fire in an **undetermined order** between the first and last triggers.

-----

## 🔑 Managing Access

### Creating and Managing Users in SQL Server

Security involves creating **Logins** (server level) and mapping them to **Users** (database level).

1.  **Create Login:** A server-level identity that allows a person or application to connect to the SQL Server instance.
    ```sql
    CREATE LOGIN MyLogin WITH PASSWORD = 'StrongPassword'; 
    ```
2.  **Create User:** A database-level identity linked to a Login. The User is granted permissions within that specific database.
    ```sql
    USE MyDatabase;
    CREATE USER MyUser FOR LOGIN MyLogin;
    GRANT SELECT ON TableName TO MyUser;
    ```

### Logon Triggers in SQL Server

**Logon Triggers** fire in response to the `LOGON` event on the server. They execute **after** the authentication phase but **before** the session is fully established.

  * **Primary Use Case:** Implementing highly restrictive connection policies.
  * **Real-Time Example:**
      * Limiting the number of concurrent connections for a specific Login.
      * Restricting connections from a particular IP address or application name.
      * Rejecting connections during maintenance hours.
      * **Crucial Note:** If a Logon Trigger fails (throws an error), the logon process is **aborted**, and the user cannot connect.
 */

/**
 *  Topic - 10 - Advanced Concepts
 *  
 *  That's a great list of **Advanced SQL Server Concepts**, covering data organization, management, internal workings, and specialized reporting techniques.

Here is a detailed breakdown:

-----

## 📐 Database Normalization and De-Normalization

### Database Normalization in SQL Server

**Normalization** is the process of structuring a database to reduce data redundancy and improve data integrity. It involves dividing a database into two or more tables and defining relationships between them.

  * **Goals:**
      * Eliminate redundant data (data stored in multiple places).
      * Ensure data dependencies make sense (data stored in the correct table).
      * Protect data integrity during inserts, updates, and deletes.
  * **Normal Forms (NFs):** Normalization is typically applied up to the **Third Normal Form (3NF)**:
      * **1NF:** Eliminate repeating groups.
      * **2NF:** Eliminate partial dependencies (non-key attributes depend on only part of the primary key).
      * **3NF:** Eliminate transitive dependencies (non-key attributes depend on other non-key attributes).

### Database De-Normalization in SQL Server

**De-Normalization** is the process of intentionally introducing redundancy into a database by combining data from multiple tables into one.

  * **Purpose:** To improve the **read performance** of the database for reporting and high-volume retrieval systems.
  * **Trade-off:** It sacrifices write performance (slower inserts/updates due to redundancy) and increases data storage in exchange for faster reads (fewer joins required).
  * **Use Cases:** Data warehousing (Star Schema), reporting databases, and systems where read operations vastly outnumber write operations.

-----

## ⭐ Star Schema vs Snow Flake Schema in SQL Server

These are two primary models used in **Dimensional Modeling**, typically for data warehousing.

### Star Schema

  * **Structure:** A central **Fact Table** (containing measures and foreign keys) is surrounded by several **Dimension Tables** (containing descriptive attributes).
  * **Key Feature:** Dimension tables are **not normalized**; they link directly to the Fact Table.
  * **Advantages:**
      * Simpler structure, easier to understand and query.
      * Fewer joins, leading to fast query performance.

### Snow Flake Schema

  * **Structure:** An extension of the Star Schema where the dimension tables are **normalized**. This means dimensions can branch into sub-dimension tables.
  * **Key Feature:** Reduces redundancy within dimensions.
  * **Advantages:**
      * Less data redundancy within the dimensions.
      * Saves disk space (though minor compared to the total size).
  * **Disadvantages:**
      * More complex query paths due to multiple joins for dimension attributes.
      * Slower query performance compared to Star Schema.

-----

## 📅 How to Schedule Jobs in SQL Server using SQL Server Agent

**SQL Server Agent** is a Windows service that executes scheduled administrative tasks, known as **Jobs**.

  * **Job:** A specified series of actions executed sequentially by SQL Server Agent. A Job consists of:
    1.  **Steps:** The individual tasks (e.g., executing a T-SQL script, running an SSIS package, running an OS command).
    2.  **Schedules:** Defines when the job should run (e.g., daily at 2:00 AM, every hour).
    3.  **Alerts/Notifications:** Defines how the job status (success or failure) is reported (e.g., email notification).
  * **Use Cases:** Performing daily backups, running scheduled maintenance (rebuilding indexes), running nightly ETL processes, and generating routine reports.
  * **Prerequisites:** The **SQL Server Agent service** must be running.

-----

## 💾 How SQL Server Stores and Manages Data Internally

SQL Server manages data internally using a structured hierarchy:

1.  **Pages:** The fundamental unit of data storage. Every page is **8 KB** (8192 bytes). Data (rows), index entries, and allocation information are all stored in pages.
2.  **Extents:** A logical unit of storage consisting of **eight contiguous pages (64 KB)**. Extents are used to efficiently manage disk space allocation.
      * **Uniform Extents:** Owned by a single object (table or index).
      * **Mixed Extents:** Shared by up to eight different objects.
3.  **Files and Filegroups:**
      * **Data Files (.mdf, .ndf):** Physical files storing the database data and objects. The primary data file is `.mdf`; secondary data files are `.ndf`.
      * **Log Files (.ldf):** Store transaction log information.
      * **Filegroups:** A logical collection of one or more data files used for administrative or performance reasons (e.g., storing frequently accessed tables on fast SSDs).

-----

## 🔄 Change Data Capture (CDC) in SQL Server

**Change Data Capture (CDC)** is a feature that records INSERTs, UPDATEs, and DELETEs applied to SQL Server tables, and stores the details of these changes in special history tables.

  * **Mechanism:** CDC uses the **Transaction Log** to asynchronously capture data changes.
  * **Purpose:** Provides a reliable, low-impact way to track changes over time.
  * **Use Cases:**
      * **ETL Processes:** Loading only the incremental changes from a source system into a data warehouse (instead of reloading the whole table).
      * **Auditing:** Providing a history of modifications to data.
      * **Replication/Synchronization:** Keeping different systems synchronized with minimal latency.

-----

## 📊 PIVOT and UNPIVOT in SQL Server

### How to Implement PIVOT and UNPIVOT in SQL Server

**PIVOT** and **UNPIVOT** are relational operators used to transform tables for better reporting or analysis.

#### PIVOT (Rows to Columns)

  * **Function:** Rotates a table-valued expression by turning unique values from one column (the *pivot column*) into multiple columns in the output.
  * **Use Case:** Aggregating data and displaying summaries horizontally (e.g., displaying sales totals for each month as separate columns).

<!-- end list -->

```sql
SELECT *
FROM (
    SELECT Category, SalesAmount, SalesYear
    FROM SalesData
) AS SourceTable
PIVOT (
    SUM(SalesAmount)  -- Aggregate function
    FOR SalesYear     -- Pivot column
    IN ([2023], [2024]) -- Target columns
) AS PivotTable;
```

#### UNPIVOT (Columns to Rows)

  * **Function:** Performs the reverse of PIVOT, rotating the values from multiple columns into unique values in a single column.
  * **Use Case:** Returning data to a normalized or standard format for easier charting or consumption by other processes.

<!-- end list -->

```sql
SELECT Category, SalesYear, SalesAmount
FROM (
    SELECT Category, [2023], [2024] 
    FROM SalesPivot
) AS SourceTable
UNPIVOT (
    SalesAmount      -- Value column (data from 2023/2024 columns)
    FOR SalesYear    -- Pivot column (will hold '2023' or '2024')
    IN ([2023], [2024])
) AS UnPivotTable;
```

### Reverse PIVOT Table in SQL Server

The term **Reverse PIVOT** is essentially another way to describe the **UNPIVOT** operation. It means taking data that is already spread across multiple columns (a "pivoted" format) and transforming it back into a taller, narrower format where those column headers become data values in a single row. The standard T-SQL operator for this is **`UNPIVOT`**.
 */

/**
 *  Topic - 11 - Performance improvements in SQL Server queries
 *  
 *  Performance improvements in SQL Server queries are primarily achieved by optimizing **indexing**, minimizing data access, and ensuring the Query Optimizer uses the most efficient strategy.

Here is a breakdown of key techniques and concepts:

---

## 🔑 Indexing and Data Access Optimization

### Create Index on Proper Column to Improve Performance

Indexes are crucial for performance, acting like a sorted table of contents to quickly locate data rows.

* **Rule of Thumb:** Create **Non-Clustered Indexes** on columns frequently used in:
    * **`WHERE`** clauses (search arguments).
    * **`JOIN`** conditions.
    * **`ORDER BY`** and **`GROUP BY`** clauses.
* **Clustered Index:** Defines the **physical order** in which data rows are stored in the table. A table can only have **one** Clustered Index. It is typically created on the **Primary Key** or a heavily-queried, sequentially-accessed column (like an `OrderDate`).
* **Avoid Over-Indexing:** While indexes speed up reads (`SELECT`), they slow down writes (`INSERT`, `UPDATE`, `DELETE`) because the index structure must be maintained on every data modification.

### Performance Improvement using Unique Keys

The use of **Unique Keys** improves query performance because they guarantee data integrity and provide valuable information to the Query Optimizer.

* **Implicit Indexing:** Creating a `UNIQUE` constraint on a column automatically creates a **Unique Index** (usually Non-Clustered) on that column.
* **Optimizer Advantage:** When the Optimizer knows a column is unique, it can use the index to perform a highly efficient **Index Seek**, knowing it only needs to find one matching row. This is faster than having to check the entire table or range of the index to ensure no duplicates exist.

### When to Choose Table Scan and when to choose Seek Scan

The SQL Server Query Optimizer decides whether to perform a **Scan** or a **Seek** operation.

| Operation | Index Seek | Index Scan / Table Scan |
| :--- | :--- | :--- |
| **Description** | Uses the index structure (B-Tree) to navigate directly to the rows or data pages that satisfy the query predicate. | Reads all rows in the entire table or the entire index to find the qualifying rows. |
| **Cost Basis** | Proportional to the **number of qualifying rows and pages**. | Proportional to the **total number of rows/pages** in the table/index. |
| **When to Choose** | **Highly Selective Queries:** When a small percentage of rows (e.g., $<5\%-10\%$) qualify for the filter condition (e.g., `WHERE CustomerID = 123`). | **Non-Selective Queries:** When a large percentage of rows (e.g., $>20\%$) qualify, or when the query retrieves most/all rows (e.g., no `WHERE` clause). |
| **Performance** | Generally **much faster** for large tables. | Can be faster for **small tables** or when a high percentage of data is needed, as the overhead of index navigation is avoided. |
| **Execution Plan** | Indicates proper index usage. | Indicates missing or ineffective indexing. |

### How to Use Covering Index to reduce RID Lookup (or Key Lookup)

When a query uses a **Non-Clustered Index**, but needs columns that are **not** part of that index, the SQL Server must perform a secondary operation to retrieve the missing data.

* **The Lookup Problem:**
    * **RID Lookup:** Occurs if the table is a **Heap** (no Clustered Index). The Non-Clustered Index points to a **Row Identifier (RID)**, requiring a jump to the physical location of the row.
    * **Key Lookup:** Occurs if the table has a **Clustered Index**. The Non-Clustered Index points to the **Clustered Index Key**, requiring a jump to the Clustered Index leaf level to find the full row.
    * *Lookups are expensive, slow operations that involve extra I/O.*
* **Covering Index Solution:** A **Covering Index** is a Non-Clustered Index that includes **all** the columns required by a query (in the `SELECT` list, `WHERE` clause, and `JOIN` conditions).
    * **Implementation:** Use the **`INCLUDE`** clause to add non-search columns (those in the `SELECT` list) to the leaf level of the index without making them part of the B-Tree structure.
    * **Result:** The query can be entirely satisfied by scanning the Non-Clustered Index itself, eliminating the need for a costly RID or Key Lookup and drastically reducing I/O.

---

## 🛠️ Performance Tuning Tools

### Performance Improvement using Database Engine Tuning Advisor

The **Database Engine Tuning Advisor (DTA)** is a tool included with SQL Server that helps database administrators and developers 
analyze a database's workload (a set of queries) and recommend physical design structures to improve query performance.

* **How it Works:**
    1.  **Capture Workload:** You provide the DTA with a workload, typically a SQL script file, or by capturing activity using the 
SQL Server Profiler or **Query Store**.
    2.  **Analysis:** The DTA analyzes the workload against the database schema and current statistics.
    3.  **Recommendations:** It suggests modifications to the physical design, including:
        * Creating new **Indexes** (Clustered, Non-Clustered, and Indexed Views).
        * Creating or updating **Statistics**.
        * Implementing **Partitioning** strategies.
* **Benefit:** DTA provides a starting point for optimization by automatically identifying missing or suboptimal indexes, estimating 
* the performance improvement for each change, and generating the necessary T-SQL scripts.

---
