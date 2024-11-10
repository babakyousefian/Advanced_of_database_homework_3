# Advanced_of_database_homework_3

---

# Here’s a breakdown of the PostgreSQL tasks described in your assignment:
# Steps to Implement in PostgreSQL

```bash

    Create a Database
        Create a database named LorestanUniv.

    Define Tables

        Create tables student, lecturer, course, CourseRegister, and PresentedCourses with the specified fields and constraints. Here’s a basic outline:

    CREATE TABLE student (
        STID SERIAL PRIMARY KEY,
        FName VARCHAR(10) NOT NULL,
        LName VARCHAR(10) NOT NULL,
        Father VARCHAR(10),
        Birth DATE,
        IDS CHAR(3),
        BornCity VARCHAR(20),
        Address VARCHAR(100),
        PostalCode CHAR(10),
        CPhone VARCHAR(11),
        HPhone VARCHAR(11),
        Department VARCHAR(50),
        Major VARCHAR(50),
        Married BOOLEAN,
        ID CHAR(10)
    );

    CREATE TABLE lecturer (
        LID SERIAL PRIMARY KEY,
        FName VARCHAR(10) NOT NULL,
        LName VARCHAR(10) NOT NULL,
        ID CHAR(10),
        Department VARCHAR(50),
        Major VARCHAR(50),
        Birth DATE,
        BornCity VARCHAR(20),
        Address VARCHAR(100),
        PostalCode CHAR(10),
        CPhone VARCHAR(11),
        HPhone VARCHAR(11)
    );

    CREATE TABLE course (
        CID SERIAL PRIMARY KEY,
        CName VARCHAR(25),
        Department VARCHAR(50),
        Credit INT CHECK (Credit BETWEEN 1 AND 4)
    );

    CREATE TABLE CourseRegister (
        CID INT REFERENCES course(CID),
        SID INT REFERENCES student(STID),
        PRIMARY KEY (CID, SID)
    );

    CREATE TABLE PresentedCourses (
        CID INT REFERENCES course(CID),
        LID INT REFERENCES lecturer(LID),
        PRIMARY KEY (CID, LID)
    );
```

## Insert Data with Validation

    Insert sample records into each table with validation checks to ensure data integrity. This includes constraints on fields like Married, Department, Major, CPhone, and PostalCode.

### Implement Update and Delete Operations

    Test insertions, updates, and deletions for each table to ensure constraints are enforced.

### User Roles and Permissions
```bash

    Create three users with the specified permissions:

        CREATE USER Saman WITH PASSWORD 'password1';
        CREATE USER Armin WITH PASSWORD 'password2';
        CREATE USER Sara WITH PASSWORD 'password3';

        GRANT ALL PRIVILEGES ON DATABASE LorestanUniv TO Saman;
        GRANT SELECT ON ALL TABLES IN SCHEMA public TO Sara;
        GRANT INSERT, UPDATE, DELETE ON student TO Armin;
```

### Transactions with Savepoints
        Implement transactions for course deletions with savepoints, ensuring related records are deleted across multiple tables, but allowing rollbacks at certain stages if needed.


---

