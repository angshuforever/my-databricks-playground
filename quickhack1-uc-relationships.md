# Databricks Unity Catalog: Database Relationships and SQL Joins Tutorial

## Introduction

This tutorial combines concepts from database relationships and SQL joins, adapted for use with Databricks Unity Catalog. We'll cover:

1. Setting up Unity Catalog
2. Creating and managing tables
3. Implementing different types of relationships
4. Performing various types of SQL joins

## 1. Setting up Unity Catalog

Before we begin, ensure that Unity Catalog is enabled in your Databricks workspace. You'll need to have the necessary permissions to create and manage metastores, catalogs, and schemas.

### Create a Metastore (if not already created)

```sql
CREATE METASTORE IF NOT EXISTS my_metastore;
USE METASTORE my_metastore;
```

### Create a Catalog

```sql
CREATE CATALOG IF NOT EXISTS tutorial_catalog;
USE CATALOG tutorial_catalog;
```

### Create a Schema

```sql
CREATE SCHEMA IF NOT EXISTS relationships_and_joins;
USE SCHEMA relationships_and_joins;
```

## 2. Creating and Managing Tables

Let's create tables that we'll use throughout this tutorial to demonstrate relationships and joins.

```sql
-- Create Users table (for One-to-One relationship example)
CREATE TABLE users (
    user_id BIGINT GENERATED ALWAYS AS IDENTITY,
    username STRING NOT NULL,
    email STRING NOT NULL,
    CONSTRAINT pk_users PRIMARY KEY (user_id)
) USING DELTA;

-- Create Passports table (for One-to-One relationship example)
CREATE TABLE passports (
    passport_id BIGINT GENERATED ALWAYS AS IDENTITY,
    user_id BIGINT,
    passport_number STRING NOT NULL,
    expiry_date DATE NOT NULL,
    CONSTRAINT pk_passports PRIMARY KEY (passport_id),
    CONSTRAINT fk_passports_users FOREIGN KEY (user_id) REFERENCES users(user_id)
) USING DELTA;

-- Create Authors table (for One-to-Many relationship example)
CREATE TABLE authors (
    author_id BIGINT GENERATED ALWAYS AS IDENTITY,
    name STRING NOT NULL,
    birth_date DATE,
    CONSTRAINT pk_authors PRIMARY KEY (author_id)
) USING DELTA;

-- Create Books table (for One-to-Many relationship example)
CREATE TABLE books (
    book_id BIGINT GENERATED ALWAYS AS IDENTITY,
    title STRING NOT NULL,
    publication_year INT,
    author_id BIGINT,
    CONSTRAINT pk_books PRIMARY KEY (book_id),
    CONSTRAINT fk_books_authors FOREIGN KEY (author_id) REFERENCES authors(author_id)
) USING DELTA;

-- Create Students table (for Many-to-Many relationship example)
CREATE TABLE students (
    student_id BIGINT GENERATED ALWAYS AS IDENTITY,
    name STRING NOT NULL,
    email STRING NOT NULL,
    CONSTRAINT pk_students PRIMARY KEY (student_id)
) USING DELTA;

-- Create Courses table (for Many-to-Many relationship example)
CREATE TABLE courses (
    course_id BIGINT GENERATED ALWAYS AS IDENTITY,
    title STRING NOT NULL,
    credits INT NOT NULL,
    CONSTRAINT pk_courses PRIMARY KEY (course_id)
) USING DELTA;

-- Create Enrollments table (for Many-to-Many relationship example)
CREATE TABLE enrollments (
    enrollment_id BIGINT GENERATED ALWAYS AS IDENTITY,
    student_id BIGINT,
    course_id BIGINT,
    enrollment_date DATE NOT NULL,
    CONSTRAINT pk_enrollments PRIMARY KEY (enrollment_id),
    CONSTRAINT fk_enrollments_students FOREIGN KEY (student_id) REFERENCES students(student_id),
    CONSTRAINT fk_enrollments_courses FOREIGN KEY (course_id) REFERENCES courses(course_id)
) USING DELTA;

-- Create Departments table (for SQL Joins examples)
CREATE TABLE departments (
    dept_id BIGINT GENERATED ALWAYS AS IDENTITY,
    dept_name STRING NOT NULL,
    CONSTRAINT pk_departments PRIMARY KEY (dept_id)
) USING DELTA;

-- Create Employees table (for SQL Joins examples)
CREATE TABLE employees (
    emp_id BIGINT GENERATED ALWAYS AS IDENTITY,
    first_name STRING NOT NULL,
    last_name STRING NOT NULL,
    dept_id BIGINT,
    manager_id BIGINT,
    CONSTRAINT pk_employees PRIMARY KEY (emp_id),
    CONSTRAINT fk_employees_departments FOREIGN KEY (dept_id) REFERENCES departments(dept_id),
    CONSTRAINT fk_employees_managers FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
) USING DELTA;
```

