# Динамічне маскування даних

Динамічне маскування даних (DDM) дозволяє приховувати чутливу інформацію, надаючи доступ до замаскованих стовпців. DDM не змінює збережені дані, а лише представляє їх у замаскованому вигляді під час виконання запитів SELECT. Це забезпечує певний рівень захисту від випадкового розкриття даних. Проте будь-хто, хто має прямий доступ до файлів SSTable, зможе переглянути незахищені дані.

## Функції маскування:

DDM базується на наборі вбудованих функцій CQL, які дозволяють приховувати чутливу інформацію. Доступні функції:

| Функція  |  Опис  |
|---------------------------|--------------------------------|
| mask_null(value)  | Замінює перший аргумент на стовпець із null-значенням. Повернене значення завжди є відсутнім стовпцем, а не стовпцем із null-значенням. Приклади: `mask_null('Alice') → null` / `mask_null(123) → null`|
| mask_default(value)   | Замінює аргумент на фіксоване значення за замовчуванням того ж типу. Наприклад: `****` для текстових значень, `0` для чисел, `false` для булевих.Типи зі змінною довжиною (списки, множини, мапи) маскуються як порожні колекції. Типи з фіксованою довжиною маскуються значеннями за типом. Приклади: `mask_default('Alice') → '****'` / `mask_default(123) → 0` / `mask_default((list<int>) [1, 2, 3]) → []` / `mask_default((vector<int, 3>) [1, 2, 3]) → [0, 0, 0]` |
| mask_replace(value, replacement) | Замінює перший аргумент на значення, задане другим аргументом, яке має бути того ж типу. Приклади: `mask_replace('Alice', 'REDACTED') → 'REDACTED'` / `mask_replace(123, -1) → -1` |
| mask_inner(value, begin, end, [padding]) | Повертає копію текстового аргументу, замінюючи всі символи, окрім перших і останніх, на символ заповнення (за замовчуванням `*`). Аргументи `begin` і `end` задають кількість видимих символів на початку та в кінці. Приклади: / `mask_inner('Alice', 1, 2) → 'Ace'` / `mask_inner('Alice', 1, null) → 'A'` / `mask_inner('Alice', null, 2) → '*ce'` / `mask_inner('Alice', 2, 1, '#') → 'Al##e'` |
| mask_outer(value, begin, end, [padding]) | Повертає копію текстового аргументу, замінюючи перший і останній символи на символ заповнення (за замовчуванням `*`). Аргументи `begin` і `end` задають кількість видимих символів на початку та в кінці. Приклади:`mask_outer('Alice', 1, 2) → '*li'` / `mask_outer('Alice', 1, null) → '*lice'` / `mask_outer('Alice', null, 2) → 'Ali'` / `mask_outer('Alice', 2, 1, '#') → '##ic#'` |
| mask_hash(value, [algorithm]) | Повертає blob із хешем переданого значення. Другий аргумент (необов’язковий) визначає алгоритм хешування. Алгоритм за замовчуванням — SHA-256. Приклади: `mask_hash('Alice')` / `mask_hash('Alice', 'SHA-512')` |

Функції маскування можуть використовуватися в запитах `SELECT`, щоб отримати замаскований вигляд даних.

#### Наприклад:
```sql
CREATE TABLE patients (
   id timeuuid PRIMARY KEY,
   name text,
   birth date
);

INSERT INTO patients(id, name, birth) VALUES (now(), 'alice', '1982-01-02');
INSERT INTO patients(id, name, birth) VALUES (now(), 'bob', '1982-01-02');

SELECT mask_inner(name, 1, null), mask_default(birth) FROM patients;
```
#### Результат
```
system.mask_inner(name, 1, NULL) | system.mask_default(birth)
---------------------------------+-----------------------------
                            b**  |                 1970-01-01
                          a****  |                 1970-01-01
```

## Прикріплення функцій маскування до стовпців таблиці

