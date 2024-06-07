# SQL Exercises Progress

## Postgres setup

```shell
podman run --replace --name postgres-tutorial -e POSTGRES_HOST_AUTH_METHOD=trust -e POSTGRES_PASSWORD=password -e POSTGRES_DB=tutorial -d -p 5432:5432 postgres
podman exec -it postgres-tutorial psql -U postgres -d tutorial
```

## Tables setup

```sql
CREATE TABLE employees
(
    employee_id   INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT,
    project_id    INT
);
```

```sql
CREATE TABLE departments
(
    department_id   INT PRIMARY KEY,
    department_name VARCHAR(100)
);
```

```sql
CREATE TABLE projects
(
    project_id   INT PRIMARY KEY,
    project_name VARCHAR(100),
    budget       DECIMAL(10, 2)
);
```

### Sample data setup

```sql
-- Insert data into departments
INSERT INTO departments (department_id, department_name)
VALUES (1, 'HR'),
       (2, 'IT'),
       (3, 'Finance'),
       (4, 'Marketing'),
       (5, 'Sales');

-- Insert data into projects
INSERT INTO projects (project_id, project_name, budget)
VALUES (1, 'Project Alpha', 10000.00),
       (2, 'Project Beta', 20000.00),
       (3, 'Project Gamma', 15000.00),
       (4, 'Project Delta', 5000.00),
       (5, 'Project Epsilon', 25000.00);

-- Insert data into employees
INSERT INTO employees (employee_id, employee_name, department_id, project_id)
VALUES (1, 'Alice', 1, 101),
       (2, 'Bob', 2, 102),
       (3, 'Charlie', 3, 103),
       (4, 'David', 2, 101),
       (5, 'Eve', 1, NULL),
       (6, 'Frank', NULL, 103),
       (7, 'Grace', 4, 104),
       (8, 'Hank', 5, 105),
       (9, 'Ivy', 5, NULL),
       (10, 'Jack', NULL, NULL);
```

## Exercises

Completion mark: ✅

### 1. ✅ Find the top 3 projects with the highest budgets

**Instruction**: List the names and budgets of the top 3 projects with the highest budgets.
<details>
<summary>Solution</summary>

```sql
SELECT project_name, budget
FROM projects
ORDER BY budget DESC LIMIT 3;
```

</details>

### 2. ✅ List employees who have not been assigned to any project or department

**Instruction**: Show the names of employees who do not have a project or department assigned.
<details>
<summary>Solution</summary>

```sql
SELECT employee_name
FROM employees
WHERE department_id IS NULL
   OR project_id IS NULL;
```

</details>

### 3. ✅ List all employees with their department names and project names

**Instruction**: Show all employees with their respective department names and project names. Include employees without a department or project.

<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name, d.department_name, p.project_name
FROM employees e
         LEFT JOIN departments d ON e.department_id = d.department_id
         LEFT JOIN projects p ON e.project_id = p.project_id;
```

</details>

### 4. ✅ List all departments with the total number of employees and the total budget of their projects

**Instruction**: Show each department with the count of employees and the sum of project budgets. Include departments without employees or projects.
<details>
<summary>Solution</summary>

```sql
SELECT d.department_name, COUNT(e.employee_id) AS employee_count, SUM(p.budget) AS total_budget
FROM departments d
         LEFT JOIN employees e ON d.department_id = e.department_id
         LEFT JOIN projects p ON e.project_id = p.project_id
GROUP BY d.department_name;
```

</details>

### 5. ✅ Count the number of employees in each department and each project

**Instruction**: Show the department name, project name, and the number of employees in each combination.
<details>
<summary>Solution</summary>

```sql
SELECT d.department_name, p.project_name, COUNT(e.employee_id) AS employee_count
FROM employees e
         LEFT JOIN departments d ON e.department_id = d.department_id
         LEFT JOIN projects p ON e.project_id = p.project_id
