---
# Фінальний проєкт з дисципліни "Relational Databases: Concepts and Techniques"

## 1. Завантажуємо дані:

* Створюємо схему `pandemic` у базі даних за допомогою `SQL`-команди
  
  ~~~~sql
  CREATE schema IF NOT EXISTS pandemic;
  ~~~~

* Обрираємо схему `pandemic` за замовчуванням за допомогою SQL-команди
  
  ~~~~sql
  USE pandemic;
  ~~~~

* Імпортуйте дані за допомогою **Import wizard** та ознайомимось з ними, щоб бути у контексті

  ~~~~sql
  SELECT * FROM infectious_cases LIMIT 25;
  ~~~~
  
  ![select limit data](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2011.04.04.png)

## 2. Нормалізуємо таблицю `infectious_cases` до 3ї нормальної форми та зберігаємо у цій же схемі дві таблиці з нормалізованими даними

* Виконаємо запит, щоб ментор міг зрозуміти, скільки записів завантажено у базу даних із файлу

  ~~~~sql
  SELECT COUNT(*) FROM infectious_cases;
  ~~~~

  ![count rows](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2011.10.48.png)

* DDL для створення нових таблиць

  ~~~~sql
  CREATE TABLE entities (
    entity_id INT PRIMARY KEY AUTO_INCREMENT,
    entity_name VARCHAR(100) NOT NULL UNIQUE,
    code CHAR(10) NULL
  );

  CREATE TABLE normalized_infectious_cases (
    entity_id INT NOT NULL,
    year SMALLINT NOT NULL,
    Number_yaws INT NULL,
    polio_cases INT NULL,
    cases_guinea_worm INT NULL,
    Number_rabies INT NULL,
    Number_malaria INT NULL,
    Number_hiv INT NULL,
    Number_tuberculosis INT NULL,
    Number_smallpox INT NULL,
    Number_cholera_cases INT NULL,
    PRIMARY KEY (entity_id, year),
    FOREIGN KEY (entity_id) REFERENCES entities(entity_id)
  );
  ~~~~

  ![created tables](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2011.26.46.png)

* Мігруємо дані до нових таблиць

    ~~~~sql
    -- Крок 1: довідник entities
    INSERT INTO entities (entity_name, code)
    SELECT DISTINCT
      Entity AS entity_name,
      NULLIF(TRIM(Code), '') AS code      -- порожні рядки після імпорту CSV → NULL
    FROM infectious_cases
    ORDER BY Entity;

    -- Крок 2: факти захворюваності
    INSERT INTO normalized_infectious_cases (
      entity_id, year,
      Number_yaws, polio_cases, cases_guinea_worm,
      Number_rabies, Number_malaria, Number_hiv,
      Number_tuberculosis, Number_smallpox, Number_cholera_cases
    )
    SELECT
      e.entity_id,
      ic.Year,
      NULLIF(TRIM(ic.Number_yaws), '') ,
      NULLIF(TRIM(ic.polio_cases), '') ,
      NULLIF(TRIM(ic.cases_guinea_worm), '') ,
      NULLIF(TRIM(ic.Number_rabies), '') ,
      NULLIF(TRIM(ic.Number_malaria), '') ,
      NULLIF(TRIM(ic.Number_hiv), '') ,
      NULLIF(TRIM(ic.Number_tuberculosis), '') ,
      NULLIF(TRIM(ic.Number_smallpox), '') ,
      NULLIF(TRIM(ic.Number_cholera_cases), '')
    FROM infectious_cases ic
    JOIN entities e ON e.entity_name = ic.Entity;
  ~~~~

## 3. Проаналізуємо дані:

Для кожної унікальної комбінації `Entity` та `Code` або їх `id` рахуємо середнє, мінімальне, максимальне значення та суму для атрибута `Number_rabies`. Результат сортуємо за порахованим середнім значенням у порядку спадання. Обераємо тільки 10 рядків для виведення на екран.

Так як ми вже маємо нормалізовану схему, згрупую за `entity_id` (він однозначно ідентифікує пару `Entity+Code`), а ім'я та код підтягну для зручності читання:

~~~~sql
SELECT
    e.entity_id,
    e.entity_name,
    e.code,
    AVG(nic.Number_rabies) AS avg_rabies,
    MIN(nic.Number_rabies) AS min_rabies,
    MAX(nic.Number_rabies) AS max_rabies,
    SUM(nic.Number_rabies) AS sum_rabies
FROM normalized_infectious_cases nic
JOIN entities e ON e.entity_id = nic.entity_id
WHERE nic.Number_rabies IS NOT NULL
GROUP BY e.entity_id, e.entity_name, e.code
ORDER BY avg_rabies DESC
LIMIT 10;
~~~~

![group by](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2012.23.55.png)

## 4. Побудуємо колонку різниці в роках для нормованої таблиці

~~~~sql
SELECT
    year,
    MAKEDATE(year, 1) AS year_start,
    CURDATE() AS today,
    TIMESTAMPDIFF(YEAR, MAKEDATE(year, 1), CURDATE()) AS years_diff
FROM normalized_infectious_cases;
~~~~

![years diff](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2012.36.35.png)

## 5. Побудуємо власну функцію

* Створюємо і використаємо функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → `1996-01-01`).

  ~~~~sql
  DELIMITER $$
  
  DROP FUNCTION IF EXISTS year_diff_from_today $$
  
  CREATE FUNCTION year_diff_from_today(input_year INT)
  RETURNS INT
  DETERMINISTIC
  READS SQL DATA
  BEGIN
      DECLARE result INT;
      SET result = TIMESTAMPDIFF(YEAR, MAKEDATE(input_year, 1), CURDATE());
      RETURN result;
  END $$
  
  DELIMITER ;
  ~~~~

* Використання функції `year_diff_from_today`

  ~~~~sql
  SELECT year_diff_from_today(1996) AS diff;
  ~~~~

  ![as scalar](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2012.47.37.png)

  ~~~~sql
  SELECT
    year,
    year_diff_from_today(year) AS years_diff
  FROM normalized_infectious_cases
  GROUP BY year
  ORDER BY year;
  ~~~~

  ![in query to table](https://github.com/oslippy/goit-rdb-fp/blob/main/Screenshot%202026-04-26%20at%2012.49.24.png)
