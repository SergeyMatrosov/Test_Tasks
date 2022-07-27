```sql
--n - размер выборки
--ИЗМЕНИ n, если нужен другой размер выборки
DECLARE n INT64 DEFAULT 10;

--ЭТО НЕ ТРОГАЙ (нужно для цикла)
DECLARE i INT64 DEFAULT 0;

--сюда будем записывать
CREATE TEMP TABLE task_1 (sample_id STRING);

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
  --вводим рэнджи долей (от - до, в пределах от 0 до 1) - должны быть одинаковыми по размеру
  task_1 AS (
  SELECT
    item_id,
    IF(_range = MIN(_range) OVER (), 0, LAG(_range) OVER (ORDER BY _range)) AS from_,
    IF(_range = MAX(_range) OVER (), 1, _range) AS _to
  FROM
    (SELECT
      item_id,
      ROW_NUMBER() OVER()/COUNT(value) OVER () AS _range
    FROM base))

  SELECT
    *
  FROM task_1
);

--сэмплирование
LOOP
  SET i = i + 1;
  IF i > n THEN
    LEAVE;
  ELSE
  --берем рандомно строку и кидаем ее в task_1
  INSERT INTO task_1 (sample_id)
  SELECT
    item_id
  FROM
    (SELECT 
      item_id,
      from_,
      _to,
    FROM temptable) AS t1
  --да-да, тяжелая операция - но нам нужно одно случайное значение для всех!  
    CROSS JOIN
    (SELECT
    --генератор случайный значений от 0 до 1
      RAND() AS random) AS t2
  WHERE random BETWEEN from_ AND _to;
  END IF;
END LOOP;
```