```bash

-- 1. Create the Database
CREATE DATABASE LorestanUniv;

-- Connect to the Database
\c LorestanUniv;

-- 2. Define Tables

-- Table: student
CREATE TABLE student (
    STID SERIAL PRIMARY KEY,
    FName VARCHAR(10) NOT NULL CHECK (FName ~ '^[\p{L}]+$'), -- Only Persian letters allowed, up to 10 characters
    LName VARCHAR(10) NOT NULL CHECK (LName ~ '^[\p{L}]+$'), -- Only Persian letters allowed, up to 10 characters
    Father VARCHAR(10) CHECK (Father ~ '^[\p{L}]+$'), -- Only Persian letters allowed
    Birth DATE, -- Persian date format assumed
    IDS CHAR(3), -- Serial of birth certificate, 3-digit number
    BornCity VARCHAR(20), -- Birthplace, restricted to certain cities if needed
    Address VARCHAR(100), -- Address, max 100 characters
    PostalCode CHAR(10) CHECK (PostalCode ~ '^\d{10}$'), -- 10-digit postal code
    CPhone VARCHAR(11) CHECK (CPhone ~ '^09\d{9}$'), -- Iranian mobile format, starting with '09'
    HPhone VARCHAR(11) CHECK (HPhone ~ '^\d{11}$'), -- Landline format, 11 digits
    Department VARCHAR(50), -- Limited to predefined departments
    Major VARCHAR(50), -- Limited to engineering majors
    Married BOOLEAN, -- Marital status: true for married, false for single
    ID CHAR(10) CHECK (ID ~ '^\d{10}$') -- Iranian national ID format
);

-- Table: lecturer
CREATE TABLE lecturer (
    LID SERIAL PRIMARY KEY,
    FName VARCHAR(10) NOT NULL CHECK (FName ~ '^[\p{L}]+$'), -- Only Persian letters allowed
    LName VARCHAR(10) NOT NULL CHECK (LName ~ '^[\p{L}]+$'), -- Only Persian letters allowed
    ID CHAR(10) CHECK (ID ~ '^\d{10}$'), -- Iranian national ID format
    Department VARCHAR(50), -- Limited to predefined departments
    Major VARCHAR(50), -- Limited to engineering majors
    Birth DATE, -- Persian date format assumed
    BornCity VARCHAR(20), -- Birthplace, restricted to certain cities if needed
    Address VARCHAR(100), -- Address, max 100 characters
    PostalCode CHAR(10) CHECK (PostalCode ~ '^\d{10}$'), -- 10-digit postal code
    CPhone VARCHAR(11) CHECK (CPhone ~ '^09\d{9}$'), -- Iranian mobile format, starting with '09'
    HPhone VARCHAR(11) CHECK (HPhone ~ '^\d{11}$') -- Landline format, 11 digits
);

-- Table: course
CREATE TABLE course (
    CID SERIAL PRIMARY KEY,
    CName VARCHAR(25) CHECK (CName ~ '^[\p{L}]+$'), -- Course name in Persian, max 25 characters
    Department VARCHAR(50), -- Limited to predefined departments
    Credit INT CHECK (Credit BETWEEN 1 AND 4) -- Credit hours between 1 and 4
);

-- Table: CourseRegister
CREATE TABLE CourseRegister (
    CID INT REFERENCES course(CID) ON DELETE CASCADE,
    SID INT REFERENCES student(STID) ON DELETE CASCADE,
    PRIMARY KEY (CID, SID)
);

-- Table: PresentedCourses
CREATE TABLE PresentedCourses (
    CID INT REFERENCES course(CID) ON DELETE CASCADE,
    LID INT REFERENCES lecturer(LID) ON DELETE CASCADE,
    PRIMARY KEY (CID, LID)
);
```

### Explanation of the Code

Database and Tables: Creates the LorestanUniv database and defines five tables: student, lecturer, course, CourseRegister, and PresentedCourses.
Constraints: Implements constraints for data validation, such as Persian characters for names, specific formats for phone numbers and postal codes, and credit hours within a defined range.
Foreign Keys with ON DELETE CASCADE: Ensures that deleting a record in course or student cascades deletions in related tables like CourseRegister and PresentedCourses.

---

### Example Code for Transaction with Savepoints

- In this example, we’ll delete a course from the course table, and also delete related entries from CourseRegister and PresentedCourses. We’ll use savepoints to allow partial rollbacks if needed.

```bash

-- Start the transaction
BEGIN;

-- Set the first savepoint before deleting records in `CourseRegister`
SAVEPOINT before_course_register_delete;

-- 1. Delete records in `CourseRegister` for the selected course
DELETE FROM CourseRegister WHERE CID = 1;

-- If needed, rollback to this savepoint to undo only the deletions in `CourseRegister`
-- ROLLBACK TO SAVEPOINT before_course_register_delete;

-- Set the second savepoint before deleting records in `PresentedCourses`
SAVEPOINT before_presented_courses_delete;

-- 2. Delete records in `PresentedCourses` for the selected course
DELETE FROM PresentedCourses WHERE CID = 1;

-- If needed, rollback to this savepoint to undo only the deletions in `PresentedCourses`
-- ROLLBACK TO SAVEPOINT before_presented_courses_delete;

-- Set the third savepoint before deleting the course itself
SAVEPOINT before_course_delete;

-- 3. Delete the course from the `course` table
DELETE FROM course WHERE CID = 1;

-- Commit the transaction if all deletions are successful
COMMIT;
```

### Explanation of the Code

- BEGIN: Starts the transaction.

- Savepoints: We set three savepoints:

- before_course_register_delete: Allows rolling back deletions in CourseRegister.

- before_presented_courses_delete: Allows rolling back deletions in PresentedCourses after CourseRegister deletions have succeeded.

- before_course_delete: Allows rolling back the final deletion of the course itself.

- ROLLBACK TO SAVEPOINT: You can uncomment any ROLLBACK TO SAVEPOINT line to undo actions up to that savepoint, retaining the changes made before that point.

- COMMIT: Commits all deletions if everything goes as expected.

### Usage Scenario

If an error occurs after deleting from CourseRegister, you can rollback to before_course_register_delete to undo only those deletions and keep other tables unaffected.
Similarly, each savepoint allows partial rollbacks, making it easier to control data consistency without aborting the entire transaction unless necessary.

---

### This code will demonstrate different scenarios where you may want to:

1. Abort the entire transaction,
2. Rollback to a specific savepoint,
3. Use savepoints strategically to maintain data consistency.

### Full Example Code with Abort, Rollback, and Savepoints

