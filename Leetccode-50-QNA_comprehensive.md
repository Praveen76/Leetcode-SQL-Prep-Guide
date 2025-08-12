## Q1: [Rising Temperature](https://leetcode.com/problems/rising-temperature/)
**Note: A very good question to understand self-join by merging 2 tables on consecutive dates.**

**Method 1:**

```sql
SELECT w1.id
FROM Weather w1, Weather w2
WHERE w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
AND w1.temperature > w2.temperature;
```

```plaintext
# Output with all columns:

| id | recordDate | temperature | id | recordDate | temperature |
| -- | ---------- | ----------- | -- | ---------- | ----------- |
| 2  | 2015-01-02 | 25          | 1  | 2015-01-01 | 10          |
| 3  | 2015-01-03 | 20          | 2  | 2015-01-02 | 25          |
| 4  | 2015-01-04 | 30          | 3  | 2015-01-03 | 20          |

```

```plaintext
-- # Output of final table:
| id | recordDate | temperature | id | recordDate | temperature |
| -- | ---------- | ----------- | -- | ---------- | ----------- |
| 2  | 2015-01-02 | 25          | 1  | 2015-01-01 | 10          |
| 4  | 2015-01-04 | 30          | 3  | 2015-01-03 | 20          |
```
**Method 2:**
```sql
SELECT w1.id
FROM Weather w1, Weather w2
WHERE DATEDIFF(w1.recordDate, w2.recordDate) = 1
 AND w1.temperature > w2.temperature;
```
**Method 3:**
```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2 ON w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
WHERE w1.temperature > w2.temperature
```
```plaintext
-- # Output with all columns:
| id | recordDate | temperature | id | recordDate | temperature |
| -- | ---------- | ----------- | -- | ---------- | ----------- |
| 2  | 2015-01-02 | 25          | 1  | 2015-01-01 | 10          |
| 4  | 2015-01-04 | 30          | 3  | 2015-01-03 | 20          |
```
## Q2: [Students and Examinations](https://leetcode.com/problems/students-and-examinations/)

```sql
SELECT s.student_id, s.student_name, sub.subject_name, COUNT(e.student_id) AS attended_exams
FROM Students s

CROSS JOIN Subjects sub
LEFT JOIN Examinations e ON s.student_id = e.student_id AND sub.subject_name = e.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```

## Q3: [Biggest Single Number](https://leetcode.com/problems/biggest-single-number/)

**Method 1:**
```sql
SELECT

MAX(num) AS num

FROM (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(num) = 1
) AS unique_numbers;

```
**Method 1.1:**
```sql
WITH CTE AS(
SELECT num
from MyNumbers 
GROUP BY num
HAVING COUNT(num)=1

)
SELECT max(num) as num
FROM CTE
```
**Method 2:**
```sql
SELECT MAX(IF(cnt = 1, num, NULL)) AS num
FROM (
    SELECT num, COUNT(*) AS cnt
    FROM MyNumbers
    GROUP BY num
) AS t;
```

**Method 3:**
```sql
WITH CTE AS(    
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(num) = 1
)
SELECT max(num) as num
FROM CTE
```

## Q4: [Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)

**Method A:**
```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM   (
        SELECT salary,
               DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
        FROM   Employee
       ) t
WHERE  rnk = 2;
```
**Method B:**
```sql
WITH ranked_salaries AS (                          
    SELECT  salary,
            DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM    Employee
)
SELECT  MAX(salary) AS SecondHighestSalary         
FROM    ranked_salaries
WHERE   rnk = 2;                                   

```


**Method 1**
```sql
select 
CASE WHEN COUNT(*) = 0 THEN NULL ELSE B.salary end as SecondHighestSalary
from(
    Select 
    id,salary,
    DENSE_RANK() OVER(order by salary desc) as Ranking
    from Employee
) B
Where B.Ranking=2
;
```

**Method 2**

```sql
select 

if(COUNT(*) = 0 , NULL, B.salary) as SecondHighestSalary

from(
    Select 
    id,salary,
    DENSE_RANK() OVER(order by salary desc) as Ranking
    from Employee
) B
Where B.Ranking=2
;
```
**Method 2.1**
```sql
WITH CTE AS (
    Select 
    id,salary,
    DENSE_RANK() OVER(order by salary desc) as Ranking
    from Employee
) 

SELECT 
if(COUNT(*) = 0 , NULL, salary) as SecondHighestSalary
from CTE
Where Ranking=2
;
```

## Q5: [Immediate Food Delivery II](https://leetcode.com/problems/immediate-food-delivery-ii/)

**Method 1:**
**Step 1: Write subquery to find first order of all customers- earliest orders**
**Step 2: Fetch records from table from above step, where order_date= customer_pref_delivery_date i.e. immediate order**
**Step 3: Calculate immediate_percentage**

**Step 2 & 3**
```sql

select 
round(sum(if (B.order_date=B.customer_pref_delivery_date, 1, 0 ))*100/count(*),2) as immediate_percentage
from( 
    # Step 1: find first order of all customers- earliest orders
    select *,
    DENSE_RANK() over(Partition by customer_id order by order_date) as Ranking
    from Delivery
) B
Where B.Ranking =1
```

**Method 1.1:**
```sql
WITH CTE AS (
SELECT *,
DENSE_RANK() OVER( PARTITION BY customer_id  ORDER BY order_date ) as rnk
FROM Delivery
)

SELECT 
round(sum(if (B.order_date=B.customer_pref_delivery_date, 1, 0 ))*100/count(*),2) as immediate_percentage

FROM CTE B
WHERE rnk=1

```

** Method 1.1.1:**

```sql
WITH cte AS (
  SELECT
    customer_id,
    order_date,
    customer_pref_delivery_date,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) rn
  FROM Delivery
)
SELECT
  ROUND(100 * AVG(IF(order_date = customer_pref_delivery_date, 1, 0)), 2) AS immediate_percentage
FROM cte
WHERE rn = 1;

```

**Method 1.2:**

```sql

WITH CTE AS (
    SELECT *,
        DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rnk
    FROM Delivery
)
SELECT 
    -- ROUND(SUM(IF(order_date = customer_pref_delivery_date, 1 ,0)) * 100.0 / COUNT(*), 2) AS immediate_percentage
    ROUND(SUM(order_date = customer_pref_delivery_date) * 100.0 / COUNT(*), 2) AS immediate_percentage
FROM CTE
WHERE rnk = 1;
```

**Method 2:**