## 3. Implementing Different Types of Relationships

### One-to-One Relationship (Users and Passports)

Insert sample data:

```sql
INSERT INTO users (username, email) VALUES
('john_doe', 'john@example.com'),
('jane_smith', 'jane@example.com');

INSERT INTO passports (user_id, passport_number, expiry_date) VALUES
(1, 'AB123456', '2025-12-31'),
(2, 'CD789012', '2026-06-30');
```

Query to retrieve user and passport information:

```sql
SELECT u.username, u.email, p.passport_number, p.expiry_date
FROM users u
JOIN passports p ON u.user_id = p.user_id;
```

### One-to-Many Relationship (Authors and Books)

Insert sample data:

```sql
INSERT INTO authors (name, birth_date) VALUES
('J.K. Rowling', '1965-07-31'),
('George Orwell', '1903-06-25');

INSERT INTO books (title, publication_year, author_id) VALUES
('Harry Potter and the Philosopher''s Stone', 1997, 1),
('Harry Potter and the Chamber of Secrets', 1998, 1),
('1984', 1949, 2),
('Animal Farm', 1945, 2);
```

Query to retrieve authors and their books:

```sql
SELECT a.name, b.title, b.publication_year
FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
ORDER BY a.name, b.publication_year;
```

### Many-to-Many Relationship (Students and Courses)

Insert sample data:

```sql
INSERT INTO students (name, email) VALUES
('Alice Johnson', 'alice@example.com'),
('Bob Williams', 'bob@example.com');

INSERT INTO courses (title, credits) VALUES
('Introduction to Computer Science', 3),
('Database Management', 4),
('Web Development', 3);

INSERT INTO enrollments (student_id, course_id, enrollment_date) VALUES
(1, 1, '2023-09-01'),
(1, 2, '2023-09-01'),
(2, 2, '2023-09-02'),
(2, 3, '2023-09-02');
```

Query to retrieve students and their enrolled courses:

```sql
SELECT s.name, c.title, e.enrollment_date
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
ORDER BY s.name, c.title;
```

## 4. Performing Various Types of SQL Joins

First, let's insert some sample data into our Departments and Employees tables:

```sql
INSERT INTO departments (dept_name) VALUES
('HR'),
('IT'),
('Finance'),
('Marketing');

INSERT INTO employees (first_name, last_name, dept_id, manager_id) VALUES
('John', 'Doe', 1, NULL),
('Jane', 'Smith', 2, 1),
('Mike', 'Johnson', 2, 1),
('Emily', 'Brown', 3, 1),
('David', 'Wilson', NULL, NULL),
('Sarah', 'Lee', 4, 4);
```

Now, let's explore different types of joins:

### INNER JOIN

```sql
SELECT e.emp_id, e.first_name, e.last_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### LEFT JOIN

```sql
SELECT e.emp_id, e.first_name, e.last_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### RIGHT JOIN

```sql
SELECT e.emp_id, e.first_name, e.last_name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### FULL OUTER JOIN

```sql
SELECT e.emp_id, e.first_name, e.last_name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

### CROSS JOIN

```sql
SELECT e.first_name, e.last_name, d.dept_name
FROM employees e
CROSS JOIN departments d;
```

### SELF JOIN

```sql
SELECT e.first_name AS employee_first_name, 
       e.last_name AS employee_last_name,
       m.first_name AS manager_first_name, 
       m.last_name AS manager_last_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

## Conclusion

This tutorial has demonstrated how to use Databricks Unity Catalog to create and manage tables, implement different types of database relationships, and perform various SQL joins. Unity Catalog provides a unified governance layer for your data assets, allowing you to manage data access and permissions across your entire Databricks deployment.

Key takeaways:

1. Unity Catalog uses a three-level namespace: catalog, schema, and table.
2. Delta Lake is the default table format, providing ACID transactions and time travel capabilities.
3. Relationships between tables (One-to-One, One-to-Many, Many-to-Many) are implemented using primary and foreign key constraints.
4. SQL joins (INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF) work similarly to standard SQL, allowing you to combine data from multiple tables.

By leveraging these concepts in Unity Catalog, you can build robust, well-structured data models and perform complex data analysis tasks in your Databricks environment.