```bash

-- Start the transaction
BEGIN;

-- Set the first savepoint before deleting records in CourseRegister
SAVEPOINT before_course_register_delete;

-- 1. Delete records in CourseRegister for the selected course
DELETE FROM CourseRegister WHERE CID = 1;

-- Check deletion or simulate an error condition to rollback to this savepoint
-- Uncomment the next line to simulate a rollback to this savepoint
-- ROLLBACK TO SAVEPOINT before_course_register_delete;

-- Set the second savepoint before deleting records in PresentedCourses
SAVEPOINT before_presented_courses_delete;

-- 2. Delete records in PresentedCourses for the selected course
DELETE FROM PresentedCourses WHERE CID = 1;

-- Rollback to this savepoint if any issues occur with deleting in PresentedCourses
-- Uncomment the next line to simulate a rollback to this savepoint
-- ROLLBACK TO SAVEPOINT before_presented_courses_delete;

-- Set the third savepoint before deleting the course itself
SAVEPOINT before_course_delete;

-- 3. Delete the course from the course table
DELETE FROM course WHERE CID = 1;

-- Simulate an issue here that requires aborting the entire transaction
-- Uncomment the next line to abort the entire transaction
-- ROLLBACK;

-- Commit the transaction if all deletions are successful
COMMIT;
```

### Explanation of Key Commands

BEGIN: Begins the transaction, allowing you to group multiple operations into one atomic operation.

#### SAVEPOINT:
    We create three savepoints: before_course_register_delete, before_presented_courses_delete, and before_course_delete.
    These allow you to selectively undo parts of the transaction without aborting the entire operation.

#### ROLLBACK TO SAVEPOINT:
    Uncomment any ROLLBACK TO SAVEPOINT statement to undo actions only up to that savepoint, preserving changes made before it.
    For example, ROLLBACK TO SAVEPOINT before_course_register_delete will undo deletions in CourseRegister only, allowing you to retry or correct issues.

#### ROLLBACK:
    If a critical error occurs at any point (such as an integrity constraint violation or other issues that would prevent successful completion), use ROLLBACK to abort the entire transaction.
    This undoes all operations within the transaction, returning the database to its state before BEGIN.

#### COMMIT:
    If all operations are successful, use COMMIT to finalize all changes, making them permanent in the database.

- This structure gives you control over how to handle different stages of the transaction and flexibility to respond to issues at any point in the process.

---

1. student Table

### DELETE Operations

1.1 Assume we want to delete a student based on STID, LName, and Department.

## -- Delete a student by ID
```bash

DELETE FROM student WHERE STID = 1;

-- Delete a student by last name
DELETE FROM student WHERE LName = 'Doe';

-- Delete students from a specific department
DELETE FROM student WHERE Department = 'Computer Science';

UPDATE Operations

```

### Assume we want to update a student's FName, Department, and CPhone.

```bash

-- Update a student's first name by ID
UPDATE student
SET FName = 'Ali'
WHERE STID = 1;

-- Update a student's department by last name
UPDATE student
SET Department = 'Electrical Engineering'
WHERE LName = 'Doe';

-- Update a student's contact phone number

UPDATE student
SET CPhone = '09123456789'
WHERE STID = 2;

```

2. lecturer Table
### DELETE Operations

### Assume we want to delete a lecturer based on LID, Major, and BornCity.

```bash

-- Delete a lecturer by ID
DELETE FROM lecturer WHERE LID = 1;

-- Delete lecturers by major
DELETE FROM lecturer WHERE Major = 'Mathematics';

-- Delete lecturers by birthplace
DELETE FROM lecturer WHERE BornCity = 'Tehran';

UPDATE Operations

Assume we want to update a lecturer's Major, Department, and HPhone.

-- Update a lecturer's major by ID
UPDATE lecturer
SET Major = 'Physics'
WHERE LID = 1;

-- Update a lecturer's department by name
UPDATE lecturer
SET Department = 'Mechanical Engineering'
WHERE LName = 'Smith';

-- Update a lecturer's home phone
UPDATE lecturer
SET HPhone = '02112345678'
WHERE LID = 2;

```


3. course Table
### DELETE Operations

### Assume we want to delete a course based on CID, CName, and Department.

```bash

-- Delete a course by ID
DELETE FROM course WHERE CID = 1;

-- Delete a course by name
DELETE FROM course WHERE CName = 'Database Systems';

-- Delete courses by department
DELETE FROM course WHERE Department = 'Civil Engineering';

UPDATE Operations

Assume we want to update a course's CName, Credit, and Department.

-- Update a course name by ID
UPDATE course
SET CName = 'Advanced Databases'
WHERE CID = 1;

-- Update a course's credit hours by name
UPDATE course
SET Credit = 3
WHERE CName = 'Data Structures';

-- Update a course's department by ID
UPDATE course
SET Department = 'Software Engineering'
WHERE CID = 2;

```

## Summary

- These commands provide targeted examples for DELETE and UPDATE operations on specific attributes in each table.

---

1. student Table

### Assume the validations include constraints like Persian characters for names, a specific format for phone numbers, a 10-digit postal code, and a Boolean for marital status.

