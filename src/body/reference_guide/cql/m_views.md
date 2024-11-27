# Матереалізовані представлення

Імена матеріалізованих представлень визначаються за шаблоном:

```bnf
view_name::= re('[a-zA-Z_0-9]+')
CREATE MATERIALIZED VIEW
```

## Створення матеріалізованого представлення

Створення матеріалізованого представлення здійснюється за допомогою команди 'CREATE MATERIALIZED VIEW':

```sql
create_materialized_view_statement::= CREATE MATERIALIZED VIEW [ IF NOT EXISTS ] view_name
    AS select_statement
    PRIMARY KEY '(' primary_key ')'
    WITH table_options
```
Приклад:

```sql
CREATE MATERIALIZED VIEW monkeySpecies_by_population AS
   SELECT * FROM monkeySpecies
   WHERE population IS NOT NULL AND species IS NOT NULL
   PRIMARY KEY (population, species)
   WITH comment='Allow query by population instead of species';
```
Ця команда створює нове матеріалізоване представлення. Кожне представлення є набором рядків, які відповідають рядкам базової таблиці, вказаної в команді `SELECT`. Матеріалізоване представлення не можна оновлювати безпосередньо, але оновлення базової таблиці автоматично відображаються в представленні.

Основні етапи створення матеріалізованого представлення:
- Команда `SELECT`, яка визначає дані, включені в представлення.
- Первинний ключ, визначений для представлення.
- Опції представлення.
Якщо представлення вже існує, команда повертає помилку, якщо не вказано опцію `IF NOT EXISTS`. У цьому випадку команда нічого не робить.

*За замовчуванням матеріалізовані представлення будуються в один потік. Початкове побудування можна паралелізувати, збільшивши кількість потоків через властивість `concurrent_materialized_view_builders` у файлі `cassandra.yaml`. Це налаштування також доступне під час роботи через JMX і команди `setconcurrentviewbuilders` та `getconcurrentviewbuilders` у `nodetool`.*

### `SELECT` у матеріалізованих представленнях
Команда `SELECT` при створенні матеріалізованого представлення обмежена в декількох аспектах:

- Дозволено лише вибір стовпців базової таблиці. Заборонено використовувати функції (агрегатні чи ні), приведення типів, терміни тощо. Аліаси також не підтримуються.
Можна використовувати `*` для вибору всіх стовпців, але статичні стовпці не можуть бути включені.
Обмеження `WHERE`:
    - Не можна включати `bind_marker`.
    - Стовпці, які не є частиною первинного ключа базової таблиці, повинні мати обмеження `IS NOT NULL`.
    - Жодних інших обмежень не дозволено.
    - Стовпці, що є частиною первинного ключа представлення, мають бути обмежені хоча б `IS NOT NULL`.

### Первинний ключ у матеріалізованих представленнях

Матеріалізоване представлення має мати первинний ключ, який відповідає таким правилам:

Має включати всі стовпці первинного ключа базової таблиці.
Може включати лише один стовпець, який не є частиною первинного ключа базової таблиці.
Приклад базової таблиці:

```sql
CREATE TABLE t (
    k int,
    c1 int,
    c2 int,
    v1 int,
    v2 int,
    PRIMARY KEY (k, c1, c2)
);
```
Дозволені представлення:

```sql
CREATE MATERIALIZED VIEW mv1 AS
   SELECT * FROM t
   WHERE k IS NOT NULL AND c1 IS NOT NULL AND c2 IS NOT NULL
   PRIMARY KEY (c1, k, c2);

CREATE MATERIALIZED VIEW mv1 AS
   SELECT * FROM t
   WHERE k IS NOT NULL AND c1 IS NOT NULL AND c2 IS NOT NULL
   PRIMARY KEY (v1, k, c1, c2);
```
Недозволені представлення:

```sql
CREATE MATERIALIZED VIEW mv1 AS
   SELECT * FROM t
   WHERE k IS NOT NULL AND c1 IS NOT NULL AND c2 IS NOT NULL AND v1 IS NOT NULL
   PRIMARY KEY (v1, v2, k, c1, c2); // Помилка: не можна включати обидва v1 і v2.

CREATE MATERIALIZED VIEW mv1 AS
   SELECT * FROM t
   WHERE c1 IS NOT NULL AND c2 IS NOT NULL
   PRIMARY KEY (c1, c2); // Помилка: k має бути включено, оскільки це стовпець первинного ключа базової таблиці.
```
### Опції представлення
Матеріалізоване представлення внутрішньо реалізоване як таблиця, тому під час його створення можна використовувати ті самі опції, що й для таблиці.

## Зміна матеріалізованого представлення
Після створення можна змінювати параметри матеріалізованого представлення за допомогою команди ALTER MATERIALIZED VIEW:

```sql
alter_materialized_view_statement::= ALTER MATERIALIZED VIEW [ IF EXISTS ] view_name WITH table_options
```
Можна змінювати ті ж параметри, що й під час створення, а також ті, що доступні для таблиць.
Якщо представлення не існує, команда поверне помилку, якщо не використано опцію `IF EXISTS`. У цьому випадку команда не виконує жодних дій.

## Видалення матеріалізованого представлення
Видалення матеріалізованого представлення здійснюється за допомогою команди DROP MATERIALIZED VIEW:

```sql
drop_materialized_view_statement::= DROP MATERIALIZED VIEW [ IF EXISTS ] view_name;
```
Якщо матеріалізоване представлення не існує, команда поверне помилку, якщо не використано опцію `IF EXISTS`. У цьому випадку команда не виконує жодних дій.

*Видалення стовпців базової таблиці, які не вибрані в матеріалізованому представленні (через команди `UPDATE base SET unselected_column = null` або `DELETE unselected_column FROM base`), може призвести до втрати оновлень для інших стовпців, отриманих через хінти або ремонт. Через цю проблему рекомендується уникати видалення стовпців базової таблиці, які не вибрані в матеріалізованих представленнях, до вирішення цієї проблеми у задачі CASSANDRA-13826.*