```sql
WITH FirstOrders AS (
    SELECT 
        delivery_id,
        customer_id,
        order_date,
        customer_pref_delivery_date
    FROM Delivery
    WHERE (customer_id, order_date) IN (
        SELECT customer_id, MIN(order_date)
        FROM Delivery
        GROUP BY customer_id
    )
)
SELECT 
    ROUND(
        SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1 ELSE 0 END) * 100.0 
        / COUNT(*), 
        2
    ) AS immediate_percentage
FROM FirstOrders;
```

**Wrong Query:**
```sql
with cte as (

    select customer_id,customer_pref_delivery_date,
    min(order_date) as firstOrder
    -- min(order_date) over(partition by customer_id) as firstOrder
    from Delivery
    group by customer_id
)

select 
round(
100*count(if(firstOrder=customer_pref_delivery_date, 1, null))/ count(*) 
,2) as immediate_percentage
from cte
```
Main issue: in your CTE you `GROUP BY customer_id` but also select `customer_pref_delivery_date` without aggregating it. That value becomes arbitrary per customer (or errors with `ONLY_FULL_GROUP_BY`). So you end up comparing the true first order date to a **random** preference date, not the preference from the first order row.

Fix it by locating the actual first order row per customer, then compare:

Option A — window (simple and precise):

```
WITH cte AS (
  SELECT
    customer_id,
    order_date,
    customer_pref_delivery_date,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS rn
  FROM Delivery
)
SELECT
  ROUND(100 * AVG(IF(order_date = customer_pref_delivery_date, 1, 0)), 2) AS immediate_percentage
FROM cte
WHERE rn = 1;
```

Option B — aggregate + join back (no window):

```
WITH firsts AS (
  SELECT customer_id, MIN(order_date) AS firstOrder
  FROM Delivery
  GROUP BY customer_id
)
SELECT
  ROUND(100 * AVG(IF(d.order_date = d.customer_pref_delivery_date, 1, 0)), 2) AS immediate_percentage
FROM firsts f
JOIN Delivery d
  ON d.customer_id = f.customer_id
 AND d.order_date = f.firstOrder;
```

(If ties on the same first date are possible, add a tiebreaker like `MIN(order_id)`.)



## Q6: [Game Play Analysis IV](https://leetcode.com/problems/game-play-analysis-iv/)


**Method 1:**

```sql
WITH WithFirstDay AS (
    SELECT 
        player_id,
        event_date,
        MIN(event_date) OVER (PARTITION BY player_id) AS firstDay
    FROM Activity
)
SELECT
    ROUND(
        COUNT(DISTINCT player_id) 
        / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2
    ) AS fraction
FROM WithFirstDay
-- WHERE event_date = DATE_ADD(firstDay, INTERVAL 1 DAY);

WHERE  DATEDIFF(event_date,firstDay)=1
```
**Method 1.1:**
```sql
WITH first_login AS (
    -- Get each player's first login date.
    SELECT 
        player_id, 
        MIN(event_date) AS first_date
    FROM Activity
    GROUP BY player_id
),
consecutive AS (
    -- Find players who logged in on the day after their first login.
    SELECT 
        f.player_id
    FROM first_login f
    JOIN Activity a 
      ON f.player_id = a.player_id 
     AND a.event_date = DATE_ADD(f.first_date, INTERVAL 1 DAY)
)
SELECT ROUND(
    (SELECT COUNT(*) FROM consecutive) / (SELECT COUNT(DISTINCT player_id) FROM Activity),
    2
) AS fraction;

```

**Method 1.3: Window + ROW_NUMBER()**
```sql
WITH cte AS (
  SELECT
    player_id,
    event_date,
    ROW_NUMBER() OVER (PARTITION BY player_id ORDER BY event_date) AS rn
  FROM Activity
)
SELECT
  COUNT(DISTINCT a.player_id) * 1.0
  / (SELECT COUNT(DISTINCT player_id) FROM Activity) AS fraction
FROM cte c
JOIN Activity a
  ON a.player_id = c.player_id
 AND a.event_date = DATE_ADD(c.event_date, INTERVAL 1 DAY)
WHERE c.rn = 1;

```

**Method 1.3: Aggregate + join back (simplest)** 

```sql
WITH firsts AS (
  SELECT player_id, MIN(event_date) AS first_login
  FROM Activity
  GROUP BY player_id
)
SELECT
  COUNT(*) * 1.0
  / (SELECT COUNT(DISTINCT player_id) FROM Activity) AS fraction
FROM firsts f
JOIN Activity a
  ON a.player_id = f.player_id
 AND a.event_date = DATE_ADD(f.first_login, INTERVAL 1 DAY);

```

**Method 2:**
```sql
# Write your MySQL query statement below
SELECT
  ROUND(COUNT(DISTINCT player_id) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) AS fraction
FROM
  Activity
WHERE
  (player_id, DATE_SUB(event_date, INTERVAL 1 DAY))
  IN (
    SELECT player_id, MIN(event_date) AS first_login FROM Activity GROUP BY player_id
  )

```

**Method 3:**

```sql
select ROUND((count(*) + 0.00)/ (select count(distinct player_id) from activity), 2) as fraction
from (
    select player_id, count(*) consecutive_days
    from (
        select *, 
            datediff(day, min(event_date)over(partition by player_id order by player_id, event_date asc), event_date) +1 
            -row_number()over(partition by player_id order by player_id asc) rw
        from activity
    ) c
    where rw = 0
    group by player_id
) a
where consecutive_days >= 2 
```
## Q7: [Customer Who Visited but Did Not Make Any Transactions](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/)

```sql
SELECT v.customer_id  , 

COUNT(v.visit_id ) as count_no_trans 
FROM Visits v LEFT JOIN Transactions  t on v.visit_id=t.visit_id
WHERE t.transaction_id  IS NULL

GROUP BY v.customer_id
```

## Q8: [Employees Whose Manager Left the Company](https://leetcode.com/problems/employees-whose-manager-left-the-company/)

**Method 1:**
```sql
For every employee, checks:
    - Is this employee’s manager_id not in the list of valid employee IDs (from the inner query)?
    - AND does this employee earn less than 30,000?
If both conditions are true, that employee’s ID is included in the output.

SELECT employee_id
FROM Employees
WHERE manager_id NOT IN ( -- Step1: list of all people who are present as employees (and thus could potentially be a manager).
    SELECT employee_id
    FROM Employees
    WHERE employee_id IS NOT NULL    
)
AND salary < 30000
ORDER BY employee_id ASC;

```

