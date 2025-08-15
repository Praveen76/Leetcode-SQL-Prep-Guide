# Q1: Explain the difference between "ROWS BETWEEN 6 PRECEDING AND CURRENT ROW" and " RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW" with the help of an example.

### Answer:

`ROWS BETWEEN 6 PRECEDING` and `RANGE BETWEEN INTERVAL 6 DAY PRECEDING` behave differently.

---

## **Example Data**

Imagine our daily revenue table looks like this:

| dt         | revenue |
| ---------- | ------- |
| 2025-08-01 | 100     |
| 2025-08-02 | 200     |
| 2025-08-05 | 300     |
| 2025-08-06 | 400     |
| 2025-08-07 | 500     |

Note: there are **missing dates** (no rows for `2025-08-03` and `2025-08-04`).

---

## **Using ROWS BETWEEN 2 PRECEDING AND CURRENT ROW**

```sql
SUM(revenue) OVER (
  ORDER BY dt
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

This means: “Take the **current row** plus the **previous 2 rows in the result set**, regardless of how many days apart they are.”

Step-by-step:

* For `2025-08-07`:
  Previous 2 rows are `2025-08-05` and `2025-08-06`,
  so sum = **300 + 400 + 500 = 1200**.
  Notice that 08-01 and 08-02 are ignored even though they are only 6 days apart.

---

## **Using RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW**

```sql
SUM(revenue) OVER (
  ORDER BY dt
  RANGE BETWEEN INTERVAL 6 DAY PRECEDING AND CURRENT ROW
)
```

This means: “Take the current date, look back **6 calendar days**, and include all rows whose `dt` falls in that range.”


Step-by-step:

* For `2025-08-07`:
  The date window is `2025-08-01` to `2025-08-07`.
  That includes **100 + 200 + 300 + 400 + 500 = 1500**.
  Even though 08-03 and 08-04 are missing in the table, those days are still in the time window.

---

### **Key Differences**

| Feature               | ROWS                                                         | RANGE                                                |
| --------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| Basis                 | Physical row count in result set                             | Actual value range (dates, numbers)                  |
| Gaps in dates matter? | **Yes** (skips them entirely)                                | **No** (window covers them)                          |
| Common use            | When you have exactly 1 row per unit (e.g., per day, per ID) | When you want true time-based or value-based windows |

---

I’ve put a side-by-side table in your workspace showing both calculations.

What to notice:

* On **2025-08-07**:

  * **ROWS** (prev 2 rows + current) = 300 + 400 + 500 = **1200**.
  * **RANGE** (last 6 calendar days) spans 2025-08-01…2025-08-07, so 100+200+300+400+500 = **1500**.
* Because **RANGE** uses actual dates, it “sees” 08-01 and 08-02 even though there are gaps (no rows on 08-03, 08-04). **ROWS** only looks at the last two physical rows, ignoring earlier days within the 6-day calendar window.



