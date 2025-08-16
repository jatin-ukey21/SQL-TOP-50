# SQL Query: Movies & Users Ratings

This query solves two problems using **PostgreSQL**:

1. **Find the name of the user who has rated the greatest number of movies.**  
   - In case of a tie, return the lexicographically smaller user name.

2. **Find the movie name with the highest average rating in February 2020.**  
   - In case of a tie, return the lexicographically smaller movie name.

---

## Example

**Input Tables**

**Movies**
| movie_id | title     |
|----------|-----------|
| 1        | Avengers  |
| 2        | Frozen 2  |
| 3        | Joker     |

**Users**
| user_id | name   |
|---------|--------|
| 1       | Daniel |
| 2       | Monica |
| 3       | Maria  |
| 4       | James  |

**MovieRating**
| movie_id | user_id | rating | created_at |
|----------|---------|--------|------------|
| 1        | 1       | 3      | 2020-01-12 |
| 1        | 2       | 4      | 2020-02-11 |
| 1        | 3       | 2      | 2020-02-12 |
| 1        | 4       | 1      | 2020-01-01 |
| 2        | 1       | 5      | 2020-02-17 |
| 2        | 2       | 2      | 2020-02-01 |
| 2        | 3       | 2      | 2020-03-01 |
| 3        | 1       | 3      | 2020-02-22 |
| 3        | 2       | 4      | 2020-02-25 |

**Output**
| results  |
|----------|
| Daniel   |
| Frozen 2 |

---

## PostgreSQL Query

```sql
-- Find the top user and top movie according to conditions
(SELECT u.name AS results
 FROM Users u
 JOIN MovieRating r
   ON u.user_id = r.user_id
 GROUP BY u.name
 ORDER BY COUNT(r.rating) DESC, u.name
 LIMIT 1)

UNION ALL

(SELECT m.title AS results
 FROM Movies m
 JOIN MovieRating r 
   ON m.movie_id = r.movie_id
 WHERE EXTRACT(MONTH FROM r.created_at) = '02'
   AND EXTRACT(YEAR FROM r.created_at) = '2020'
 GROUP BY m.title
 ORDER BY AVG(r.rating) DESC, m.title
 LIMIT 1);