**Method 2:**
```sql
-- step1. Find employees whose salary is below 30K and are not managers
WITH CTE AS ( 
    SELECT *
    FROM Employees
    WHERE salary < 30000 
      AND manager_id IS NOT NULL
)
-- step2. Find employees whose salary is below 30K and whose managers' IDs are missing (i.e. managers left the company)

SELECT c.employee_id
FROM CTE c
LEFT JOIN Employees m ON c.manager_id = m.employee_id
WHERE m.employee_id IS NULL
ORDER BY c.employee_id;

```
Note: Why can't you above CTE with itself, and why a few of test cases might fail?

For example, consider this scenario:
- **Employee A** has salary <30000 and manager_id = **M**.
- **Manager M** is still in the company, but because M’s own manager_id is null, M is not included in the CTE.
- The LEFT JOIN of CTE A to CTE B on A.manager_id = B.employee_id will not find a match for M (since M is missing from the CTE), and so B.employee_id will be NULL.
- Thus, Employee A will be incorrectly selected as if their manager had left.

### Recommended Approach

To correctly check whether an employee’s manager has left, you should use the full Employees table for the manager lookup rather than a CTE that filters out managers with null manager_id. 


**Method 3:(Wrong)**
```sql

SELECT employee_id
FROM Employees
WHERE manager_id IS NULL
AND salary < 30000
ORDER BY employee_id ASC;
```
Your earlier queries were looking for:
Employees whose manager_id does not exist in the Employees table (i.e., the manager’s ID is not assigned to any current employee)
They may still have a value in manager_id—it’s just not a valid employee.
This query only returns employees who have absolutely no manager (i.e., their manager_id field is literally NULL).

Key Difference
Your earlier logic:
Finds employees with a manager ID that does not exist in the table (could be a typo, a fired manager, or a data error), and salary < 30,000.
This query:
Only finds employees at the “top of the hierarchy” (like a CEO or founder) with no manager at all, and salary < 30,000.

## Q9: [Confirmation Rate](https://leetcode.com/problems/confirmation-rate/)

**Method 1:**

```sql
select s.user_id, 
round(avg(if(c.action="confirmed",1,0)),2) as confirmation_rate

from Signups as s 
left join Confirmations as c on s.user_id= c.user_id 
group by user_id;
```

**Method 2:**

```sql
WITH CTE AS (
    SELECT
        s.user_id,
        SUM(CASE WHEN c.action = 'confirmed' THEN 1 ELSE 0 END) AS success,
        SUM(CASE WHEN c.action = 'timeout' THEN 1 ELSE 0 END) AS failure
    FROM Signups s
    LEFT JOIN Confirmations c ON s.user_id = c.user_id
    GROUP BY s.user_id
)
SELECT 
    user_id,
    ROUND(
        IF(success + failure = 0, 0, success / (success + failure)),
    2) AS confirmation_rate
FROM CTE
ORDER BY user_id;

```

**Method 3:**
```sql
SELECT
    user_id,
    ROUND(
        IF((success + failure) = 0,
           0,
           CAST(success AS DECIMAL) / CAST((success + failure) AS DECIMAL)
        ),
        2
    ) AS confirmation_rate
FROM (
    SELECT
        s.user_id,
        SUM(CASE WHEN c.action = 'confirmed' THEN 1 ELSE 0 END) AS success,
        SUM(CASE WHEN c.action = 'timeout' THEN 1 ELSE 0 END) AS failure
    FROM Signups s
    LEFT JOIN Confirmations c ON s.user_id = c.user_id
    GROUP BY s.user_id
) AS count_action;


```

**Method 4:**
```sql
WITH count_action AS (
    SELECT
        s.user_id,
        SUM(CASE WHEN c.action = 'confirmed' THEN 1 ELSE 0 END) AS success,
        SUM(CASE WHEN c.action = 'timeout' THEN 1 ELSE 0 END) AS failure
    FROM Signups s
    LEFT JOIN Confirmations c ON s.user_id = c.user_id
    GROUP BY s.user_id
)
SELECT
    user_id,
    ROUND(
        IF((success + failure) = 0, 0, CAST(success AS DECIMAL) / CAST((success + failure) AS DECIMAL)),
        2
    ) AS confirmation_rate
FROM count_action;


```

**Method 5: Using Full OUTER JOIN (Although not required; redundant)**
```sql
SELECT 
    user_id,
    ROUND(AVG(IF(action = 'confirmed', 1, 0)), 2) AS confirmation_rate
FROM (
    -- All rows from Signups (with matching confirmations if any)
    SELECT s.user_id, c.action
    FROM Signups s
    LEFT JOIN Confirmations c ON s.user_id = c.user_id

    UNION ALL

    -- Rows from Confirmations that have no matching signup
    SELECT c.user_id, c.action
   FROM Signups s
   RIGHT JOIN Confirmations c ON s.user_id = c.user_id
    WHERE s.user_id IS NULL
) t
GROUP BY user_id;

```

## Q10:[User Activity for the Past 30 Days I](https://leetcode.com/problems/user-activity-for-the-past-30-days-i/)

**Method 1:**
```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE (activity_date > "2019-06-27" AND activity_date <= "2019-07-27")
GROUP BY activity_date;
```

**Note: Suppose specific activity types were the criteria to consider as an active user then-**
```sql
WITH CTE AS (
    SELECT
        activity_date as "day",
        CASE 
            WHEN activity_type IN ('scroll_down', 'send_message') THEN user_id
        END AS activeusers
    FROM Activity
    WHERE (activity_date > "2019-06-27" AND activity_date <= "2019-07-27")

)

SELECT activity_date, COUNT(DISTINCT activeusers) AS active_users
FROM CTE 
GROUP BY activity_date;
```




## Q11: [Monthly Transactions I](https://leetcode.com/problems/monthly-transactions-i/)

**Method 1:**
```sql
SELECT
    DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(id) AS trans_count,
    SUM(IF(state='approved', 1, 0)) AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(IF(state='approved', amount, 0)) AS approved_total_amount
FROM     Transactions A
GROUP BY  month, country;
```

**Method 2:**

```sql
SELECT  
SUBSTR(trans_date,1,7) as month, 
country, count(id) as trans_count, 
SUM(CASE WHEN state = 'approved' then 1 else 0 END) as approved_count, 
SUM(amount) as trans_total_amount, 
SUM(CASE WHEN state = 'approved' then amount else 0 END) as approved_total_amount

FROM Transactions
GROUP BY month, country
```
## Q12: [Triangle Judgement](https://leetcode.com/problems/triangle-judgement/)

