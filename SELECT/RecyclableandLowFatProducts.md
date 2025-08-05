
# 📊 Problem: Recyclable and Low Fat Products

## 📘 Description

> **Problem Statement:**

Table: `Products`

```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| product_id  | int      |
| low_fats    | enum     |
| recyclable  | enum     |
+-------------+----------+
```

- `product_id` is the primary key (unique values).
- `low_fats` is an ENUM with values `'Y'` or `'N'`, where `'Y'` means the product is low fat.
- `recyclable` is an ENUM with values `'Y'` or `'N'`, where `'Y'` means the product is recyclable.

Write a SQL query to **find the IDs of products that are both low fat and recyclable**.

---

## 📥 Input Example

Table: `Products`

```
+------------+-----------+-------------+
| product_id | low_fats  | recyclable  |
+------------+-----------+-------------+
|     0      |    Y      |     N       |
|     1      |    Y      |     Y       |
|     2      |    N      |     Y       |
|     3      |    Y      |     Y       |
|     4      |    N      |     N       |
+------------+-----------+-------------+
```

---

## 🎯 Output Example

```
+------------+
| product_id |
+------------+
|     1      |
|     3      |
+------------+
```

## 🧠 Explanation

Only products with both `low_fats = 'Y'` **and** `recyclable = 'Y'` qualify.  
In this case, `product_id` 1 and 3 satisfy both conditions.

---

## 💡 SQL Query

```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y';
```