Функцію маскування можна постійно закріпити за будь-яким стовпцем таблиці. Якщо стовпець визначено як замаскований, запити `SELECT` завжди повертатимуть значення цього стовпця у замаскованій формі. Маскування є прозорим для користувачів, які виконують запити `SELECT`. Єдиний спосіб дізнатися, що стовпець замасковано, — це перевірити визначення таблиці.

Ця функція є необов’язковою та за замовчуванням вимкнена. Щоб увімкнути її, необхідно активувати властивість `dynamic_data_masking_enabled` у файлі `cassandra.yaml`.

Маски для стовпців таблиці можна визначати під час створення схеми таблиці за допомогою `CREATE TABLE`. Наприклад, функція `mask_inner` із двома аргументами:

```sql
CREATE TABLE patients (
    id timeuuid PRIMARY KEY,
    name text MASKED WITH mask_inner(1, null),
    birth date MASKED WITH mask_default()
 );
```
Під час використання запиту `SELECT` із цими даними для функції `mask_inner` потрібні три аргументи, але перший аргумент завжди пропускається при закріпленні функції за схемою таблиці. Значення цього першого аргументу завжди інтерпретується як значення замаскованого стовпця, у цьому випадку текстового стовпця.

З тієї ж причини функція маскування `mask_default` не вимагає жодного аргументу при створенні схеми таблиці, але потребує одного аргументу під час використання в запитах `SELECT`.

Дані можуть вставлятися у замасковану таблицю без змін. Наприклад:

```sql
INSERT INTO patients(id, name, birth) VALUES (now(), 'alice', '1984-01-02');
INSERT INTO patients(id, name, birth) VALUES (now(), 'bob', '1982-02-03');
```
Запит `SELECT` поверне замасковані дані. Функція маскування автоматично застосовуватиметься до значень стовпців.
```sql
SELECT name, birth FROM patients;

//  name  | birth
// -------+------------
//  a**** | 1970-01-01
//    b** | 1970-01-01
```
Запит `ALTER TABLE` можна використовувати для зміни функції маскування для стовпця таблиці.
```sql
ALTER TABLE patients ALTER name
  MASKED WITH mask_default();
```
Аналогічно, функцію маскування можна від’єднати від стовпця за допомогою запиту `ALTER TABLE`:
```sql
ALTER TABLE patients ALTER name
  DROP MASKED;
```

## Дозволи

Звичайні користувачі створюються без дозволу `UNMASK` і бачать лише замасковані значення. Надання дозволу `UNMASK` дозволяє користувачеві отримувати незамасковані значення стовпців. Суперкористувачі автоматично отримують дозвіл `UNMASK` і бачать незамасковані значення у результатах запитів `SELECT`.

Наприклад, припустимо, у нас є таблиця із замаскованими стовпцями:

```sql
CREATE TABLE patients (
    id timeuuid PRIMARY KEY,
    name text MASKED WITH mask_inner(1, null),
    birth date MASKED WITH mask_default()
 );
```
Додамо дані до таблиці:

```sql
INSERT INTO patients(id, name, birth) VALUES (now(), 'alice', '1984-01-02');
INSERT INTO patients(id, name, birth) VALUES (now(), 'bob', '1982-02-03');
```
Користувач без дозволу `UNMASK` бачить замасковані дані:

```sql
LOGIN unprivileged
SELECT name, birth FROM patients;

//  name  | birth
// -------+------------
//  a**** | 1970-01-01
//    b** | 1970-01-01
```
Створимо двох користувачів із дозволом `SELECT` для таблиці, але надамо дозвіл `UNMASK` лише одному з них:

```sql
CREATE USER privileged WITH PASSWORD 'xyz';
GRANT SELECT ON TABLE patients TO privileged;
GRANT UNMASK ON TABLE patients TO privileged;

CREATE USER unprivileged WITH PASSWORD 'xyz';
GRANT SELECT ON TABLE patients TO unprivileged;
```
Користувач із дозволом `UNMASK` бачить незамасковані дані:

```sql
LOGIN privileged
SELECT name, birth FROM patients;

//  name  | birth
// -------+------------
//  alice | 1984-01-02
//    bob | 1982-02-03
```
Користувач без дозволу `UNMASK` бачить лише замасковані дані:

```sql
LOGIN unprivileged
SELECT name, birth FROM patients;

//  name  | birth
// -------+------------
//  a**** | 1970-01-01
//    b** | 1970-01-01
```
Дозвіл `UNMASK` працює, як і будь-який інший дозвіл, і може бути відкликаний:

```sql
REVOKE UNMASK ON TABLE patients
  FROM privileged;
```
Важливо: якщо автентифікація вимкнена, анонімний користувач за замовчуванням має всі дозволи, включно з `UNMASK`, і може бачити незамасковані дані. Таким чином, використання функцій маскування даних має сенс лише за умови ввімкненої автентифікації.

Тільки користувачі з дозволом `UNMASK` можуть використовувати замасковані стовпці в умові `WHERE` запиту `SELECT`. Користувачі без дозволу `UNMASK` отримують таку помилку:

```sql
CREATE USER untrusted_user WITH PASSWORD 'xyz';
GRANT SELECT ON TABLE patients TO untrusted_user;
LOGIN untrusted_user
SELECT name, birth FROM patients WHERE name = 'Alice' ALLOW FILTERING;

// Unauthorized: Error from server: code=2100 [Unauthorized] message="User untrusted_user has no UNMASK nor SELECT_UNMASK permission on table k.patients"
```
У деяких випадках довірений користувач бази даних повинен створювати замасковані дані для запитів від ненадійних зовнішніх користувачів. Наприклад, довірений додаток може підключатися до бази даних і запитувати замасковані дані для відображення кінцевим користувачам. У такому випадку довірений користувач (додаток) може отримати дозвіл `SELECT_MASKED`. Цей дозвіл дозволяє запитувати замасковані стовпці в умові WHERE запиту `SELECT`, але результати все одно будуть замаскованими:

```sql
CREATE USER trusted_user WITH PASSWORD 'xyz';
GRANT SELECT, SELECT_MASKED ON TABLE patients TO trusted_user;
LOGIN trusted_user
SELECT name, birth FROM patients WHERE name = 'Alice' ALLOW FILTERING;

//  name  | birth
// -------+------------
//  a**** | 1970-01-01
```

## Користувацькі функції

До стовпця таблиці можна прикріплювати користувацькі функції (User-Defined Functions, UDFs). UDF, які використовуються для маскування, повинні належати до того ж простору ключів, що й замаскована таблиця. Значення стовпця, яке потрібно замаскувати, передається як перший аргумент прикріпленої UDF. Тому UDF, прикріплені до стовпця, повинні мати щонайменше один аргумент, і цей аргумент має відповідати типу замаскованого стовпця. Також прикріплена UDF повинна повертати значення того ж типу, що й замаскований стовпець.

#### Наприклад:

```sql
CREATE FUNCTION redact(input text)
   CALLED ON NULL INPUT
   RETURNS text
   LANGUAGE java
   AS 'return "redacted";';

CREATE TABLE patients (
   id timeuuid PRIMARY KEY,
   name text MASKED WITH redact(),
   birth date
);
```
Це створює залежність між схемою таблиці та функціями. Будь-яка спроба видалити функцію буде відхилена, доки існує ця залежність. Щоб видалити функцію, спочатку потрібно прибрати маскування стовпця в таблиці:

```sql
ALTER TABLE patients ALTER name
  DROP MASKED;
```
Видалення стовпця, таблиці або простору ключів, які містять стовпець, також видалить цю залежність.

Зверніть увагу, що агрегатні функції не можна використовувати як функції маскування.