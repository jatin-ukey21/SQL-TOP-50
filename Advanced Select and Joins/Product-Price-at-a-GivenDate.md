Here’s the **GitHub `README.md` style** version for your query:

---

````markdown
# 🛒 PostgreSQL Query — Find Product Prices on a Specific Date

## **Problem**
We are given a `Products` table:

| Column Name  | Type    | Description |
|--------------|---------|-------------|
| product_id   | int     | ID of the product |
| new_price    | int     | Price after change |
| change_date  | date    | Date of price change |

- `(product_id, change_date)` is the **primary key**.
- Initially, all products have a price of **10**.
- We need to find **the price of each product on `2019-08-16`**.

---

## **Approach**
We handle two **separate cases** and then combine results with `UNION ALL`:

### **Case 1** — Product price never changed before `2019-08-16`
- If the earliest change date is **after** our target date, the product price on `2019-08-16` is still the default `10`.

```sql
SELECT
  product_id,
  10 AS price
FROM
  Products
GROUP BY
  product_id
HAVING
  MIN(change_date) > '2019-08-16';
````

---

### **Case 2** — Product price changed before or on `2019-08-16`

* We find the **most recent change date** on or before the target date.
* Then we fetch the price from that change.

```sql
SELECT
  product_id,
  new_price AS price
FROM
  Products
WHERE
  (product_id, change_date) IN (
    SELECT
      product_id,
      MAX(change_date)
    FROM
      Products
    WHERE
      change_date <= '2019-08-16'
    GROUP BY
      product_id
  );
```

---

### **Combine Results**

We use `UNION ALL` because:

* The two cases are **mutually exclusive**.
* `UNION ALL` is **faster** than `UNION` since no duplicate check is needed.

```sql
SELECT
  product_id,
  10 AS price
FROM
  Products
GROUP BY
  product_id
HAVING
  MIN(change_date) > '2019-08-16'

UNION ALL

SELECT
  product_id,
  new_price AS price
FROM
  Products
WHERE
  (product_id, change_date) IN (
    SELECT
      product_id,
      MAX(change_date)
    FROM
      Products
    WHERE
      change_date <= '2019-08-16'
    GROUP BY
      product_id
  );
```

---

## **How It Works Internally**

1. **Case 1:**

   * Groups products.
   * Checks the earliest change date.
   * If it’s later than target date → returns `price = 10`.

2. **Case 2:**

   * Finds latest change date before target.
   * Matches `(product_id, change_date)` back to `Products` to get price.

3. **UNION ALL:** Combines both results into one table.

---

## **Example**

### Input:

| product\_id | new\_price | change\_date |
| ----------- | ---------- | ------------ |
| 1           | 20         | 2019-08-14   |
| 2           | 50         | 2019-08-14   |
| 1           | 30         | 2019-08-15   |
| 1           | 35         | 2019-08-16   |
| 2           | 65         | 2019-08-17   |
| 3           | 20         | 2019-08-18   |

### Output:

| product\_id | price |
| ----------- | ----- |
| 1           | 35    |
| 2           | 50    |
| 3           | 10    |

---

✅ **Final Insight:**

* **Case 1:** Products without a change before the date → `10`.
* **Case 2:** Products with a change before/on the date → last known price.

```