```sql
Select
x,y,z,
IF(x+y >z AND x+z >y AND y+z>x, "Yes", "No") as triangle

from Triangle
```

## Q12:[Last Person to Fit in the Bus](https://leetcode.com/problems/last-person-to-fit-in-the-bus/)

** Method 1:**
```sql
WITH CTE AS (
    SELECT *,
           SUM(weight) OVER (ORDER BY turn ASC) AS `Total Weight`
    FROM Queue 
)

SELECT person_name 
FROM CTE
WHERE `Total Weight` <= 1000
ORDER BY `Total Weight` DESC
LIMIT 1

```
**Method 2:**
```sql
SELECT person_name
FROM (
    SELECT
        person_name,
        turn,
        SUM(weight) OVER (ORDER BY turn ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS total_weight
    FROM Queue
) AS ranked
WHERE total_weight <= 1000
ORDER BY turn DESC
LIMIT 1;
```
**Method 3:**
```sql

WITH CTE AS (
    SELECT 
        turn, person_name, weight,
        SUM(weight) OVER(ORDER BY turn ASC) AS tot_weight 
    FROM Queue
    ORDER BY turn
)
SELECT person_name
FROM Queue q
WHERE q.turn = (SELECT MAX(turn) FROM CTE WHERE tot_weight <= 1000);
```
**Method 4:**
```sql
SELECT 
    q1.person_name
FROM Queue q1 
JOIN Queue q2 ON q1.turn >= q2.turn
GROUP BY q1.turn
HAVING SUM(q2.weight) <= 1000
ORDER BY SUM(q2.weight) DESC
LIMIT 1
```
## Q13: [Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)

```sql
delete p1 from person p1,person p2 
where p1.email=p2.email and p1.id>=p2.id;
```

## Q13.1: Why p1.id<>=p2.id wouldn't work in above query?

For below input data-

```plaintext
| id | email            |
|----|------------------|
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
```
Using the query:

```sql
SELECT 
    P1.id AS P1_id,
    P1.email AS P1_email,
    P2.id AS P2_id,
    P2.email AS P2_email
FROM Person P1, Person P2
WHERE P1.email = P2.email 
  AND P1.id <> P2.id;
```
```plaintext
The intermediate result would be:

| P1_id | P1_email         | P2_id | P2_email         |
|-------|------------------|-------|------------------|
| 1     | john@example.com | 3     | john@example.com |
| 3     | john@example.com | 1     | john@example.com |
```

**Explanation:**

- For `john@example.com`, there are two rows (id = 1 and id = 3).  
- The join condition `P1.email = P2.email` finds that these two rows share the same email.  
- The condition `P1.id <> P2.id` ensures that a row isn't paired with itself.  
- As a result, you get both combinations: one pairing where `P1` is the row with id = 1 and `P2` is the row with id = 3, and another pairing where `P1` is the row with id = 3 and `P2` is the row with id = 1.

When you use `P1.id > P2.id`, only one of these pairings is returned (the one with the higher id as P1), which prevents deleting both copies. Without that condition, the intermediate join produces duplicate pairs, and if used in a DELETE statement, it may lead to unintended results.

## Q14: [List the Products Ordered in a Period](https://leetcode.com/problems/list-the-products-ordered-in-a-period/)

```sql

    SELECT 
        A.product_name,
        SUM(B.unit) as unit
    FROM Products A
    INNER JOIN Orders B ON A.product_id = B.product_id
    WHERE DATE_FORMAT(B.order_date, '%M %Y') = 'February 2020'
    group by A.product_name 
    HAVING unit>=100

```
## Q15: [Customers Who Bought All Products](https://leetcode.com/problems/customers-who-bought-all-products/)

```sql

SELECT  customer_id 
FROM Customer 
GROUP BY customer_id
HAVING COUNT(distinct product_key) = (SELECT COUNT(product_key) FROM Product)
```
## Q16: [Group Sold Products By The Date](https://leetcode.com/problems/group-sold-products-by-the-date/)

```sql

SELECT 
    sell_date,
    COUNT(DISTINCT product) as num_sold,
    GROUP_CONCAT(DISTINCT product ORDER BY product) as products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```
## Q17: [Count Salary Categories](https://leetcode.com/problems/count-salary-categories/)


**Method 1:**

```sql
WITH flat_category AS (
    SELECT 
        SUM(CASE WHEN income < 20000 THEN 1 ELSE 0 END) AS `Low Salary`, 
        SUM(CASE WHEN income >= 20000 AND income <= 50000 THEN 1 ELSE 0 END) AS `Average Salary`,
        SUM(CASE WHEN income > 50000 THEN 1 ELSE 0 END) AS `High Salary`
    FROM Accounts
)

SELECT 'Low Salary' AS category, `Low Salary` AS accounts_count FROM flat_category
UNION ALL
SELECT 'Average Salary', `Average Salary` FROM flat_category
UNION ALL
SELECT 'High Salary', `High Salary` FROM flat_category;
```
**Method 2:**

```sql
SELECT 'High Salary'   AS category, SUM(income > 50000)  AS accounts_count FROM Accounts
UNION ALL
SELECT 'Low Salary' , SUM(income < 20000)  AS accounts_count FROM Accounts
UNION ALL
SELECT 'Average Salary' , SUM(income BETWEEN 20000 AND 50000) AS accounts_count FROM Accounts;

```

## Q18: [Product Price at a Given Date](https://leetcode.com/problems/product-price-at-a-given-date/)


**Method 1**
```sql
# Write your MySQL query statement below

SELECT 
product_id, 
new_price AS price 

FROM Products 
WHERE (product_id, change_date) IN (
                                    SELECT 
                                    product_id, max(change_date) 
                                    FROM Products 
                                    WHERE change_date <= '2019-08-16' 
                                    GROUP BY product_id)

UNION


SELECT product_id, 10 AS price 
FROM Products 
WHERE product_id NOT IN (SELECT product_id 
                        FROM Products 
                        WHERE change_date <= '2019-08-16');

```
**Note:**
- First part: For each product, fetch its most recent price on or before '2019-08-16'.
- Second part: For products without any price by that date, return 10.

