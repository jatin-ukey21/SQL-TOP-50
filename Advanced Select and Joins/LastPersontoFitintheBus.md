# Last Person to Fit in the Bus

## Problem Statement

We have a queue of people waiting to board a bus. The bus has a **weight limit of 1000 kg**.  
People board one at a time in order of their `turn`.  
We need to **find the name of the last person who can board without exceeding the weight limit**.

**Table: `Queue`**

| Column Name  | Type    | Description |
|--------------|---------|-------------|
| person_id    | int     | Unique ID for each person |
| person_name  | varchar | Name of the person |
| weight       | int     | Person's weight in kilograms |
| turn         | int     | Boarding order (1 = first person to board) |

- `person_id` and `turn` are unique and range from 1 to `n`.
- The **first person always fits** (weight ≤ 1000 kg).

---

### Example Input
| person_id | person_name | weight | turn |
|-----------|-------------|--------|------|
| 5         | Alice       | 250    | 1    |
| 4         | Bob         | 175    | 5    |
| 3         | Alex        | 350    | 2    |
| 6         | John Cena   | 400    | 3    |
| 1         | Winston     | 500    | 6    |
| 2         | Marie       | 200    | 4    |

### Expected Output
| person_name |
|-------------|
| John Cena   |

**Explanation:**

Ordered by `turn`:

| Turn | Name      | Weight | Total Weight | Status         |
|------|-----------|--------|--------------|----------------|
| 1    | Alice     | 250    | 250          | Boarded        |
| 2    | Alex      | 350    | 600          | Boarded        |
| 3    | John Cena | 400    | 1000         | **Last to fit**|
| 4    | Marie     | 200    | 1200         | Cannot board   |
| 5    | Bob       | 175    | —            | Skipped        |
| 6    | Winston   | 500    | —            | Skipped        |

---

## Approach

We calculate a **running total** of the `weight` column ordered by `turn` using the **`SUM(...) OVER (ORDER BY turn)`** window function.  
This lets us track the cumulative weight as people board.

Then:
1. Filter to only those rows where `total_weight <= 1000`.
2. Order by `total_weight` descending.
3. Pick the **first row** — which will be the last person to fit.

---

## SQL Solution (PostgreSQL)

```sql
SELECT person_name 
FROM (
    SELECT person_name,
           SUM(weight) OVER (ORDER BY turn) AS total_weight
    FROM Queue
) t
WHERE total_weight <= 1000
ORDER BY total_weight DESC
LIMIT 1;