GROUP BY d.department_name, p.project_name;
```

</details>

### 6. ✅ Add 'side_projects' table

**INSTRUCTION**: Create a new table called `side_projects` with the following columns:

- `project_id` with type `INT` and primary key
- `project_name` with type `VARCHAR(100)`
- `budget` with type `DECIMAL(10, 2)`
- `volunteers_count` with type `INT`
- `payment_status` with type `BOOL`

Then create 3 new side projects with the following details:

- Project name: 'Project Zeta', Budget: 5000.00, Volunteers: 3, Payment: TRUE
- Project name: 'Project Theta', Budget: 8000.00, Volunteers: 5, Payment: FALSE
- Project name: 'Project Iota', Budget: 3000.00, Volunteers: 2, Payment: FALSE

<details>
<summary>Solution</summary>

```sql
CREATE TABLE side_projects
(
    project_id       INT PRIMARY KEY,
    project_name     VARCHAR(100),
    budget           DECIMAL(10, 2),
    volunteers_count INT,
    payment_status   BOOLEAN
);

INSERT INTO side_projects (project_id, project_name, budget, volunteers_count, payment_status)
VALUES (1, 'Project Zeta', 5000.00, 3, TRUE),
       (2, 'Project Theta', 8000.00, 5, FALSE),
       (3, 'Project Iota', 3000.00, 2, FALSE);
```

</details>

### 7. ✅ Add employees_side_projects table

**INSTRUCTION**: Because each employee can work on multiple side projects,
we need to create a new table called `employees_side_projects` with the following columns:

- `employee_id` with type `INT`
- `project_id` with type `INT`

Then assign the following employees to the side projects:

- `employee_id` 1 to `side_project_id` 1, 2
- `employee_id` 2 to `side_project_id` 1
- `employee_id` 3 to `side_project_id` 1, 2, 3
- `employee_id` 7 to `side_project_id` 3
- `employee_id` 8 to `side_project_id` 2, 3
- `employee_id` 10 to `side_project_id` 1, 3

**NOTES**: The `employee_id` and `project_id` columns should be a composite primary key.
The `employee_id` and `project_id` columns should be a foreign key references to the `employees` and `side_projects` tables respectively.

<details>
<summary>Solution</summary>

```sql
CREATE TABLE employees_side_projects
(
    employee_id     INT,
    side_project_id INT,
    PRIMARY KEY (employee_id, side_project_id),
    FOREIGN KEY (employee_id) REFERENCES employees (employee_id),
    FOREIGN KEY (side_project_id) REFERENCES side_projects (project_id)
);

INSERT INTO employees_side_projects (employee_id, side_project_id)
VALUES (1, 1),
       (1, 2),
       (2, 1),
       (3, 1),
       (3, 2),
       (3, 3),
       (7, 3),
       (8, 2),
       (8, 3),
       (10, 1),
       (10, 3);

```

</details>

### 8. ✅ Find employees working on multiple projects

**Instruction**: List the names of employees who are working on more than one project (main or side).
<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name
FROM employees e
         JOIN employees_side_projects esp ON e.employee_id = esp.employee_id
GROUP BY e.employee_name
HAVING COUNT(esp.side_project_id) + COUNT(e.project_id) > 1;
```

</details>

### 9. ✅ Find the total number of projects each employee is working on and sort in descending order

**Instruction**: Show each employee's name and the total number of projects they are working on.
<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name, COUNT(DISTINCT project_id) + COUNT(DISTINCT side_project_id) AS total_projects
FROM employees e
         LEFT JOIN employees_side_projects esp ON e.employee_id = esp.employee_id
GROUP BY e.employee_name
ORDER BY total_projects DESC;
```

### 10. ✅ List all projects

**Instruction**: List all projects (main or side) with their names and order by name in ascending order.

<details>
<summary>Solution</summary>

```sql
SELECT project_name
FROM projects
UNION
SELECT project_name
FROM side_projects
ORDER BY project_name DESC;
```

</details>

### 11. ✅ List projects and sort by number of people

**Instruction**: List all projects (main or side) with their names and order by the number of people working on them in descending order.

<details>
<summary>Solution</summary>

```sql
SELECT project_name,
       Count(e.employee_id) AS people