```bash

INSERT INTO student (
    FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, 
    CPhone, HPhone, Department, Major, Married, ID
) VALUES (
    'علی',               -- FName (Persian, max 10 characters)
    'رضایی',            -- LName (Persian, max 10 characters)
    'حسین',             -- Father (Persian, max 10 characters)
    '2001-04-15',       -- Birth (assuming Gregorian date for simplicity)
    '123',              -- IDS (3-digit serial)
    'Tehran',           -- BornCity
    '123 Main St, Tehran', -- Address (max 100 characters)
    '1234567890',       -- PostalCode (10 digits)
    '09121234567',      -- CPhone (Iranian mobile format)
    '02112345678',      -- HPhone (Iranian landline format)
    'Engineering',      -- Department
    'Computer Science', -- Major
    TRUE,               -- Married (Boolean)
    '0123456789'        -- ID (10-digit national ID)
);

```

2. lecturer Table

### Assume validations like Persian characters for names, standard national ID, a 10-digit postal code, and 11-digit phone numbers.

```bash

INSERT INTO lecturer (
    FName, LName, ID, Department, Major, Birth, BornCity, 
    Address, PostalCode, CPhone, HPhone
) VALUES (
    'محمد',               -- FName (Persian, max 10 characters)
    'احمدی',             -- LName (Persian, max 10 characters)
    '1234567890',        -- ID (10-digit national ID)
    'Engineering',       -- Department
    'Electrical Engineering', -- Major
    '1980-07-20',       -- Birth (assuming Gregorian date)
    'Mashhad',           -- BornCity
    '456 Second St, Mashhad', -- Address (max 100 characters)
    '0987654321',        -- PostalCode (10 digits)
    '09127654321',       -- CPhone (Iranian mobile format)
    '02165432198'        -- HPhone (Iranian landline format)
);

```

3. course Table

### Assume the validations require a Persian name for the course, predefined departments, and a credit value between 1 and 4.


```bash

INSERT INTO course (
    CName, Department, Credit
) VALUES (
    'سیستم‌های پایگاه داده', -- CName (Persian, max 25 characters)
    'Computer Science',       -- Department
    3                         -- Credit (between 1 and 4)
);


```

4. CourseRegister Table

### Assume CourseRegister links student and course tables, so we need valid CID and SID that exist in their respective tables.

```bash

INSERT INTO CourseRegister (
    CID, SID
) VALUES (
    1, -- CID (Assuming course with ID 1 exists)
    1  -- SID (Assuming student with ID 1 exists)
);


```

5. PresentedCourses Table

### Assume PresentedCourses links course and lecturer tables, so we need valid CID and LID that exist in their respective tables.

```bash

INSERT INTO PresentedCourses (
    CID, LID
) VALUES (
    1, -- CID (Assuming course with ID 1 exists)
    1  -- LID (Assuming lecturer with ID 1 exists)
);

```

## Summary

- These INSERT statements should comply with all specified constraints. Ensure the respective primary keys (CID, SID, and LID) already exist in the tables to satisfy foreign key requirements for CourseRegister and PresentedCourses.

---


#### To show additional fields like CName, Department, Credit (from the course table), and FName, LName (from the student table) in CourseRegister, we can create a view in PostgreSQL. This view will allow us to join CourseRegister with course and student tables and display the augmented information without duplicating data.

## Here’s how you can create this view:

1. Creating the CourseRegister View with Augmented Fields


```bash

CREATE VIEW CourseRegisterView AS
SELECT 
    cr.CID,
    cr.SID,
    c.CName AS CourseName,
    c.Department AS CourseDepartment,
    c.Credit AS CourseCredit,
    s.FName AS StudentFirstName,
    s.LName AS StudentLastName
FROM 
    CourseRegister cr
JOIN 
    course c ON cr.CID = c.CID
JOIN 
    student s ON cr.SID = s.STID;

```

## Explanation of the Code

1. View Definition: The CREATE VIEW CourseRegisterView AS statement defines a view named CourseRegisterView.
2. JOIN Clauses: We use JOIN statements to link CourseRegister with the course and student tables.
3. CourseRegister is joined with course on CID, allowing us to fetch CName, Department, and Credit from the course table.
4. CourseRegister is joined with student on SID, allowing us to fetch FName and LName from the student table.    Selected Fields: The SELECT statement pulls relevant columns from each table to display in the view.

## Sample Query to Use the View

- After creating the view, you can run queries to see the augmented data in CourseRegisterView:

```bash

-- Select all records from the CourseRegisterView
SELECT * FROM CourseRegisterView;

```

### This will display the CID, SID, CourseName, CourseDepartment, CourseCredit, StudentFirstName, and StudentLastName fields for each record in CourseRegister, showing you the related information from both the course and student tables.
### Benefits of Using a View

1. No Data Duplication: The view dynamically retrieves data, so there’s no need to store redundant information in CourseRegister.
2. Automatic Updates: Any changes in course or student tables will automatically reflect in CourseRegisterView.
3. Simplified Queries: You can now query CourseRegisterView to get the augmented data without writing complex joins every time.

