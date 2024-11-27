# Функції

CQL підтримує дві основні категорії функцій:
- Скалярні функції (Scalar functions):
Приймають одне або кілька значень та повертають один результат.
- Агрегатні функції (Aggregate functions):
Агрегують кілька рядків, отриманих у результаті запиту SELECT.

CQL надає як вбудовані (нативні) функції, так і можливість створювати власні користувацькі функції.

*Використання користувацьких функцій за замовчуванням вимкнено через міркування безпеки. Навіть якщо їх увімкнено, виконання таких функцій обмежене "пісочницею", що знижує ризик, але не гарантує абсолютного захисту. Щоб увімкнути користувацькі функції, необхідно налаштувати параметр `user_defined_functions_enabled` у файлі `cassandra.yaml`.*

Функції ідентифікуються за їхнім іменем:

```bnf
function_name ::= [ keyspace_name'.' ] name
```

## Скалярні функції
### Нативні функції
#### Cast
Функція `cast` використовується для перетворення одного типу даних у інший.

Підтримувані перетворення:

|Перетворюваний тип |	Перетворення до типу |
|--|-----|
|ascii |	text, varchar |
|bigint |	tinyint, smallint, int, float, double, decimal, varint, text, varchar |
|boolean |	text, varchar |
|counter |	tinyint, smallint, int, bigint, float, double, decimal, varint, text, varchar |
|date |	timestamp |
|decimal |	tinyint, smallint, int, bigint, float, double, varint, text, varchar |
|double |	tinyint, smallint, int, bigint, float, decimal, varint, text, varchar |
|float |	tinyint, smallint, int, bigint, double, decimal, varint, text, varchar |
|inet |	text, varchar |
|int |	tinyint, smallint, bigint, float, double, decimal, varint, text, varchar |
|smallint |	tinyint, int, bigint, float, double, decimal, varint, text, varchar |
|time |	text, varchar |
|timestamp |	date, text, varchar |
|timeuuid |	timestamp, date, text, varchar |
|tinyint |	tinyint, smallint, int, bigint, float, double, decimal, varint, text, varchar |
|uuid |	text, varchar |
|varint |	tinyint, smallint, int, bigint, float, double, decimal, text, varchar |
Примітки:

Перетворення строго відповідають семантиці Java. Наприклад, значення double зі значенням 1 буде перетворено на текстове значення '1.0'.
Приклад використання:
```sql
SELECT avg(cast(count AS double)) FROM myTable;
```
#### Token
Функція `token` обчислює токен для заданого ключа розділу. Підпис функції залежить від таблиці та партиціонера, який використовується кластером.

| Партиціонер |	Тип результату |
|----------|--------------|
| Murmur3Partitioner |	bigint |
| RandomPartitioner |	varint |
| ByteOrderedPartitioner |	blob |
Приклад таблиці:

```sql
CREATE TABLE users (
    userid text PRIMARY KEY,
    username text,
);
```
Таблиця використовує партиціонер за замовчуванням `Murmur3Partitioner`. Функція token приймає єдиний аргумент `text` (тип ключа розділу `userid`) і повертає результат типу `bigint`.

#### uuid
Функція `uuid` не приймає параметрів і генерує випадковий UUID типу 4. Вона підходить для використання в запитах `INSERT` або `UPDATE`.

#### Функції для timeuuid
##### `now`
Функція `now` не приймає аргументів і генерує на координуючому вузлі унікальний `timeuuid` у момент виклику функції.

Використовується для вставки даних, але її значення непридатне для використання в умовах `WHERE`.
Наприклад:
```sql
SELECT * FROM myTable WHERE t = now();
```
Цей запит не поверне результат, оскільки `now()` завжди повертає унікальне значення.
Аліасом для `now` є `current_timeuuid`.

##### `min_timeuuid` і `max_timeuuid`
Ці функції приймають значення `timestamp` (або рядок дати) і повертають "фейкові" `timeuuid`, відповідно, найменше та найбільше можливе для заданого часу.

