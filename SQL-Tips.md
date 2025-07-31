# 1. Does MySQL support FULL OUTER JOIN explicitly?

MySQL doesn't support the `FULL OUTER JOIN` syntax directly. However, you can simulate it by combining a `LEFT JOIN` and a `RIGHT JOIN` using `UNION` (or `UNION ALL` if you want to keep duplicates). For example:

```sql
SELECT *
FROM table1
LEFT JOIN table2 ON table1.common_column = table2.common_column
UNION
SELECT *
FROM table1
RIGHT JOIN table2 ON table1.common_column = table2.common_column;
```

This query returns all rows from both tables, matching rows where possible, and including non-matching rows from both sides.