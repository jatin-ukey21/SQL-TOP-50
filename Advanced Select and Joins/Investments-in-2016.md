---

# Problem: Investment Value Analysis

We are given a table **Insurance** with the following columns:

* `pid`: Policy ID (Primary Key)
* `tiv_2015`: Total investment value in 2015
* `tiv_2016`: Total investment value in 2016
* `lat`: Latitude of the city
* `lon`: Longitude of the city

We need to calculate the **sum of tiv\_2016** for all policyholders who satisfy **both conditions**:

1. Their `tiv_2015` value must be shared with **at least one other policyholder**.
2. Their `(lat, lon)` pair must be **unique** (no other policyholder lives in the same city).

Finally, round the result to **two decimal places**.

---

## Example

**Input**

| pid | tiv\_2015 | tiv\_2016 | lat | lon |
| --- | --------- | --------- | --- | --- |
| 1   | 10        | 5         | 10  | 10  |
| 2   | 20        | 20        | 20  | 20  |
| 3   | 10        | 30        | 20  | 20  |
| 4   | 10        | 40        | 40  | 40  |

**Output**

| tiv\_2016 |
| --------- |
| 45.00     |

**Explanation**

* `pid = 1`: tiv\_2015 = 10 (shared with pid 3 & 4 ✅), location (10,10) unique ✅ → included
* `pid = 2`: tiv\_2015 = 20 (not shared ❌), location (20,20) not unique ❌ → excluded
* `pid = 3`: tiv\_2015 = 10 (shared ✅), but location (20,20) not unique ❌ → excluded
* `pid = 4`: tiv\_2015 = 10 (shared ✅), location (40,40) unique ✅ → included

Total = 5 + 40 = **45.00**

---

## PostgreSQL Solution

```sql
-- Write your PostgreSQL query statement below
SELECT ROUND(SUM(tiv_2016)::numeric,2) AS tiv_2016
FROM Insurance 
WHERE tiv_2015 IN (
    SELECT tiv_2015 
    FROM Insurance 
    GROUP BY tiv_2015
    HAVING COUNT(*) > 1
)
AND (lat, lon) IN (
    SELECT lat, lon 
    FROM Insurance 
    GROUP BY lat, lon
    HAVING COUNT(*) = 1
);
```

---

✅ This ensures we only sum values for policyholders that meet **both conditions**.

---
