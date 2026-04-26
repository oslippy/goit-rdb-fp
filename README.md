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

  