**Method 2**
```sql
# Part 1 — products with no change ≤ date → price 10

select 
distinct product_id, 10 as price 
from Products 
where product_id not in(
                        select distinct product_id 
                        from Products 
                        where change_date <='2019-08-16' )
union

# Part 2 — products with at least one change ≤ date → take the last one 
select 
product_id, new_price as price 
from Products 
where (product_id,change_date) in (
                        select product_id , max(change_date) as date 
                        from Products 
                        where change_date <='2019-08-16' 
                        group by product_id)
```
## Q19: [User Activity for the Past 30 Days I](https://leetcode.com/problems/user-activity-for-the-past-30-days-i/)

```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE (activity_date > "2019-06-27" AND activity_date <= "2019-07-27")
GROUP BY activity_date;
```
## Q20: [Article Views I](https://leetcode.com/problems/article-views-i/)

```sql
select  distinct author_id as id 
from Views 
where author_id = viewer_id 
order by id  asc;
```
## Q21: [Average Selling Price](https://leetcode.com/problems/average-selling-price/)

```sql
# Write your MySQL query statement below
SELECT p.product_id, 
IFNULL(ROUND(SUM(units*price)/SUM(units),2),0) AS average_price

FROM Prices p 
LEFT JOIN UnitsSold u ON p.product_id = u.product_id AND u.purchase_date BETWEEN start_date AND end_date

group by product_id
```

## Q 21.a : Why is the below query wrong for the above problem but the above query is correct?

The **first query** is more correct. 

### **Explanation:**

- **First Query:**  
  ```sql
  SELECT p.product_id, 
         IFNULL(ROUND(SUM(u.units * p.price) / SUM(u.units), 2), 0) AS average_price
  FROM Prices p 
  LEFT JOIN UnitsSold u 
    ON p.product_id = u.product_id 
   AND u.purchase_date BETWEEN p.start_date AND p.end_date
  GROUP BY p.product_id;
  ```  
  - The condition `u.purchase_date BETWEEN p.start_date AND p.end_date` is placed in the `ON` clause.  
  - This ensures that **all products from the Prices table are included** (even those with no sales) because the `LEFT JOIN` is preserved.
  - For products with no matching sales (i.e., no rows in `UnitsSold`), `u.units` will be `NULL`, and `IFNULL(..., 0)` ensures the `average_price` is set to `0`.

- **Second Query:**  
  ```sql
  SELECT p.product_id, 
         IFNULL(ROUND(SUM(u.units * p.price) / SUM(u.units), 2), 0) AS average_price
  FROM Prices p 
  LEFT JOIN UnitsSold u 
    ON p.product_id = u.product_id 
  WHERE u.purchase_date BETWEEN p.start_date AND p.end_date
  GROUP BY p.product_id;
  ```  
  - The date condition is placed in the `WHERE` clause, which **filters out rows where `u.purchase_date` is `NULL`**.
  - This effectively turns the `LEFT JOIN` into an `INNER JOIN`, thereby **excluding products with no sales** from the result.

Since the problem requires that products with no sold units should return an average selling price of 0, the **first query** is the correct approach.


## Q22: [Percentage of Users Attended a Contest](https://leetcode.com/problems/percentage-of-users-attended-a-contest/)

```sql
select 
contest_id, 
round(count(distinct user_id) * 100 /(select count(user_id) from Users) ,2) as percentage
from  Register
group by contest_id
order by percentage desc,contest_id
```
## Q23: [Invalid Tweets](https://leetcode.com/problems/invalid-tweets/)

```sql
Select tweet_id
from Tweets
where length(content) >15
```
## Q24: [Find Followers Count](https://leetcode.com/problems/find-followers-count/)

```sql
select user_id , count(distinct follower_id) as followers_count
from followers
group by user_id
order by user_id asc , followers_count asc;
```
## Q25: [Employee Bonus](https://leetcode.com/problems/employee-bonus/)

```sql
Select A.name, B.bonus
from Employee A
LEFT JOIN Bonus B on A.empId=B.empId
where B.Bonus <1000 or B.Bonus is NULL
```
## Q26: [Queries Quality and Percentage](https://leetcode.com/problems/queries-quality-and-percentage/)
This is a classic use-case for conditional aggregation.
You need to calculate the percentage per query_name, i.e.,
(number of poor queries for that name) / (total queries for that name).

You want the percentage of poor queries (rating < 3) for each query_name, not for the entire table.

**Method 1:**
```sql
SELECT
    query_name,
    ROUND(AVG(rating / position),2) AS quality,
    ROUND(COUNT(IF(rating < 3, 1, null)) * 100.0 / COUNT(*),2) AS poor_query_percentage
FROM Queries
GROUP BY query_name;
```

That’s a SQL trick to make `COUNT()` count **only the matching rows**.

### How it works

* `COUNT(expr)` in SQL **ignores** `NULL` values.
* If you write:

```sql
COUNT(IF(rating < 3, 1, NULL))
```

it means:

* When `rating < 3` → return `1` (not `NULL`) → **counts it**
* When `rating >= 3` → return `NULL` → **ignored by COUNT()**

### Why not `COUNT(IF(..., 0))`?

If you use `0` instead of `NULL`:

```sql
COUNT(IF(rating < 3, 1, 0))
```

then:

* When `rating >= 3`, `0` is still **not NULL**, so it gets counted too.
* That means **all rows** would be counted, not just the poor ones.


## Similar example: https://leetcode.com/problems/monthly-transactions-i/description/?envType=study-plan-v2&envId=top-sql-50

**Method 1.1**
```sql
SELECT query_name, 
ROUND(AVG(rating/position),2) AS quality, 
ROUND(AVG(IF(rating < 3, 1, 0))*100,2) 
AS poor_query_percentage 
FROM Queries 
WHERE query_name IS NOT NULL 
GROUP BY query_name;
```
**Method 2**

```sql
select
query_name,
round(avg(cast(rating as decimal) / position), 2) as quality,
round(sum(case when rating < 3 then 1 else 0 end) * 100 / count(*), 2) as poor_query_percentage
from queries
WHERE query_name IS NOT NULL
group by query_name
```


## Q27: [Restaurant Growth](https://leetcode.com/problems/restaurant-growth/)

**Method 1:**
```sql
WITH CTE AS (
select visited_on , sum(amount) as amount
from customer
group by visited_on


)
select 
visited_on, 
sum(amount) OVER(ROWS BETWEEN 6 PRECEDING  and CURRENT row)as amount,
ROUND((
avg(amount) OVER(ROWS BETWEEN 6 PRECEDING  and CURRENT row) 
),2)
as average_amount
from cte
order by visited_on
limit 1213131212 offset 6

```

