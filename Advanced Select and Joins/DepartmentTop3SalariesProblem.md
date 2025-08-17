````markdown
# High Earners per Department (Top 3 Unique Salaries)

Find all employees who are **high earners** in their department.  
A **high earner** = an employee whose salary is in the **top three unique salaries** **within their department**.

---

## Tables

### `Employee`
| Column        | Type | Notes                          |
|---------------|------|--------------------------------|
| id            | int  | PK                             |
| name          | text |                                |
| salary        | int  |                                |
| departmentId  | int  | FK → `Department.id`           |

### `Department`
| Column | Type | Notes |
|--------|------|-------|
| id     | int  | PK    |
| name   | text |       |

---

## Example Input

**Employee**
| id | name  | salary | departmentId |
|----|-------|--------|--------------|
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |

**Department**
| id | name  |
|----|-------|
| 1  | IT    |
| 2  | Sales |

---

## Goal

For each department, list **all employees** whose salary is one of the **top 3 distinct salaries** in that department, along with the department name and salary.

---

## Approach

1. **Partition by department** and **order salaries descending**.
2. Use **`DENSE_RANK()`** so **equal salaries share the same rank**.
3. Keep rows where **rank ≤ 3**.
4. **Join** back to `Department` to show department names.

> Why `DENSE_RANK()`?  
> Because the requirement is **top 3 unique salaries**. If two employees have the same salary, they should **both** be included under that one rank. `ROW_NUMBER()` would assign different numbers to ties, which is not what we want.

---

## PostgreSQL Solution

```sql
SELECT d.name AS "Department",
       e.name AS "Employee",
       e.salary AS "Salary"
FROM (
    SELECT id,
           name,
           salary,
           departmentId,
           DENSE_RANK() OVER (
               PARTITION BY departmentId
               ORDER BY salary DESC
           ) AS sal_rank
    FROM Employee
) e
JOIN Department d
  ON d.id = e.departmentId
WHERE e.sal_rank <= 3
ORDER BY d.name, e.salary DESC, e.name;
````

---

## How the Ranking Works (Intermediate View)

Compute `sal_rank` with `DENSE_RANK()` **per department**:

**Department: IT (id = 1)**
Salaries (desc): `90000, 85000, 85000, 70000, 69000`

| name  | salary | sal\_rank |
| ----- | ------ | --------- |
| Max   | 90000  | 1         |
| Joe   | 85000  | 2         |
| Randy | 85000  | 2         |
| Will  | 70000  | 3         |
| Janet | 69000  | 4         |

**Department: Sales (id = 2)**
Salaries (desc): `80000, 60000`

| name  | salary | sal\_rank |
| ----- | ------ | --------- |
| Henry | 80000  | 1         |
| Sam   | 60000  | 2         |

Filter `sal_rank ≤ 3` → keep:

* IT: Max (90000), Joe (85000), Randy (85000), Will (70000)
* Sales: Henry (80000), Sam (60000)

---

## Expected Output

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |

---