Приклад:
```sql
SELECT * FROM myTable
 WHERE t > max_timeuuid('2013-01-01 00:05+0000')
   AND t < min_timeuuid('2013-02-02 10:00+0000');
```
Цей запит вибирає всі рядки, де стовпець `t` (`timeuuid`) пізніший за `'2013-01-01 00:05+0000'` і раніший за `'2013-02-02 10:00+0000'`.

*Значення, згенеровані `min_timeuuid` і `max_timeuuid`, є "фейковими", оскільки вони не дотримуються процесу генерації UUID на основі часу, визначеного [IETF RFC 4122](https://datatracker.ietf.org/doc/html/rfc4122). Використовуйте ці методи лише для запитів, а не для вставки даних, щоб уникнути можливого перезапису даних.*

#### Функції роботи з датою та часом
##### Отримання поточної дати/часу
Для отримання дати/часу в момент виклику доступні наступні функції:

| Ім'я функції |	Тип результату |
|---|---|
| current_timestamp |	timestamp |
| current_date |	date |
| current_time |	time |
| current_timeuuid |	timeUUID |

Приклад: Вибір даних за останні два дні:

```sql
SELECT * FROM myTable WHERE date >= current_date() - 2d;
```
##### Функції для перетворення часу
Функції для перетворення `timeuuid`, `timestamp` або `date` в інші типи:

| Ім'я функції |	Тип вхідних даних	| Опис |
|---|---|---|
| to_date |	timeuuid |	Перетворює timeuuid на date |
| to_date |	timestamp |	Перетворює timestamp на date |
| to_timestamp |	timeuuid |	Перетворює timeuuid на timestamp |
| to_timestamp |	date |	Перетворює date на timestamp |
| to_unix_timestamp |	timeuuid |	Перетворює timeuuid на bigInt |
| to_unix_timestamp |	timestamp |	Перетворює timestamp на bigInt |
| to_unix_timestamp |	date |	Перетворює date на bigInt |


#### Функції для перетворення в Blob
CQL надає функції для перетворення нативних типів у бінарні дані (blob) і назад:

- `type_as_blob` приймає аргумент нативного типу і повертає його у вигляді blob.
- `blob_as_type` приймає аргумент типу blob і перетворює його в заданий тип.

Приклад:

```cs
bigint_as_blob(3) → 0x0000000000000003
blob_as_bigint(0x0000000000000003) → 3
```

#### Математичні функції
CQL підтримує наступні математичні функції, результат яких завжди має той самий тип, що й вхідне значення:

| Назва функції |	Опис |
|--|--|
| abs |	Повертає абсолютне значення аргументу. |
| exp |	Повертає число e в степені аргументу. |
| log |	Повертає натуральний логарифм аргументу. |
| log10 |	Повертає логарифм бази 10 аргументу. |
| round |	Округлює аргумент до найближчого цілого числа (режим округлення HALF_UP). |

#### Функції для колекцій
CQL надає функції для роботи зі стовпцями типу колекцій:

| Назва функції |	Тип вхідного аргументу | Опис |
|---|---|---|
| map_keys |	map |	Повертає ключі мапи у вигляді множини. |
| map_values |	map	| Повертає значення мапи у вигляді списку. |
| collection_count |	map, set, list	| Повертає кількість елементів у колекції. |
| collection_min |	set, list	| Повертає мінімальний елемент колекції. |
| collection_max |	set, list	| Повертає максимальний елемент колекції. |
| collection_sum |	Числовий set, list	| Обчислює суму елементів колекції. |
| collection_avg |	Числовий set, list	| Обчислює середнє значення елементів колекції. Повертає 0 для порожніх колекцій. |

*Функції `collection_sum` і `collection_avg` повертають значення того ж типу, що й елементи колекції, що може спричинити переповнення.*

#### Функції для маскування даних
CQL підтримує функції для приховування реального вмісту чутливих даних:

| Функція | Опис |
|---|---|
| mask_null(value) |	Замінює значення на null. |
| mask_default(value) |	Замінює значення на фіксоване значення за замовчуванням (наприклад, **** для тексту, 0 для чисел). |
| mask_replace(value, replacement) |	Замінює значення аргументу на вказане замінне значення. |
| mask_inner(value, begin, end, [padding]) |	Замінює всі символи текстового аргументу, окрім перших і останніх, символом заповнення. |
| mask_outer(value, begin, end, [padding]) |	Замінює перший і останній символи тексту символом заповнення. |
| mask_hash(value, [algorithm]) |	Повертає blob, що містить хеш аргументу. За замовчуванням використовується алгоритм SHA-256. |

Приклади:
```cs
mask_null('Alice') → null
mask_default('Alice') → '****'
mask_replace('Alice', 'REDACTED') → 'REDACTED'
mask_inner('Alice', 1, 2) → 'Ace'
mask_outer('Alice', 1, 2) → '*li'
mask_hash('Alice') → [хеш значення]
```

#### Функції для обчислення схожості векторів
CQL надає функції для обчислення коефіцієнтів схожості між векторами чисел із плаваючою комою.

| Функція |	Опис |
|---|---|
| similarity_cosine(vector, vector)	| Обчислює косинусну схожість між двома векторами однакової розмірності. | 
| similarity_euclidean(vector, vector) |	Обчислює євклідову відстань між двома векторами однакової розмірності. | 
| similarity_dot_product(vector, vector) |	Обчислює скалярний добуток між двома векторами однакової розмірності. | 

##### `similarity_cosine(vector, vector)`
Обчислює косинусну схожість між двома векторами.

Результат варіюється від `-1` до `1`, де `1` означає ідеальну схожість, а `0` — повну ортогональність.

Приклади:

```cs
similarity_cosine([0.1, 0.2], null) → null
similarity_cosine([0.1, 0.2], [0.1, 0.2]) → 1
similarity_cosine([0.1, 0.2], [-0.1, -0.2]) → 0
similarity_cosine([0.1, 0.2], [0.9, 0.8]) → 0.964238
```
##### `similarity_euclidean(vector, vector)`
Обчислює євклідову відстань між двома векторами.
Чим менше значення, тим ближчі вектори у просторі.

Приклади:

```cs
similarity_euclidean([0.1, 0.2], null) → null
similarity_euclidean([0.1, 0.2], [0.1, 0.2]) → 1
similarity_euclidean([0.1, 0.2], [-0.1, -0.2]) → 0.833333
similarity_euclidean([0.1, 0.2], [0.9, 0.8]) → 0.5
```
##### `similarity_dot_product(vector, vector)`
Обчислює скалярний добуток між двома векторами.
Ця функція часто використовується для векторних представлень у машинному навчанні та рекомендованих системах.

Приклади:

```cs
similarity_dot_product([0.1, 0.2], null) → null
similarity_dot_product([0.1, 0.2], [0.1, 0.2]) → 0.525
similarity_dot_product([0.1, 0.2], [-0.1, -0.2]) → 0.475
similarity_dot_product([0.1, 0.2], [0.9, 0.8]) → 0.625
```

### Користувацькі функції

Користувацькі функції (User-Defined Functions, UDFs) дозволяють виконувати код, написаний користувачем, безпосередньо в Cassandra. За замовчуванням Cassandra підтримує створення функцій на Java.

#### Основні характеристики UDFs:
- Частина схеми Cassandra:
UDFs є частиною схеми Cassandra, автоматично розповсюджуються на всі вузли в кластері.
- Перевантаження функцій:
UDFs можуть бути перевантажені, тобто можна створювати кілька функцій з однаковими іменами, але різними типами аргументів.
Приклад:
```sql
CREATE FUNCTION sample(arg int) ...;
CREATE FUNCTION sample(arg text) ...;
```
- Використання складних типів:
UDFs підтримують аргументи та повертають значення складних типів, таких як колекції, кортежі (tuples) та користувацькі типи (UDTs). Вони використовують функції перетворення DataStax Java Driver.
- Уразливість до помилок:
UDFs можуть викликати винятки, наприклад, null pointer або незаконні аргументи. Якщо виняток виникає під час виконання функції, запит завершується невдало.

*Починаючи з Cassandra 4.1, UDFs на JavaScript застаріли і видаляються в Cassandra 5.0 (див. CASSANDRA-17281, CASSANDRA-18252).*

#### Створення користувацьких функцій
Функції визначаються за допомогою команди `CREATE FUNCTION`. Код функції огороджується подвійними знаками долара `$$`.

Приклад:

```sql
CREATE FUNCTION some_function(arg int)
    RETURNS NULL ON NULL INPUT
    RETURNS int
    LANGUAGE java
    AS $$ return arg; $$;

SELECT some_function(column) FROM atable ...;
UPDATE atable SET col = some_function(?) ...;
```

Користувацькі типи даних (UDT) та кортежі можуть використовуватись як аргументи та значення, що повертаються.

```sql
CREATE TYPE custom_type (txt text, i int);
CREATE FUNCTION fct_using_udt(udtarg frozen<custom_type>)
    RETURNS NULL ON NULL INPUT
    RETURNS text
    LANGUAGE java
    AS $$ return udtarg.getString("txt"); $$;
```

UDFContext надає функції для створення нових значень UDT та кортежів:

Приклад створення UDT у функції:

```sql
CREATE TYPE custom_type (txt text, i int);
CREATE FUNCTION fct_using_udt(somearg int)
    RETURNS NULL ON NULL INPUT
    RETURNS custom_type
    LANGUAGE java
    AS $$
        UDTValue udt = udfContext.newReturnUDTValue();
        udt.setString("txt", "some string");
        udt.setInt("i", 42);
        return udt;
    $$;
```

Інтерфейс UDFContext:

```java
public interface UDFContext {
    UDTValue newArgUDTValue(String argName);
    UDTValue newArgUDTValue(int argNum);
    UDTValue newReturnUDTValue();
    UDTValue newUDTValue(String udtName);
    TupleValue newArgTupleValue(String argName);
    TupleValue newArgTupleValue(int argNum);
    TupleValue newReturnTupleValue();
    TupleValue newTupleValue(String cqlDefinition);
}
```
Java UDFs мають імпортовані такі класи та інтерфейси за замовчуванням:

```java
Copy code
import java.nio.ByteBuffer;
import java.util.List;
import java.util.Map;
import java.util.Set;
import org.apache.cassandra.cql3.functions.UDFContext;
import com.datastax.driver.core.TypeCodec;
import com.datastax.driver.core.TupleValue;
import com.datastax.driver.core.UDTValue;
```

*Ці імпорти недоступні для скриптових UDFs.*


## Створення функції

Створення нової користувацької функції (User-Defined Function, UDF) здійснюється за допомогою команди CREATE FUNCTION.

Синтаксис:
```bnf
create_function_statement::= CREATE [ OR REPLACE ] FUNCTION [ IF NOT EXISTS]
	function_name '(' arguments_declaration ')'
	[ CALLED | RETURNS NULL ] ON NULL INPUT
	RETURNS cql_type
	LANGUAGE identifier
	AS string arguments_declaration: identifier cql_type ( ',' identifier cql_type )*
```

Особливості:
- `OR REPLACE`: Створює нову функцію або замінює існуючу з тією ж сигнатурою.
- `IF NOT EXISTS`: Дозволяє створювати функцію тільки якщо її сигнатура ще не існує.
*Не можна використовувати `OR REPLACE` та `IF NOT EXISTS` одночасно.*
- Обробка `NULL`:
`RETURNS NULL ON NULL INPUT` — повертає `null`, якщо будь-який аргумент дорівнює `null`.
`CALLED ON NULL INPUT` — викликає функцію навіть для `null`-аргументів.

Приклади:
Створення та заміна функції:

```sql
CREATE OR REPLACE FUNCTION somefunction(somearg int, anotherarg text)
    RETURNS NULL ON NULL INPUT
    RETURNS text
    LANGUAGE java
    AS $$
        return anotherarg + somearg;
    $$;
```
Створення функції в певному просторі ключів:

```sql
CREATE FUNCTION IF NOT EXISTS mykeyspace.fname(someArg int)
    CALLED ON NULL INPUT
    RETURNS text
    LANGUAGE java
    AS $$
        return "Hello " + someArg;
    $$;
```

## Видалення функції
Команда `DROP FUNCTION` видаляє існуючу функцію.

Синтаксис:
```bnf
drop_function_statement::= DROP FUNCTION [ IF EXISTS ] function_name [ '(' arguments_signature ')' ]
arguments_signature::= cql_type ( ',' cql_type )*
```
Особливості:
- `IF EXISTS`: Уникає помилки, якщо функція не існує.
- Якщо існує кілька функцій з однаковою назвою, необхідно вказати типи аргументів у `arguments_signature`.

Приклади:
```sql
DROP FUNCTION myfunction;
DROP FUNCTION mykeyspace.afunction;
DROP FUNCTION afunction(int);
DROP FUNCTION afunction(text);
```

## Агрегатні функції
Агрегатні функції обробляють набір рядків і повертають одне значення.

### Вбудовані функції:
#### Підрахування рядків.
Функція `COUNT` використовується для підрахунку кількості рядків, що повертаються у результаті запиту.

```sql
SELECT COUNT(*) FROM plays;
SELECT COUNT(scores) FROM plays;
```

#### Обчислення мінімального та максимального значення
`MIN` та `MAX`: Обчислює мінімальне та максимальне значення в стовпці.

```sql
SELECT MIN(players), MAX(players) FROM plays WHERE game = 'quake';
```

#### Обчислення суми

`SUM`: Обчислює суму значень стовпця.

```sql
SELECT SUM(players) FROM plays;
SELECT SUM(CAST(players AS VARINT)) FROM plays;
```

#### Обчислення середнього значення

`AVG`: Обчислює середнє значення стовпця.

```sql
SELECT AVG(players) FROM plays;
SELECT AVG(CAST(players AS FLOAT)) FROM plays;
```

### Користувацькі агрегати (UDA)
Користувацькі агрегати дозволяють створювати власні агрегатні функції. Для цього потрібно:

- Визначити функцію стану (`SFUNC`).
- Визначити тип стану (`STYPE`).
- *(Необов’язково)* Визначити функцію завершення (`FINALFUNC`).
- Ініціалізувати стан (`INITCOND`).

Приклад UDA:
Функція для підрахунку середнього значення:

```sql
CREATE OR REPLACE FUNCTION test.averageState(state tuple<int,bigint>, val int)
    CALLED ON NULL INPUT
    RETURNS tuple
    LANGUAGE java
    AS $$
        if (val != null) {
            state.setInt(0, state.getInt(0) + 1);
            state.setLong(1, state.getLong(1) + val.intValue());
        }
        return state;
    $$;

CREATE OR REPLACE FUNCTION test.averageFinal(state tuple<int,bigint>)
    CALLED ON NULL INPUT
    RETURNS double
    LANGUAGE java
    AS $$
        if (state.getInt(0) == 0) return null;
        return (double) state.getLong(1) / state.getInt(0);
    $$;

CREATE OR REPLACE AGGREGATE test.average(int)
    SFUNC averageState
    STYPE tuple
    FINALFUNC averageFinal
    INITCOND (0, 0);
```
Використання агрегатної функції:

```sql
CREATE TABLE test.atable (
    pk int PRIMARY KEY,
    val int
);

INSERT INTO test.atable (pk, val) VALUES (1, 1);
INSERT INTO test.atable (pk, val) VALUES (2, 2);
INSERT INTO test.atable (pk, val) VALUES (3, 3);

SELECT test.average(val) FROM test.atable;
```

## Створення користувацької агрегатної функції
Команда `CREATE AGGREGATE` використовується для створення (або заміни) користувацької агрегатної функції.

Синтаксис:
```bnf
create_aggregate_statement ::= CREATE [ OR REPLACE ] AGGREGATE [ IF NOT EXISTS ]
                                function_name '(' arguments_signature')'
                                SFUNC function_name
                                STYPE cql_type:
                                [ FINALFUNC function_name]
                                [ INITCOND term ]
```
Опис ключових компонентів:
- `OR REPLACE`:
Створює нову агрегатну функцію або замінює існуючу з тією ж сигнатурою.
Якщо не вказано, команда завершується помилкою, якщо функція з тією ж сигнатурою вже існує.

- `IF NOT EXISTS`:
Створює агрегатну функцію, тільки якщо вона ще не існує.
*Не можна використовувати разом із `OR REPLACE`.*

- `STYPE`:
Тип даних для стану функції, обов’язковий параметр.

- `INITCOND`:
Початкове значення стану агрегату. Якщо не вказано, за замовчуванням `null`.

- `SFUNC`:
Ім'я існуючої функції, яка обробляє стан (state function).
    - Перший аргумент цієї функції має бути типу `STYPE`.
    - Інші аргументи повинні відповідати типам аргументів агрегату.

- `FINALFUNC` (опціонально):
Виконується після обробки всіх рядків із поточним станом як аргументом.
    - Приймає єдиний аргумент типу `STYPE`.
    - Тип, що повертається, може відрізнятися від `STYPE`.

Поведінка з `NULL`:
Якщо станова функція оголошена з `RETURNS NULL ON NULL INPUT` і викликається з `null`, стан не оновлюється.
Якщо останній стан є `null` і функція завершення (`FINALFUNC`) оголошена з `RETURNS NULL ON NULL INPUT`, результат агрегату буде `null`.
Результат:
Якщо `FINALFUNC` не визначена, тип результату — `STYPE`.
Якщо `FINALFUNC` визначена, тип результату — тип, що повертається цією функцією.
Приклад:
Функція обчислення середнього значення:

```sql
CREATE OR REPLACE FUNCTION test.averageState(state tuple<int, bigint>, val int)
    CALLED ON NULL INPUT
    RETURNS tuple
    LANGUAGE java
    AS $$
        if (val != null) {
            state.setInt(0, state.getInt(0) + 1);
            state.setLong(1, state.getLong(1) + val.intValue());
        }
        return state;
    $$;

CREATE OR REPLACE FUNCTION test.averageFinal(state tuple<int, bigint>)
    CALLED ON NULL INPUT
    RETURNS double
    LANGUAGE java
    AS $$
        if (state.getInt(0) == 0) return null;
        return (double) state.getLong(1) / state.getInt(0);
    $$;

CREATE OR REPLACE AGGREGATE test.average(int)
    SFUNC averageState
    STYPE tuple
    FINALFUNC averageFinal
    INITCOND (0, 0);
```
Використання агрегатної функції:

```sql
SELECT test.average(column_name) FROM table_name;
```

## Видалення користувацької агрегатної функції
Команда `DROP AGGREGATE` видаляє користувацьку агрегатну функцію.

Синтаксис:
```bnf
drop_aggregate_statement::= DROP AGGREGATE [ IF EXISTS ] function_name[ '(' arguments_signature ')'
]
```
Опис:
`IF EXISTS`:
Видаляє функцію, якщо вона існує; нічого не робить, якщо її немає.

`arguments_signature`:
Вказується, якщо є кілька перевантажених функцій з однаковими іменами, але різними сигнатурами.

Приклади:
Видалення функції за назвою:

```sql
DROP AGGREGATE myAggregate;
```
Видалення функції в певному просторі ключів:

```sql
DROP AGGREGATE myKeyspace.anAggregate;
```
Видалення функції з певною сигнатурою:

```sql
DROP AGGREGATE someAggregate(int);
DROP AGGREGATE someAggregate(text);
```