- This approach should meet your requirement for displaying related fields from multiple tables in CourseRegister.

---

### For the PresentedCourses table, we can create a view that joins it with the course and lecturer tables to display the additional fields: CName, Department, and Credit (from course), as well as FName and LName (from lecturer).
### Creating the PresentedCourses View with Augmented Fields

```bash

CREATE VIEW PresentedCoursesView AS
SELECT 
    pc.CID,
    pc.LID,
    c.CName AS CourseName,
    c.Department AS CourseDepartment,
    c.Credit AS CourseCredit,
    l.FName AS LecturerFirstName,
    l.LName AS LecturerLastName
FROM 
    PresentedCourses pc
JOIN 
    course c ON pc.CID = c.CID
JOIN 
    lecturer l ON pc.LID = l.LID;

```

## Explanation of the Code

1. View Definition: CREATE VIEW PresentedCoursesView AS defines a view named PresentedCoursesView.
2. JOIN Clauses:
3. PresentedCourses is joined with course on CID to retrieve CName, Department, and Credit fields from the course table.
4. PresentedCourses is also joined with lecturer on LID to retrieve FName and LName fields from the lecturer table.
5. Selected Fields: We select CID and LID from PresentedCourses, along with CourseName, CourseDepartment, CourseCredit, LecturerFirstName, and LecturerLastName.

### Sample Query to Use the View

### After creating this view, you can run queries on PresentedCoursesView to see the augmented data:


```bash

-- Select all records from the PresentedCoursesView
SELECT * FROM PresentedCoursesView;
```

## This query will display each course along with its lecturer’s name and the course details like department and credit hours, which are fetched from related tables.
#### Benefits of Using This View

6. Data Integrity: The view does not store additional information, so data remains consistent with the course and lecturer tables.
7. Automatic Updates: Changes in the course or lecturer tables will automatically be reflected in PresentedCoursesView.
8. Simplified Queries: The view allows easy access to combined data without needing complex joins in each query.

- This approach meets the requirement of displaying additional related information for PresentedCourses without duplicating data.

---

## To set up user permissions as you described in PostgreSQL, we need to grant specific privileges to each user:

1. Saman: Full access to the database (admin).
2. Armin: Read-only access to all tables.
3. Sara: Write access on the student table and read access on all other tables.

#### Here’s the code to implement these roles and permissions:
1. Create Users

- - First, create the users if they don't already exist.


```bash

CREATE USER Saman WITH PASSWORD 'password1';
CREATE USER Armin WITH PASSWORD 'password2';
CREATE USER Sara WITH PASSWORD 'password3';
```

2. Grant Permissions
- - Grant Full Access to Saman (Admin)

- - Make Saman the owner of the database and grant all privileges

```bash 

ALTER DATABASE LoresranU OWNER TO Saman;

-- Grant Saman all privileges on all tables in the public schema
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO Saman;

-- Grant Saman privileges to create, alter, and delete tables
GRANT ALL PRIVILEGES ON SCHEMA public TO Saman;

Grant Read-Only Access to Armin

-- Grant Armin read-only access on all tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO Armin;

-- Ensure that any future tables created in the schema automatically grant SELECT to Armin
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO Armin;

Grant Write Access on student and Read Access on Other Tables to Sara

-- Grant Sara read-only access on all tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO Sara;

-- Grant Sara insert, update, and delete privileges on the `student` table
GRANT INSERT, UPDATE, DELETE ON student TO Sara;

-- Ensure any future tables grant read-only access to Sara automatically
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO Sara;

```

### Summary of Permissions

1. Saman: Full administrative access on the database.
2. Armin: Read-only access on all tables.
3. Sara: Write access on the student table and read-only access on all other tables.

- This setup should provide the desired access controls for each user.

---

## To demonstrate INSERT, DELETE, UPDATE, and READ queries for each user (Saman, Armin, and Sara), here is the SQL code based on the permissions given:

1. Saman: Full access (admin privileges) on all tables.
2. Armin: Read-only access on all tables.
3. Sara: Write access on the student table and read-only access on all other tables.

### Let's go through the sample queries for each user on all tables.
### Code for Saman (Admin Access)

--- 
---
# INSERT into student
---
---

```bash

-- Run as Saman
INSERT INTO student (
    FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, 
    CPhone, HPhone, Department, Major, Married, ID
) VALUES (
    'reza', 'mohammadi', 'hossein', '2000-01-01', '456', 'Shiraz', 
    '456 Main St', '1234567890', '09123456789', '06633202566', 
    'Engineering', 'Electrical Engineering', FALSE, '1234567890'
);

DELETE from student

-- Run as Saman
DELETE FROM student WHERE FName = 'reza' AND LName = 'mohammadi';

```
---
---
# UPDATE on student
---
---

