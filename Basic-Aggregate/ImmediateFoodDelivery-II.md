# README — Percentage of Immediate First Orders

## Problem (Plain English)

You’re given a `Delivery` table with each customer’s orders:

* `order_date`: when the order was placed
* `customer_pref_delivery_date`: when the customer wanted it delivered

Definitions:

* **Immediate** = `order_date = customer_pref_delivery_date`
* **First order per customer** = the earliest `order_date` for that `customer_id` (guaranteed unique)

**Task:** Among only the **first orders** of all customers, compute the **percentage that are immediate**, rounded to 2 decimal places.

---

## Query 1 (CTE + ROW\_NUMBER)

```sql
WITH CTE AS (
  SELECT *
  FROM (
    SELECT
      *,
      ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS rn
    FROM Delivery
  ) t
  WHERE rn = 1
)
SELECT ROUND(
         SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1.0 ELSE 0.0 END)
         * 100.0 / COUNT(*),
         2
       ) AS immediate_percentage
FROM CTE;
```

### What it does (step by step)

1. **Window label first order per customer**

   * `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date)` ranks each customer’s orders by date; the earliest gets `rn = 1`.

2. **Keep only first orders**

   * Outer `WHERE rn = 1` filters to exactly one row per customer (their first order).

3. **Compute the percentage**

   * `CASE … THEN 1.0 ELSE 0.0 END` marks an order as **1** if it’s immediate, **0** otherwise.
   * `SUM(...) / COUNT(*)` gives the fraction of immediate first orders.
   * Multiply by `100.0` to get a percentage, `ROUND(..., 2)` for 2 decimals.

### Time & Space Complexity

* Let **n** = total rows, **c** = number of customers.
* The window function needs to **order** rows within each partition; overall this is typically **O(n log n)** time.
* Final aggregation is **O(c)**.
* Space: **O(n)** for the window/sort buffers.

**Index tip:** An index on `(customer_id, order_date)` can reduce sorting work and speed up the window operation.

---

## Query 2 (Semi-join with MIN + AVG trick)

```sql
SELECT ROUND(
         AVG(
           CASE
             WHEN order_date = customer_pref_delivery_date THEN 1
             ELSE 0
           END
         ) * 100.0,
         2
       ) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date) IN (
  SELECT customer_id, MIN(order_date)
  FROM Delivery
  GROUP BY customer_id
);
```

### What it does (step by step)

1. **Find each customer’s first order date**

   * Subquery `SELECT customer_id, MIN(order_date) … GROUP BY customer_id` returns one row per customer (their earliest order date).

2. **Filter to only those first-order rows**

   * `WHERE (customer_id, order_date) IN ( … )` keeps only the first order per customer (tuple comparison = semi-join).

3. **Compute the percentage with `AVG`**

   * `CASE … THEN 1 ELSE 0 END` yields 1 for immediate, 0 otherwise.
   * `AVG(...)` directly gives the fraction of immediate first orders (since it averages 0s and 1s).
   * Multiply by `100.0`, then `ROUND(..., 2)`.

### Time & Space Complexity

* The subquery is a `GROUP BY` with `MIN(order_date)` over **n** rows → typically **O(n)** (hash aggregate) time.
* The `IN` (tuple) is executed as a semi-join; with hashing it’s **O(n)** expected.
* Final `AVG` runs over **c** rows.
* Space: **O(c)** for the grouped result.

**Index tip:** An index on `(customer_id, order_date)` helps both finding `MIN(order_date)` and probing the semi-join.

---

## Which one to choose?

* **Query 2** is generally **faster** and **simpler**: it avoids window sorting and uses a straightforward `GROUP BY` + semi-join.
* **Query 1** is perfectly valid and idiomatic when you already need row-numbered partitions, but it’s typically more expensive due to the per-partition ordering.

---

## Rounding & Numeric Notes (PostgreSQL)

* Use a decimal in math to avoid integer division: e.g., `1.0`, `100.0`.
* `ROUND(numeric, 2)` expects a `numeric` type; your expressions here naturally become numeric due to the decimal literals.