**Method2:**
```sql
WITH temp AS (
    SELECT visited_on, SUM(amount) AS amount
    FROM Customer
    GROUP BY visited_on
)

SELECT 
    visited_on, 
    SUM(amount) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS amount, 
    ROUND(AVG(amount*1.00) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS average_amount
FROM temp
ORDER BY visited_on
LIMIT 18446744073709551615 OFFSET 6;
```
## Q27.1: What would happen if you just use outer query something like below?



```plaintext
| visited_on | amount | average_amount |
|------------|--------|----------------|
| 2019-01-07 | 860    | 122.86         |
| 2019-01-08 | 840    | 120            |
| 2019-01-09 | 840    | 120            |
| 2019-01-10 | 850    | 121.43         |
```
Let's compare the two approaches step by step to see why the wrong query produces different (and incorrect) results:

---

### **Wrong Query:**

```sql
SELECT 
    visited_on,
    ROUND(SUM(amount) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS amount,
    ROUND(AVG(amount) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS average_amount
FROM Customer
GROUP BY visited_on
LIMIT 18446744073709551615 OFFSET 6;
```

**What Happens Internally:**

1. **GROUP BY visited_on:**
   - The query groups raw Customer rows by each day.
   - For each day, an aggregated value is computed.  
   - **However**, because window functions are used _in the same query_ that performs the grouping, MySQL applies the window functions on the grouped result.  
   - If the grouping isn’t isolated properly, MySQL might not compute the aggregated daily totals as expected.

2. **Window Functions on Grouped Results:**
   - The window functions use the column `amount` (which ideally should be the daily total) to compute a 7-day rolling sum and average.
   - In our wrong query, the daily total for **2019-01-10** comes out as **850** rather than the correct **1000**.  
   - Consequently, the moving average computed over the window becomes **121.43**.

**Wrong Query Output:**

| visited_on | amount | average_amount |
| ---------- | ------ | -------------- |
| 2019-01-07 | 860    | 122.86         |
| 2019-01-08 | 840    | 120            |
| 2019-01-09 | 840    | 120            |
| 2019-01-10 | 850    | 121.43         |

*Note:* The daily total on 2019-01-10 is wrong (850 instead of 1000) because the grouping and window calculations are mixed in one query without isolating the aggregation step. This in turn affects the moving average.

---

### **Right Query:**

```sql
WITH temp AS (
    SELECT visited_on, SUM(amount) AS amount
    FROM Customer
    GROUP BY visited_on
)
SELECT 
    visited_on, 
    SUM(amount) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS amount, 
    ROUND(AVG(amount * 1.00) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) AS average_amount
FROM temp
ORDER BY visited_on
LIMIT 18446744073709551615 OFFSET 6;
```

**What Happens Internally:**

1. **CTE (temp) – Pre-Aggregation:**
   - The CTE (`temp`) first groups the Customer table by `visited_on` and calculates the **correct daily total** for `amount`.
   - Now, each day appears exactly once with its proper aggregated total.
   - For example, on **2019-01-10**, the correct daily total is computed as **1000**.

2. **Window Functions on Pre-Aggregated Data:**
   - In the outer query, the window functions are applied to the pre-aggregated daily totals.
   - The 7-day rolling sum and average are now computed over the correct daily totals.
   - Thus, for 2019-01-10, the moving average is correctly computed as **142.86**.

**Right (Expected) Output:**

| visited_on | amount | average_amount |
| ---------- | ------ | -------------- |
| 2019-01-07 | 860    | 122.86         |
| 2019-01-08 | 840    | 120            |
| 2019-01-09 | 840    | 120            |
| 2019-01-10 | 1000   | 142.86         |

---

### **Key Differences:**

- **Isolation of Aggregation:**
  - **Wrong Query:** Combines grouping and window functions in one step, which leads to an incorrect daily total (850 for 2019-01-10) because the window functions are applied on an intermediate result that hasn’t been cleanly aggregated.
  - **Right Query:** Uses a CTE to first pre-aggregate daily totals. This ensures that each day's `amount` is correct (1000 for 2019-01-10).

- **Effect on Moving Average:**
  - Since the moving average is computed using the daily total from each row, an incorrect daily total (850) leads to an incorrect moving average (121.43) in the wrong query.
  - The correct daily total (1000) in the right query yields the expected moving average (142.86).

---

### **Conclusion:**

The right query separates the daily aggregation (summing amounts per visited_on) from the moving window calculation. This isolation ensures that each day’s aggregated amount is accurate, which in turn produces the correct rolling average. In contrast, mixing the GROUP BY and window functions in one query (as in the wrong query) can lead to miscalculations in the daily totals and, consequently, the moving average.

This analysis explains why the wrong query returns 850 and 121.43 for 2019-01-10, while the right query (and expected output) returns 1000 and 142.86.


## Q28: [Average Selling Price](https://leetcode.com/problems/average-selling-price/)

```sql
SELECT p.product_id, 
IFNULL(ROUND(SUM(units*price)/SUM(units),2),0) AS average_price

FROM Prices p 
LEFT JOIN UnitsSold u ON p.product_id = u.product_id AND u.purchase_date BETWEEN start_date AND end_date

group by product_id
```

## Q29:  [Monthly Transactions I](https://leetcode.com/problems/monthly-transactions-i/)


**Method 1:**
```sql
SELECT
    DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(id) AS trans_count,
    SUM(IF(state='approved', 1, 0)) AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(IF(state='approved', amount, 0)) AS approved_total_amount
FROM     Transactions A
GROUP BY  month, country;
```
**Method 2:**
```sql
SELECT  

SUBSTR(trans_date,1,7) as month, 
country, count(id) as trans_count, 
SUM(CASE WHEN state = 'approved' then 1 else 0 END) as approved_count, 
SUM(amount) as trans_total_amount, 
SUM(CASE WHEN state = 'approved' then amount else 0 END) as approved_total_amount

FROM Transactions
GROUP BY month, country
```
## Q30: [Product Sales Analysis III](https://leetcode.com/problems/product-sales-analysis-iii/)

**Method 1:**

```sql
SELECT C.product_id, C.year as first_year, C.quantity, C.price
FROM (
    SELECT A.year, B.product_id, A.quantity, A.price,
           DENSE_RANK() OVER (PARTITION BY B.product_id ORDER BY A.year ASC) AS Ranking
    FROM Sales A
    INNER JOIN Product B ON A.product_id = B.product_id
) AS C
WHERE C.Ranking = 1;
```