```bash

-- Run as Saman
UPDATE student
SET Department = 'Computer Science'
WHERE FName = 'ali' AND LName = 'alavi';

```
---
---
# READ from student
---
---

```bash

-- Run as Saman
SELECT * FROM student;

Code for Armin (Read-Only Access)

Since Armin has read-only access, they can only execute SELECT queries.
READ from student

-- Run as Armin
SELECT * FROM student;

READ from lecturer

-- Run as Armin
SELECT * FROM lecturer;

READ from course

-- Run as Armin
SELECT * FROM course;

READ from CourseRegister

-- Run as Armin
SELECT * FROM CourseRegister;

READ from PresentedCourses

-- Run as Armin
SELECT * FROM PresentedCourses;
```

- - Note: Since Armin has read-only access, any attempt to run INSERT, DELETE, or UPDATE will result in an error.

### Code for Sara (Write on student Table, Read-Only on Others)

---
---
# INSERT into student
---
---

```bash

-- Run as Sara
INSERT INTO student (
    FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, 
    CPhone, HPhone, Department, Major, Married, ID
) VALUES (
    'zahra', 'bahrami', 'ali', '1999-02-15', '789', 'Tehran', 
    '789 Main St', '9876543210', '09127654321', '02133202566', 
    'Science', 'Physics', FALSE, '9876543210'
);

DELETE from student

-- Run as Sara
DELETE FROM student WHERE FName = 'zahra' AND LName = 'bahrami';

UPDATE on student

-- Run as Sara
UPDATE student
SET Department = 'Mathematics'
WHERE FName = 'ali' AND LName = 'alavi';

READ from student

-- Run as Sara
SELECT * FROM student;

READ from lecturer

-- Run as Sara
SELECT * FROM lecturer;

READ from course

-- Run as Sara
SELECT * FROM course;

READ from CourseRegister

-- Run as Sara
SELECT * FROM CourseRegister;

READ from PresentedCourses

-- Run as Sara
SELECT * FROM PresentedCourses;

```

- - Note: Any attempt by Sara to INSERT, DELETE, or UPDATE on tables other than student will result in an error.

### Summary of Permissions

1. Saman: Can perform all INSERT, DELETE, UPDATE, and READ operations on all tables.
2. Armin: Restricted to READ operations only on all tables.
3. Sara: Can INSERT, DELETE, and UPDATE only on the student table, but can READ from all tables.

---
---
---

# Comprehensive Explanation of Code
## Database Setup

1. Creating Users and Permissions:
    - Saman: Full administrative access, meaning Saman can perform all operations on any table.
    - Armin: Read-only access across all tables, meaning Armin can only execute SELECT queries.
    - Sara: Write access on the student table and read-only access on all other tables, allowing Sara to insert, update, and delete records in student, but only read from other tables.

2. Tables:
    - student: Contains student information such as FName, LName, Father, Birth, Department, etc.
    - lecturer: Contains lecturer details like FName, LName, Department, and contact information.
    - course: Stores course details, including CName, Department, and Credit.
    - CourseRegister: Links students and courses.
    - PresentedCourses: Links courses and lecturers.

3. Operations for Each User
4. Operations for Saman (Admin Access)

```bash

    INSERT:
        Saman inserts a new record in the student table with details such as FName = 'reza', LName = 'mohammadi', and other information.
    DELETE:
        Saman deletes a student with FName = 'reza' and LName = 'mohammadi', demonstrating admin capability to remove records.
    UPDATE:
        Saman updates the Department for a student with FName = 'ali' and LName = 'alavi to "Computer Science."
    READ:
        Saman selects all records from the student table to view the data, demonstrating read access.
```

5. Operations for Armin (Read-Only Access)

### Since Armin has only read access, only SELECT queries can be performed:

```bash

    READ:
        Armin selects all records from student, lecturer, course, CourseRegister, and PresentedCourses tables.
        Any INSERT, DELETE, or UPDATE attempt will result in an error due to insufficient permissions.

Operations for Sara (Write Access on student, Read-Only on Others)

    INSERT:
        Sara inserts a new student record into the student table with details like FName = 'zahra', LName = 'bahrami', Department = 'Science', and Major = 'Physics'.
    DELETE:
        Sara deletes the record for the student with FName = 'zahra' and LName = 'bahrami.
    UPDATE:
        Sara updates the Department field for the student with FName = 'ali' and LName = 'alavi to "Mathematics."
    READ:
        Sara selects all records from student and has read-only access to all other tables (lecturer, course, CourseRegister, and PresentedCourses).

```


# Database User Permissions and Operations

## Overview

This project sets up a PostgreSQL database with tables for managing information related to students, lecturers, courses, course registration, and presented courses. Different users (`Saman`, `Armin`, and `Sara`) are granted specific permissions on the database, demonstrating different levels of access control.

### Users and Permissions

1. **Saman** - Admin user with full access:
   - Can perform `INSERT`, `DELETE`, `UPDATE`, and `READ` on all tables.
