```sql
--n - размер выборки
--ИЗМЕНИ n, если нужен другой размер выборки
DECLARE n INT64 DEFAULT 10;

--ЭТО НЕ ТРОГАЙ (нужно для цикла)
DECLARE i INT64 DEFAULT 0;

--сюда будем записывать
CREATE TEMP TABLE task_2 (sample STRING);

--отсюда будем брать рандомно выборки
CREATE TEMP TABLE temptable AS (
  WITH base AS (
  SELECT "1" AS item_id, 1 AS value
  UNION ALL
  SELECT "2", 2
  UNION ALL
  SELECT "3", 6
  ),
  --подготовка данных
  --считаем вероятность в зависимости от value и суммы 
  prob_t1 AS 
  (SELECT
    item_id,
    value,
    value/SUM(value) OVER () AS prob
  FROM base),
  --вводим рэнджи долей (от - до)
  prob_t2 AS (
  SELECT
    item_id,
    IF(prob = MIN(prob) OVER (), 0, LAG(running_sum) OVER (ORDER BY running_sum)) AS from_,
    IF(prob = MAX(prob) OVER (), 1, running_sum) AS _to
  FROM
    (SELECT
      item_id,
      prob,
      SUM(prob) OVER (ORDER BY prob ASC) AS running_sum
    FROM prob_t1)
  )

  SELECT
    *
  FROM prob_t2
);

--сэмплирование
LOOP
  SET i = i + 1;
  IF i > n THEN
    LEAVE;
  ELSE
  INSERT INTO task_2 (sample)
  SELECT
    item_id
  FROM
    (SELECT 
      item_id,
      from_,
      _to
    FROM temptable)
   --да-да, тяжелая операция - но нам нужно одно случайное значение для всех!  
    CROSS JOIN
    (SELECT
    --генератор случайный значений от 0 до 1
      RAND() AS random) AS t2
  --непрерывнй рендж от 0 до 1 включительно -> 0.1123213, 0.435535 и т.дю
  WHERE random BETWEEN from_ AND _to;
  END IF;
END LOOP;
```
