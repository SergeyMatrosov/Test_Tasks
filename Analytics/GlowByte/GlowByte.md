```sql
WITH
  UserTable AS (
  SELECT "abc001" AS userid, "2020-05-01" AS date, 0 AS balance UNION ALL
  SELECT "abc001", "2020-05-02", 100 UNION ALL
  SELECT "abc001", "2020-05-03", 50 UNION ALL
  SELECT "abc001", "2020-05-04", 30 UNION ALL
  SELECT "abc001", "2020-05-05", 0 UNION ALL
  SELECT "abc002", "2020-05-01", 100 UNION ALL
  SELECT "abc002", "2020-05-02", 50 UNION ALL
  SELECT "abc002", "2020-05-03", 0 UNION ALL
  SELECT "abc002", "2020-05-04", 0 UNION ALL
  SELECT "abc002", "2020-05-05", 200),
  
  Ranges AS
    (SELECT
        userid,
        LAST_VALUE(date_from IGNORE NULLS) OVER (PARTITION BY userid ORDER BY date RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS date_from,
        LAST_VALUE(date_to IGNORE NULLS) OVER (PARTITION BY userid ORDER BY date DESC RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS date_to,
        balance
    FROM
        (SELECT 
            userid,
            date,
            IF(LAG(diff) OVER (PARTITION BY userid ORDER BY date) IS NULL OR LAG(diff) OVER (PARTITION BY userid ORDER BY date) > 1, date, NULL) AS date_from,
            IF(diff > 1 OR LEAD(diff) OVER (PARTITION BY userid ORDER BY date) IS NULL, date, NULL) AS date_to,
            balance 
        FROM
            (SELECT 
                userid,
                date,
                IF(LEAD(date) OVER (PARTITION BY userid ORDER BY date) IS NULL, date, LEAD(date) OVER (PARTITION BY userid ORDER BY date)) AS next_date,
                IF(DATETIME_DIFF(TIMESTAMP(LEAD(date) OVER (PARTITION BY userid ORDER BY date)), TIMESTAMP(date), DAY) IS NULL, 1, DATETIME_DIFF(TIMESTAMP(LEAD(date) OVER (PARTITION BY userid ORDER BY date)), TIMESTAMP(date), DAY)) AS diff,
                balance
            FROM UserTable
            WHERE balance != 0))
     ORDER BY userid, date_from)

SELECT 
    DISTINCT 
    userid,
    date_from,
    date_to,
    DATETIME_DIFF(TIMESTAMP(date_to), TIMESTAMP(date_from), DAY)+1 AS length,
    AVG(balance) OVER (PARTITION BY userid, date_from, date_to) AS balance_avg
FROM Ranges
```
