Домашнее задание к занятию «Базы данных» - `Эсадов Роман`
---
### Задание 1
Employees:

    EmployeeID INT PRIMARY KEY,
    FullName VARCHAR(100)

Salaries:

    SalaryID INT PRIMARY KEY,
    EmployeeID INT,
    Salary DECIMAL(10, 2),
    Внешний ключ EmployeeID

Positions:

    PositionID INT PRIMARY KEY,
    EmployeeID INT,
    PositionTitle VARCHAR(100),
    Внешний ключ EmployeeID

DepartmentTypes:

    DepartmentTypeID INT PRIMARY KEY,
    DepartmentType VARCHAR(100)


StructuralDepartments:

    StructuralDepartmentID INT PRIMARY KEY,
    DepartmentTypeID INT,
    StructuralDepartmentName VARCHAR(100),
    Внешний ключ DepartmentTypeID

HireDates:

    HireDateID INT PRIMARY KEY,
    EmployeeID INT,
    HireDate DATE,
    Внешний ключ EmployeeID

BranchAddresses:

    BranchAddressID INT PRIMARY KEY,
    Address VARCHAR(200)

EmployeeProjects:

    EmployeeProjectID INT PRIMARY KEY,
    EmployeeID INT,
    ProjectName VARCHAR(100),
    Внешний ключ EmployeeID
