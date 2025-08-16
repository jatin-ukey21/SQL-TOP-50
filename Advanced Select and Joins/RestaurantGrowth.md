---

# 7-Day Moving Average of Customer Payments

## Problem

We need to compute the **7-day moving average** of how much customers paid in the restaurant.

* Each window = `current day + 6 days before`.
* The result must include the **total amount paid in the 7 days window** and the **average (rounded to 2 decimals)**.
* Return only dates where a full 7-day window exists.

---

## Example

### Input:

```
Customer table:
+-------------+--------------+--------------+-------------+
| customer_id | name         | visited_on   | amount      |
+-------------+--------------+--------------+-------------+
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 
```

### Output:

```
+--------------+--------------+----------------+
| visited_on   | amount       | average_amount |
+--------------+--------------+----------------+
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |
```

---

## PostgreSQL Solution

```sql
-- Compute 7-day moving average of payments
SELECT 
    visited_on,
    (
        -- Total amount in the last 7 days (current day + 6 days before)
        SELECT SUM(amount)
        FROM Customer
        WHERE visited_on BETWEEN (c.visited_on - INTERVAL '6 DAY') AND c.visited_on
    ) AS amount,
    ROUND(
        (
            -- Average = sum over 7 days / 7
            SELECT SUM(amount)/7.0
            FROM Customer
            WHERE visited_on BETWEEN (c.visited_on - INTERVAL '6 DAY') AND c.visited_on
        ),
        2
    ) AS average_amount
FROM Customer c
-- Only consider dates where a full 7-day window exists
WHERE visited_on >= (
        SELECT MIN(visited_on) + INTERVAL '6 DAY'
        FROM Customer
    )
GROUP BY visited_on
ORDER BY visited_on;

-- Outer level: visited_on is the same as c.visited_on (unambiguous → alias optional).
-- Inside subqueries: you must use c.visited_on to refer to the outer row; otherwise, it will point to the inner table’s column.
```

---

## Explanation of the Logic

1. **Outer Query (`Customer c`)**

   * Iterates over each date (`visited_on`).
   * Only keeps dates where a full 7-day window exists (starting from `MIN(visited_on) + 6 days`).

2. **Subquery for `amount`**

   * Calculates the **total payments** between `(c.visited_on - 6 days)` and `c.visited_on`.

3. **Subquery for `average_amount`**

   * Takes the same **7-day sum** but divides by `7.0` to get average.
   * Rounded to 2 decimal places.

4. **GROUP BY visited\_on**

   * Ensures aggregation per day.

5. **ORDER BY visited\_on**

   * Returns results in chronological order.

---

✅ This query follows the problem exactly and produces the required moving averages.

---