**Method 2:**
```sql
WITH CTE AS (    
    SELECT A.product_id, A.year, A.quantity, A.price,
           DENSE_RANK() OVER (PARTITION BY B.product_id ORDER BY A.year ASC) AS Ranking
    FROM Sales A
    INNER JOIN Product B ON A.product_id = B.product_id

)

SELECT product_id, year as first_year, quantity, price
FROM CTE
WHERE Ranking =1
ORDER BY 4
```

## Q31: [Rising Temperature](https://leetcode.com/problems/rising-temperature/)

**Method 1**
```sql
SELECT w1.id
FROM Weather w1, Weather w2
WHERE w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
AND w1.temperature > w2.temperature;
```
```plaintext
# Output with all columns:
 | id | recordDate | temperature | id | recordDate | temperature |
 | -- | ---------- | ----------- | -- | ---------- | ----------- |
 | 2  | 2015-01-02 | 25          | 1  | 2015-01-01 | 10          |
 | 4  | 2015-01-04 | 30          | 3  | 2015-01-03 | 20          |

```
**Method 2**
```sql
SELECT w1.id
FROM Weather w1, Weather w2
WHERE DATEDIFF(w1.recordDate, w2.recordDate) = 1
 AND w1.temperature > w2.temperature;
```

** Method 3 **
```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2 ON w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
WHERE w1.temperature > w2.temperature
```
```plaintext
# Output with all columns:
| id | recordDate | temperature | id | recordDate | temperature |
| -- | ---------- | ----------- | -- | ---------- | ----------- |
| 2  | 2015-01-02 | 25          | 1  | 2015-01-01 | 10          |
| 4  | 2015-01-04 | 30          | 3  | 2015-01-03 | 20          |
```

## Q 32: [Game Play Analysis IV](https://leetcode.com/problems/game-play-analysis-iv/)


**Method 1:**
```sql
WITH WithFirstDay AS (
    SELECT 
        player_id,
        event_date,
        MIN(event_date) OVER (PARTITION BY player_id) AS firstDay
    FROM Activity
)
SELECT
    ROUND(
        COUNT(DISTINCT player_id) 
        / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2
    ) AS fraction
FROM WithFirstDay
-- WHERE event_date = DATE_ADD(firstDay, INTERVAL 1 DAY);

WHERE  DATEDIFF(event_date,firstDay)=1
```

```sql
# Write your MySQL query statement below
SELECT
  ROUND(COUNT(DISTINCT player_id) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) AS fraction
FROM
  Activity
WHERE
  (player_id, DATE_SUB(event_date, INTERVAL 1 DAY))
  IN (
    SELECT player_id, MIN(event_date) AS first_login FROM Activity GROUP BY player_id
  )
```

** Method 2** Review below query for error once again

```sql
select ROUND((count(*) + 0.00)/ (select count(distinct player_id) from activity), 2) as fraction
from (
    select player_id, count(*) consecutive_days
    from (
        select *, 
            datediff(day, min(event_date)over(partition by player_id order by player_id, event_date asc), event_date) +1 
            -row_number()over(partition by player_id order by player_id asc) rw
        from activity
    ) c
    where rw = 0
    group by player_id
) a
where consecutive_days >= 2 
```

## Q33: [Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)

** Method 1**
```sql
with cte as (
 select num,
    lead(num,1) over() num1,
    lead(num,2) over() num2
    from logs
)

select distinct num ConsecutiveNums 
from 
cte where 
(num=num1) and (num=num2)
```

** Method 2**
```sql
 Select 
 distinct B.num ConsecutiveNums 

 from(
    select num,
    lead(num,1) over() num1,
    lead(num,2) over() num2
    from logs A
 )  B
 where (B.num=B.num1) and (B.num=B.num2)
```
## Q34: [Exchange Seats](https://leetcode.com/problems/exchange-seats/)

** Method 1**
```sql
SELECT
    IF(id % 2 = 1 AND id < (SELECT MAX(id) FROM Seat), id + 1, IF(id % 2 = 0, id - 1, id)) AS id,
    student
FROM Seat
ORDER BY id;
```

** Method 2**
```sql
SELECT
    CASE
        WHEN
            id = (SELECT MAX(id) FROM Seat) AND MOD(id, 2) = 1 
            THEN id
        WHEN
            MOD(id, 2) = 1
            THEN id + 1
        ELSE
            id - 1
    END AS id, student 
FROM Seat 
ORDER BY id;
```
##Revise below ones again.

## Q35: [Students and Examinations](https://leetcode.com/problems/students-and-examinations/)

```sql
# Write your MySQL query statement below
SELECT s.student_id, s.student_name, sub.subject_name, COUNT(e.student_id) AS attended_exams
FROM Students s

CROSS JOIN Subjects sub
LEFT JOIN Examinations e ON s.student_id = e.student_id AND sub.subject_name = e.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```
## Q36: [Average Time of Process per Machine](https://leetcode.com/problems/average-time-of-process-per-machine/)

** Method 1**

```sql
select a1.machine_id, round(avg(a2.timestamp-a1.timestamp), 3) as processing_time 
from Activity a1
join Activity a2 
on a1.machine_id=a2.machine_id and a1.process_id=a2.process_id
and a1.activity_type='start' and a2.activity_type='end'
group by a1.machine_id;
```

** Method 1.1: **

```sql
WITH cte AS (
  SELECT
    machine_id,
    process_id,
    SUM(CASE WHEN activity_type = 'end'  THEN `timestamp`
             ELSE -`timestamp` END) AS time_taken
  FROM Activity
  GROUP BY machine_id, process_id
)
SELECT
  a.machine_id,
  ROUND(AVG(a.time_taken), 3) AS processing_time
FROM cte a
GROUP BY a.machine_id
ORDER BY a.machine_id ASC, processing_time DESC;

```

** Method 1.2: **

```sql
WITH cte AS (
  SELECT
    machine_id,
    process_id,
    MAX(CASE WHEN activity_type = 'end'   THEN `timestamp` END) -
    MAX(CASE WHEN activity_type = 'start' THEN `timestamp` END) AS time_taken
  FROM Activity
  GROUP BY machine_id, process_id
)
SELECT
  machine_id,
  ROUND(AVG(time_taken), 3) AS processing_time
FROM cte
GROUP BY machine_id
ORDER BY machine_id ASC, processing_time DESC;

```


** Method 2**
```sql
select 
A.machine_id,
round(
      (select avg(a1.timestamp) from Activity A where A.activity_type = 'end' and A.machine_id = B.machine_id) - 
      (select avg(a1.timestamp) from Activity A where A.activity_type = 'start' and A.machine_id = B.machine_id)
,3) as processing_time
from Activity B
group by B.machine_id
```
## Q37: [Product Sales Analysis III](https://leetcode.com/problems/product-sales-analysis-iii/)