2. **Armin** - Read-only user:
   - Can only perform `READ` operations across all tables.
3. **Sara** - Write on `student` table, read-only on others:
   - Can perform `INSERT`, `DELETE`, and `UPDATE` on the `student` table.
   - Has `READ` access on all other tables.

---

## Tables

1. **student**: Stores information about students, including fields like `FName`, `LName`, `Birth`, `Department`, etc.
2. **lecturer**: Stores details about lecturers, including `FName`, `LName`, `Department`, etc.
3. **course**: Contains course information such as `CName`, `Department`, and `Credit`.
4. **CourseRegister**: Links students and courses.
5. **PresentedCourses**: Links courses and lecturers.

---

## SQL Commands for Each User

### Saman (Full Access - Admin)

#### Insert

```sql
INSERT INTO student (FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, CPhone, HPhone, Department, Major, Married, ID)
VALUES ('reza', 'mohammadi', 'hossein', '2000-01-01', '456', 'Shiraz', '456 Main St', '1234567890', '09123456789', '06633202566', 'Engineering', 'Electrical Engineering', FALSE, '1234567890');

Delete

DELETE FROM student WHERE FName = 'reza' AND LName = 'mohammadi';

Update

UPDATE student SET Department = 'Computer Science' WHERE FName = 'ali' AND LName = 'alavi';

Read

SELECT * FROM student;

Armin (Read-Only Access)
Read (Example for All Tables)

SELECT * FROM student;
SELECT * FROM lecturer;
SELECT * FROM course;
SELECT * FROM CourseRegister;
SELECT * FROM PresentedCourses;

Sara (Write on student Table, Read-Only on Others)
Insert

INSERT INTO student (FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, CPhone, HPhone, Department, Major, Married, ID)
VALUES ('zahra', 'bahrami', 'ali', '1999-02-15', '789', 'Tehran', '789 Main St', '9876543210', '09127654321', '02133202566', 'Science', 'Physics', FALSE, '9876543210');

Delete

DELETE FROM student WHERE FName = 'zahra' AND LName = 'bahrami';

Update

UPDATE student SET Department = 'Mathematics' WHERE FName = 'ali' AND LName = 'alavi';

Read (Example for All Tables)

SELECT * FROM student;
SELECT * FROM lecturer;
SELECT * FROM course;
SELECT * FROM CourseRegister;
SELECT * FROM PresentedCourses;

Summary of Permissions
User	Tables	Permissions
Saman	All tables	INSERT, DELETE, UPDATE, READ (Admin)
Armin	All tables	READ only
Sara	student	INSERT, DELETE, UPDATE
	All tables	READ only

This setup demonstrates how to manage and restrict access in PostgreSQL based on user roles and permissions.
License

This project is licensed under the MIT License.

```

- - - This `README.md` file gives a structured overview of the users, permissions, table definitions, and SQL operations available to each user. It’s ready for documentation purposes in any database project with similar role-based access control needs.


---
---
---

# University Database Project with PostgreSQL

- - This project sets up a PostgreSQL database to manage university information, including students, lecturers, courses, and course enrollments. The database is designed to handle role-based access for multiple users and demonstrates transaction control using rollback, savepoints, abort, and commit.

---

## Overview

The database contains several tables representing core university entities:
- `student`: Stores student information.
- `lecturer`: Stores lecturer information.
- `course`: Contains details on each course offered.
- `CourseRegister`: Links students to courses they are enrolled in.
- `PresentedCourses`: Links lecturers to courses they teach.

Three user roles (`Saman`, `Armin`, and `Sara`) are set up with distinct permissions to demonstrate role-based access control in PostgreSQL.

---

## Users and Permissions

1. **Saman**: Admin user with full access.
   - Can perform `INSERT`, `DELETE`, `UPDATE`, and `READ` operations on all tables.
   - Has permission to manage transactions (e.g., `COMMIT`, `ROLLBACK`, `SAVEPOINT`).

2. **Armin**: Read-only user.
   - Can only perform `READ` operations on all tables.

3. **Sara**: Write access on the `student` table, read-only on others.
   - Can perform `INSERT`, `DELETE`, and `UPDATE` operations on the `student` table.
   - Has `READ` access on all other tables.

---

## Table Definitions

### `student` Table
- Contains fields such as `FName`, `LName`, `Father`, `Birth`, `Department`, etc.
- Holds information on each student and enforces data constraints (e.g., phone number format, ID length).

### `lecturer` Table
- Holds lecturer details, including fields like `FName`, `LName`, `Department`, and `Major`.

### `course` Table
- Contains course information such as `CName`, `Department`, and `Credit`.

### `CourseRegister` Table
- Links students to the courses they are enrolled in.

### `PresentedCourses` Table
- Links lecturers to the courses they teach.

---

## SQL Operations by User

### Saman (Admin Access)

