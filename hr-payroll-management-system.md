# HR & Payroll Management System

## Project Overview

This project demonstrates the design and implementation of a relational **HR and Payroll Management System** using SQL.

The system manages:

- Employee records
- Department structures
- Payroll processing
- Salary calculations
- Automated payroll actions
- Reporting and analysis
- Database optimization

The goal is to create a structured database solution that improves HR operations, reduces manual payroll processing, and enables data-driven reporting.

---

# Database Design

The database consists of three main entities:

| Table | Purpose |
|---|---|
| `Employees` | Stores employee information and employment status |
| `Departments` | Stores department details |
| `Payroll` | Stores employee salary transactions |

---

# Table Creation

## Departments Table

Stores company department information.

```sql
CREATE TABLE Departments (

    DepartmentID INT PRIMARY KEY IDENTITY(1,1),

    DepartmentName NVARCHAR(50) UNIQUE NOT NULL

);
```

---

## Employees Table

Stores employee details including salary and employment status.

```sql
CREATE TABLE Employees (

    Emp_ID INT PRIMARY KEY IDENTITY(1,1),

    FirstName NVARCHAR(50) NOT NULL,

    LastName NVARCHAR(50) NOT NULL,

    Email NVARCHAR(100) UNIQUE NOT NULL,

    PhoneNumber NVARCHAR(20),

    DepartmentID INT NOT NULL,

    HireDate DATE DEFAULT GETDATE(),

    Salary DECIMAL(10,2) DEFAULT 50000,

    Status NVARCHAR(20) DEFAULT 'Active',

    CONSTRAINT FK_Department

    FOREIGN KEY (DepartmentID)

    REFERENCES Departments(DepartmentID)

);
```

---

## Payroll Table

Tracks employee salary payments.

```sql
CREATE TABLE Payroll (

    PayrollID INT PRIMARY KEY IDENTITY(1,1),

    Emp_ID INT NOT NULL,

    PayDate DATE DEFAULT GETDATE(),

    BaseSalary DECIMAL(10,2) NOT NULL,

    Bonus DECIMAL(10,2) DEFAULT 0,

    TotalSalary AS (BaseSalary + Bonus),

    CONSTRAINT FK_Payroll_Emp

    FOREIGN KEY (Emp_ID)

    REFERENCES Employees(Emp_ID)

);
```

---

# Automated Payroll Processing

## Stored Procedure

A stored procedure was created to process payroll for all active employees.

The procedure:

1. Retrieves active employees
2. Calculates bonuses
3. Inserts payroll records
4. Uses transactions to maintain data integrity

```sql
CREATE PROCEDURE ProcessPayroll

AS

BEGIN

    DECLARE 
        @EmpID INT,
        @BaseSalary DECIMAL(10,2),
        @Bonus DECIMAL(10,2);


    DECLARE payroll_cursor CURSOR FOR

    SELECT Emp_ID, Salary
    FROM Employees
    WHERE Status = 'Active';


    BEGIN TRANSACTION;


    BEGIN TRY

        OPEN payroll_cursor;


        FETCH NEXT FROM payroll_cursor
        INTO @EmpID, @BaseSalary;


        WHILE @@FETCH_STATUS = 0

        BEGIN

            SET @Bonus = @BaseSalary * 0.05;


            INSERT INTO Payroll
            (
                Emp_ID,
                BaseSalary,
                Bonus
            )

            VALUES
            (
                @EmpID,
                @BaseSalary,
                @Bonus
            );


            FETCH NEXT FROM payroll_cursor
            INTO @EmpID, @BaseSalary;

        END;


        COMMIT TRANSACTION;


        CLOSE payroll_cursor;

        DEALLOCATE payroll_cursor;


    END TRY


    BEGIN CATCH

        ROLLBACK TRANSACTION;

        PRINT 'Error processing payroll!';


    END CATCH;

END;
```

---

# Database Triggers

## Automatic Payroll Creation

This trigger creates a payroll record whenever a new employee is added.

```sql
CREATE TRIGGER trg_InsertPayroll

ON Employees

AFTER INSERT

AS

BEGIN

    INSERT INTO Payroll
    (
        Emp_ID,
        BaseSalary
    )

    SELECT 
        Emp_ID,
        Salary

    FROM inserted;

END;
```

---

## Employee Status Update

Automatically marks employees as inactive if salary becomes zero.

```sql
CREATE TRIGGER trg_FireEmployee

ON Employees

AFTER UPDATE

AS

BEGIN

    UPDATE Employees

    SET Status = 'Inactive'

    WHERE Emp_ID IN
    (
        SELECT Emp_ID
        FROM inserted
        WHERE Salary = 0
    );

END;
```

---

# Data Analysis Queries

## Employee Payroll Overview

Returns employee details with department and payroll information.

```sql
SELECT

    e.Emp_ID,
    e.FirstName,
    e.LastName,
    e.Email,
    d.DepartmentName,
    e.Status,
    p.PayDate,
    p.BaseSalary,
    p.Bonus,
    p.TotalSalary

FROM Employees e

JOIN Departments d

ON e.DepartmentID = d.DepartmentID

LEFT JOIN Payroll p

ON e.Emp_ID = p.Emp_ID

ORDER BY p.PayDate DESC;
```

---

## Department Salary Expenses

Calculates total payroll expense per department.

```sql
SELECT

    d.DepartmentName,

    SUM(p.TotalSalary) AS TotalExpense

FROM Payroll p

JOIN Employees e

ON p.Emp_ID = e.Emp_ID

JOIN Departments d

ON e.DepartmentID = d.DepartmentID

GROUP BY d.DepartmentName;
```

---

## Highest Bonus Recipients

Finds the top five employees receiving the highest bonuses.

```sql
SELECT TOP 5

    e.FirstName,

    e.LastName,

    p.Bonus,

    p.TotalSalary

FROM Payroll p

JOIN Employees e

ON p.Emp_ID = e.Emp_ID

ORDER BY p.Bonus DESC;
```

---

# Database Optimization

Indexes were created to improve query performance.

## Employee Search Optimization

```sql
CREATE INDEX idx_Employees_LastName

ON Employees (LastName);
```

---

## Payroll Date Optimization

```sql
CREATE INDEX idx_Payroll_PayDate

ON Payroll (PayDate);
```

---

# Table Partitioning

Payroll data can grow significantly over time.

Partitioning improves performance by splitting large datasets into smaller logical sections.

```sql
CREATE PARTITION FUNCTION pf_PayDateRange (DATE)

AS RANGE RIGHT

FOR VALUES
(
    '2024-01-01',
    '2025-01-01'
);



CREATE PARTITION SCHEME ps_PayDateScheme

AS PARTITION pf_PayDateRange

ALL TO ([PRIMARY]);
```

---

# Key Skills Demonstrated

This project demonstrates:

- SQL database design
- Relational modeling
- Primary and foreign keys
- Stored procedures
- Transactions
- Cursors
- Database triggers
- Data analysis queries
- Index optimization
- Table partitioning

---

# Possible Improvements

Future improvements could include:

- User authentication and permissions
- Employee attendance tracking
- Tax calculations
- Leave management
- Payroll reporting dashboard
- Integration with Power BI/Tableau