```sql
SELECT C.product_id, C.year as first_year, C.quantity, C.price
FROM (
    SELECT A.year, B.product_id, A.quantity, A.price,
           DENSE_RANK() OVER (PARTITION BY B.product_id ORDER BY A.year ASC) AS Ranking
    FROM Sales A
    INNER JOIN Product B ON A.product_id = B.product_id
) AS C
WHERE C.Ranking = 1;
```


## Q38: [Friend Requests ||: Who has the most friends] (https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/?envType=study-plan-v2&envId=top-sql-50)
Answer:

**Method 1:**

```sql

#Method 1: Easiest solution

#Step 1 – Combine requester and accepter into one column

with base as(
    select requester_id id 
    from RequestAccepted
union all
select accepter_id id 
from RequestAccepted)


# Step 2 – Count friendships per user

select id, count(*) num  
from base 
group by 1 
order by 2 desc 
limit 1

```

**Method 2:**

```sql
-- #Method 2: Another solution

select requester_id as id,
       (select count(*) from RequestAccepted
            where id=requester_id or id=accepter_id) as num
from RequestAccepted
group by requester_id
order by num desc 
limit 1

```

**Note: why do we need to union all them to get answer to that question?**
We need the `UNION ALL` because in the **friendship counting problem** both `requester_id` and `accepter_id` represent the *same thing* in the end — a user who gained a friend.

If we only counted `requester_id`, we’d be ignoring half the friendships (the people who accepted).
If we only counted `accepter_id`, we’d miss the people who initiated requests.

---

### **Why not just count directly in one query?**

Let’s take the sample:

| requester\_id | accepter\_id |
| ------------- | ------------ |
| 1             | 2            |
| 2             | 3            |
| 1             | 3            |

---

#### Without `UNION ALL` — counting only requesters:

```
id | num
1  | 2
2  | 1
```

(We completely missed counting `3` who accepted two requests.)

---

#### Without `UNION ALL` — counting only accepters:

```
id | num
2  | 1
3  | 2
```

(Now we missed `1` who sent two requests.)

---

### **With `UNION ALL`**

We flatten both columns into one:

```
1
2
1
2
3
3
```

Then counting:

```
id | num
1  | 2
2  | 2
3  | 2
```

Now **every friendship is counted for both sides**.

---

### **Why not `UNION` (without ALL)?**

`UNION` removes duplicates *between* the two lists before stacking, which would mess up the counts because if the same user appears in both requester and accepter columns for different friendships, some rows would be wrongly eliminated.

That’s why **`UNION ALL` is required** — duplicates represent real separate friendships.


## Q39: [Investments in 2016](https://leetcode.com/problems/investments-in-2016/?envType=study-plan-v2&envId=top-sql-50)

**Method 1:**
```sql
WITH cte AS (
  SELECT *,
         COUNT(*) OVER (PARTITION BY tiv_2015) AS c2015,
         COUNT(*) OVER (PARTITION BY lat, lon) AS ccity
  FROM Insurance
)
SELECT ROUND(SUM(tiv_2016), 2) AS tiv_2016
FROM cte
WHERE c2015 > 1 AND ccity = 1;

```


**Method 2:**
```sql
SELECT ROUND(SUM(tiv_2016), 2) AS tiv_2016
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
)
```



## **First Solution — Using `GROUP BY` Subqueries**

```sql
SELECT ROUND(SUM(tiv_2016), 2) AS tiv_2016
FROM Insurance i
WHERE i.tiv_2015 IN (
  SELECT tiv_2015
  FROM Insurance
  GROUP BY tiv_2015
  HAVING COUNT(*) > 1
)
AND (i.lat, i.lon) IN (
  SELECT lat, lon
  FROM Insurance
  GROUP BY lat, lon
  HAVING COUNT(*) = 1
);
```

### **Logic**

1. **Requirement 1:**
   *Policyholder must have the same `tiv_2015` as at least one other policyholder.*

   * This is achieved by:

     ```sql
     SELECT tiv_2015
     FROM Insurance
     GROUP BY tiv_2015
     HAVING COUNT(*) > 1
     ```

     This returns `tiv_2015` values that appear more than once in the table.

2. **Requirement 2:**
   *Policyholder’s `(lat, lon)` must be unique.*

   * This is done by:

     ```sql
     SELECT lat, lon
     FROM Insurance
     GROUP BY lat, lon
     HAVING COUNT(*) = 1
     ```

     This returns coordinates that appear only once.

3. **Main Query Filtering:**

   * We keep only those rows where:

     * Their `tiv_2015` is in the first subquery result (**has duplicates**).
     * Their `(lat, lon)` is in the second subquery result (**unique location**).

4. **Final Step:**

   * We `SUM(tiv_2016)` for those rows and `ROUND` it to 2 decimal places.

---

## **Second Solution — Using Window Functions (MySQL 8+ / PostgreSQL)**

```sql
WITH t AS (
  SELECT *,
         COUNT(*) OVER (PARTITION BY tiv_2015) AS c2015,
         COUNT(*) OVER (PARTITION BY lat, lon) AS ccity
  FROM Insurance
)
SELECT ROUND(SUM(tiv_2016), 2) AS tiv_2016
FROM t
WHERE c2015 > 1 AND ccity = 1;
```

### **Logic**

1. **COUNT(\*) OVER (PARTITION BY tiv\_2015)**:

   * For each row, counts how many rows have the same `tiv_2015`.
   * This avoids the need for a separate `GROUP BY` subquery.
   * `c2015 > 1` means "there is at least one other row with same `tiv_2015`".

2. **COUNT(\*) OVER (PARTITION BY lat, lon)**:

   * For each row, counts how many rows share the same `(lat, lon)` coordinates.
   * `ccity = 1` means "location is unique".

3. **WHERE condition**:

   * Keep only rows that meet both conditions.

4. **SUM & ROUND**:

   * Sum up `tiv_2016` for those rows and round to 2 decimals.

---

### **Key Differences**

* **First solution:**

  * More compatible with older SQL versions (works in MySQL < 8.0, PostgreSQL, etc.).
  * Uses subqueries with `GROUP BY`.
* **Second solution:**

  * Requires MySQL 8+ or a database that supports **window functions**.
  * Often faster since it avoids multiple scans of the table.