FROM employees e
         LEFT JOIN projects p
                   ON e.project_id = p.project_id
WHERE project_name IS NOT NULL
GROUP BY project_name
UNION
SELECT project_name,
       Count(e.employee_id) AS people
FROM employees_side_projects s
         LEFT JOIN employees e
                   ON s.employee_id = e.employee_id
         LEFT JOIN side_projects p
                   ON s.side_project_id = p.project_id
GROUP BY project_name
ORDER BY people DESC

```

</details>

----------------------------------------------------------------
----------------------------------------------------------------
----------------------------------------------------------------
----------------------------------------------------------------
----------------------------------------------------------------
----------------------------------------------------------------

### 9. List employees and the total budget of the projects they are working on

**Instruction**: Show each employee's name and the total budget of the projects they are assigned to.
<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name, SUM(p.budget) AS total_budget
FROM employees e
         LEFT JOIN projects p ON e.project_id = p.project_id
GROUP BY e.employee_name;
```

</details>

### 10. Find departments with more than 2 projects

**Instruction**: List the names of departments that have employees working on more than 2 different projects.
<details>
<summary>Solution</summary>

```sql
SELECT d.department_name
FROM departments d
         JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name
HAVING COUNT(DISTINCT e.project_id) > 2;
```

</details>

### 11. Find the employee with the highest total project budget

**Instruction**: Show the name of the employee who is working on projects with the highest combined budget.
<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name
FROM employees e
         JOIN projects p ON e.project_id = p.project_id
GROUP BY e.employee_name
ORDER BY SUM(p.budget) DESC LIMIT 1;
```

</details>

### 12. List all projects with their average employee count per department

**Instruction**: Show each project's name and the average number of employees per department working on that project.
<details>
<summary>Solution</summary>

```sql
SELECT p.project_name, AVG(employee_count) AS avg_employee_count_per_dept
FROM (SELECT p.project_id, p.project_name, d.department_id, COUNT(e.employee_id) AS employee_count
      FROM projects p
               LEFT JOIN employees e ON p.project_id = e.project_id
               LEFT JOIN departments d ON e.department_id = d.department_id
      GROUP BY p.project_id, p.project_name, d.department_id) subquery
GROUP BY project_name;
```

</details>

### 13. Find departments with no employees assigned to projects with budgets over $15,000

**Instruction**: List the names of departments that do not have any employees working on projects with a budget over $15,000.
<details>
<summary>Solution</summary>

```sql
SELECT d.department_name
FROM departments d
         LEFT JOIN employees e ON d.department_id = e.department_id
         LEFT JOIN projects p ON e.project_id = p.project_id
GROUP BY d.department_name
HAVING SUM(CASE WHEN p.budget > 15000 THEN 1 ELSE 0 END) = 0;
```

</details>

### 14. List all employees and their next project based on alphabetical order of project names

**Instruction**: Show each employee's name and the name of their next project in alphabetical order, or NULL if they do not have a project.
<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name,
       (SELECT MIN(p.project_name)
        FROM projects p
        WHERE p.project_name > COALESCE((SELECT project_name FROM projects WHERE project_id = e.project_id), '')) AS next_project
FROM employees e;
```

</details>

### 15. Find the average budget of projects for each department

**Instruction**: Show each department's name and the average budget of the projects their employees are assigned to.
<details>
<summary>Solution</summary>

```sql
SELECT d.department_name, AVG(p.budget) AS avg_budget
FROM departments d
         JOIN employees e ON d.department_id = e.department_id
         JOIN projects p ON e.project_id = p.project_id
GROUP BY d.department_name;
```

</details>
