# Count Salary Categories

## Problem Statement

We have a table `Accounts` that stores information about the **monthly income** for each bank account.  
We need to **count the number of accounts** that fall into three salary categories:

- **Low Salary**: incomes `< 20000`
- **Average Salary**: incomes between `20000` and `50000` (inclusive)
- **High Salary**: incomes `> 50000`

The output must **include all three categories** even if the count is `0`.

---

### Table: `Accounts`

| Column Name | Type | Description |
|-------------|------|-------------|
| account_id  | int  | Unique ID for each account |
| income      | int  | Monthly income for the account |

---

## Example

### Input:
| account_id | income |
|------------|--------|
| 3          | 108939 |
| 2          | 12747  |
| 8          | 87709  |
| 6          | 91796  |

### Output:
| category       | accounts_count |
|----------------|----------------|
| Low Salary     | 1              |
| Average Salary | 0              |
| High Salary    | 3              |

**Explanation:**
- Low Salary: Account `2` only.
- Average Salary: None.
- High Salary: Accounts `3`, `6`, and `8`.

---

## Approach

We write **three SELECT statements** (one for each category) and combine them using **`UNION ALL`** to ensure all categories appear in the result.  
We use `COUNT(CASE WHEN ... THEN 1 END)` to count rows matching each category condition.  
`ELSE NULL` ensures non-matching rows are ignored in the count.

---

## SQL Solution (PostgreSQL)

```sql
SELECT 'Low Salary' AS category,
       COUNT(CASE WHEN income < 20000 THEN 1 ELSE NULL END) AS accounts_count
FROM Accounts

UNION ALL

SELECT 'Average Salary' AS category,
       COUNT(CASE WHEN income >= 20000 AND income <= 50000 THEN 1 ELSE NULL END) AS accounts_count
FROM Accounts

UNION ALL

SELECT 'High Salary' AS category,
       COUNT(CASE WHEN income > 50000 THEN 1 ELSE NULL END) AS accounts_count
FROM Accounts;
