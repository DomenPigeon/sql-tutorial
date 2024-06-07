# SQL Exercises Progress

## Postgres setup

```shell
podman run --replace --name postgres-tutorial -e POSTGRES_HOST_AUTH_METHOD=trust -e POSTGRES_PASSWORD=password -e POSTGRES_DB=tutorial -d -p 5432:5432 postgres
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
VALUES (101, 'Project Alpha', 10000.00),
       (102, 'Project Beta', 20000.00),
       (103, 'Project Gamma', 15000.00),
       (104, 'Project Delta', 5000.00),
       (105, 'Project Epsilon', 25000.00);

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

### 1. List all employees with their department names and project names

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

### 2. List all departments with the total number of employees and the total budget of their projects

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

### 3. Find the top 3 projects with the highest budgets

**Instruction**: List the names and budgets of the top 3 projects with the highest budgets.
<details>
<summary>Solution</summary>

```sql
SELECT project_name, budget
FROM projects
ORDER BY budget DESC LIMIT 3;
```

</details>

### 4. List employees who have not been assigned to any project or department

**Instruction**: Show the names of employees who do not have a project or department assigned.
<details>
<summary>Solution</summary>

```sql
SELECT employee_name
FROM employees
WHERE department_id IS NULL
  AND project_id IS NULL;
```

</details>

### 5. Count the number of employees in each department and each project

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

### 6. Find employees working on multiple projects

**Instruction**: List the names of employees who are assigned to more than one project.
<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name
FROM employees e
         JOIN (SELECT employee_id
               FROM employees
               GROUP BY employee_id
               HAVING COUNT(DISTINCT project_id) > 1) multi_projects ON e.employee_id = multi_projects.employee_id;
```

</details>

### 7. List departments without projects

**Instruction**: Show the names of departments that do not have any employees assigned to projects.
<details>
<summary>Solution</summary>

```sql
SELECT d.department_name
FROM departments d
         LEFT JOIN employees e ON d.department_id = e.department_id
         LEFT JOIN projects p ON e.project_id = p.project_id
WHERE p.project_id IS NULL;
```

</details>

### 8. Find the project with the most employees and its budget

**Instruction**: Show the name and budget of the project that has the most employees assigned.
<details>
<summary>Solution</summary>

```sql
SELECT p.project_name, p.budget
FROM projects p
         JOIN employees e ON p.project_id = e.project_id
GROUP BY p.project_name, p.budget
ORDER BY COUNT(e.employee_id) DESC LIMIT 1;
```

</details>

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