1. **INSERT**:
   ```sql
   INSERT INTO student (FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, CPhone, HPhone, Department, Major, Married, ID)
   VALUES ('reza', 'mohammadi', 'hossein', '2000-01-01', '456', 'Shiraz', '456 Main St', '1234567890', '09123456789', '06633202566', 'Engineering', 'Electrical Engineering', FALSE, '1234567890');

    DELETE:

```bash

DELETE FROM student WHERE FName = 'reza' AND LName = 'mohammadi';

UPDATE:

UPDATE student SET Department = 'Computer Science' WHERE FName = 'ali' AND LName = 'alavi';

READ:

SELECT * FROM student;

Transactions (Savepoint, Rollback, Commit):

    BEGIN;

    -- Set a savepoint
    SAVEPOINT before_deleting_course;

    -- Delete from CourseRegister
    DELETE FROM CourseRegister WHERE CID = 1;

    -- Rollback to savepoint if needed
    ROLLBACK TO SAVEPOINT before_deleting_course;

    -- Set another savepoint
    SAVEPOINT before_final_deletion;

    -- Delete the course
    DELETE FROM course WHERE CID = 1;

    -- Commit the transaction
    COMMIT;

Armin (Read-Only Access)

    READ operations (examples for each table):

    SELECT * FROM student;
    SELECT * FROM lecturer;
    SELECT * FROM course;
    SELECT * FROM CourseRegister;
    SELECT * FROM PresentedCourses;

Sara (Write Access on student Table, Read-Only on Others)

    INSERT (on student):

INSERT INTO student (FName, LName, Father, Birth, IDS, BornCity, Address, PostalCode, CPhone, HPhone, Department, Major, Married, ID)
VALUES ('zahra', 'bahrami', 'ali', '1999-02-15', '789', 'Tehran', '789 Main St', '9876543210', '09127654321', '02133202566', 'Science', 'Physics', FALSE, '9876543210');

DELETE (on student):

DELETE FROM student WHERE FName = 'zahra' AND LName = 'bahrami';

UPDATE (on student):

UPDATE student SET Department = 'Mathematics' WHERE FName = 'ali' AND LName = 'alavi';

READ (examples for student and other tables):

    SELECT * FROM student;
    SELECT * FROM lecturer;
    SELECT * FROM course;
    SELECT * FROM CourseRegister;
    SELECT * FROM PresentedCourses;

Transaction Management

    BEGIN: Starts a transaction.
    SAVEPOINT: Sets a savepoint within a transaction, allowing partial rollbacks.
    ROLLBACK TO SAVEPOINT: Rolls back to a specific savepoint without canceling the entire transaction.
    ROLLBACK: Cancels all changes within the transaction, returning the database to its initial state before BEGIN.
    COMMIT: Finalizes the transaction, saving all changes to the database.

Transaction Example

In this example, we demonstrate a multi-step deletion in the course table with savepoints and conditional rollbacks.

```

```bash

BEGIN;

-- Set a savepoint before deleting from CourseRegister
SAVEPOINT before_course_register_delete;

-- Delete entries in CourseRegister related to the course
DELETE FROM CourseRegister WHERE CID = 1;

-- Rollback to this savepoint if needed
-- ROLLBACK TO SAVEPOINT before_course_register_delete;

-- Set another savepoint before deleting from PresentedCourses
SAVEPOINT before_presented_courses_delete;

-- Delete entries in PresentedCourses related to the course
DELETE FROM PresentedCourses WHERE CID = 1;

-- Rollback to this savepoint if needed
-- ROLLBACK TO SAVEPOINT before_presented_courses_delete;

-- Set a final savepoint before deleting the course itself
SAVEPOINT before_course_delete;

-- Delete the course from the course table
DELETE FROM course WHERE CID = 1;

-- Commit the transaction if all steps are successful
COMMIT;

```


- Explanation of Transaction Commands

    - SAVEPOINT: Creates a checkpoint within a transaction. If a step fails or conditions require a partial undo, we can use ROLLBACK TO SAVEPOINT instead of restarting the entire transaction.
    - ROLLBACK TO SAVEPOINT: Rolls back changes to a specific savepoint, allowing for flexible error handling within a transaction.
    - COMMIT: Confirms and applies all changes made within the transaction, making them permanent.

## Summary of Permissions and Operations
- - - User	Tables	Permissions
- - - Saman	All tables	Full access (INSERT, DELETE, UPDATE, READ, Transaction control)
- - - Armin	All tables	Read-only (SELECT only)
- - - Sara	student	INSERT, DELETE, UPDATE on student only
	- - - All tables	Read-only (SELECT only on other tables)

#### This setup demonstrates how to manage and restrict access in PostgreSQL based on user roles and permissions, while also implementing transaction management with rollback, savepoints, and commit.

# License

## This project is licensed under the MIT License.


##### This `README.md` provides a full explanation of the project structure, user permissions, operations, transaction management commands, and example queries for each user. It also includes descriptions of transaction control, such as savepoints and rollbacks, to give clear instructions on using the database effectively.


---
---
---
# @Author : Babak Yousefian
---
---
---

