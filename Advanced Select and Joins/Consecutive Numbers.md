# SQL Problem — Find Numbers Appearing at Least Three Times Consecutively

## Problem Statement
We are given a table `Logs`:

| Column Name | Type    |
|-------------|---------|
| id          | int     |
| num         | varchar |

- `id` is the primary key (auto-incremented starting from 1).
- We need to **find all numbers that appear at least three times consecutively** in the table.
- The output should return **each qualifying number only once**, regardless of how many such streaks it has.

---

## Example

### Input:
**Logs table:**
| id  | num |
|-----|-----|
| 1   | 1   |
| 2   | 1   |
| 3   | 1   |
| 4   | 2   |
| 5   | 1   |
| 6   | 2   |
| 7   | 2   |
| 8   | 2   |

### Output:
| ConsecutiveNum |
|----------------|
| 1              |
| 2              |

---

## Approach

### Idea
We can check for **three consecutive rows** having the same `num` by:
1. Joining the table to itself **three times** (`l1`, `l2`, `l3`).
2. Matching:
   - `l1.id = l2.id - 1` → `l2` is right after `l1`
   - `l2.id = l3.id - 1` → `l3` is right after `l2`
3. Ensuring:
   - `l1.num = l2.num = l3.num`

---

## SQL Solution

```sql
SELECT DISTINCT l1.num AS ConsecutiveNum
FROM Logs l1
JOIN Logs l2 
  ON l1.id = l2.id - 1
JOIN Logs l3 
  ON l2.id = l3.id - 1
WHERE l1.num = l2.num 
  AND l2.num = l3.num;
```

---

## Step-by-Step Execution

1. **Self-joins**:
   - First join ensures `l2` is the next row after `l1`.
   - Second join ensures `l3` is the next row after `l2`.

2. **Filtering**:
   - Keep only rows where all three numbers match.

3. **DISTINCT**:
   - Removes duplicates in case a number appears in multiple different consecutive triplets.

---

## Visual Join Example

Given the example input:

| l1.id | l1.num | l2.id | l2.num | l3.id | l3.num |
|-------|--------|-------|--------|-------|--------|
| 1     | 1      | 2     | 1      | 3     | 1      |
| 2     | 1      | 3     | 1      | 4     | 2      |
| 6     | 2      | 7     | 2      | 8     | 2      |

After `WHERE` filtering:
- `(1, 1, 1)` → qualifies (num = 1)
- `(6, 2, 2)` → qualifies (num = 2)

---

## Final Output:
| ConsecutiveNum |
|----------------|
| 1              |
| 2              |

---
