# JSON
Cassandra з версії 2.2 підтримує роботу з `JSON` у запитах `SELECT` та `INSERT`. Це забезпечує зручний спосіб роботи з документами у форматі `JSON`, але не змінює основний API CQL — схема таблиць усе ще зберігається.

## SELECT JSON
Ключове слово `JSON` у запиті `SELECT` повертає кожний рядок як окрему `JSON-кодовану` мапу.
Ключі в мапі відповідають назвам стовпців у результатах звичайного `SELECT`. Винятком є чутливі до регістру назви стовпців із великими літерами — вони оточуються подвійними лапками (із екрануванням).
Приклад:

```sql
SELECT JSON a, ttl(b) FROM mytable;
```
Особливості:

Значення повертаються у `JSON`-форматі, що відповідає типу даних стовпця (див. таблицю нижче).

## INSERT JSON
У запиті `INSERT` ключове слово `JSON` дозволяє вставляти `JSON-кодовану` мапу як єдиний рядок.
Формат `JSON` повинен відповідати результату `SELECT JSON` для тієї ж таблиці, включно з обов’язковим екрануванням чутливих до регістру назв стовпців.
Приклад:

```sql
INSERT INTO mytable JSON '{ "\"myKey\"": 0, "value": 0}';
```
Обробка пропущених стовпців:

`DEFAULT NULL`: Пропущені стовпці отримають значення NULL (створюється tombstone).
`DEFAULT UNSET`: Пропущені стовпці залишаються незмінними.

## JSON-кодування типів даних Cassandra
| Type     | Formats Accepted         | Return Format    | Notes                                                                 |
|--------------|-------------------------------|-----------------------|---------------------------------------------------------------------------|
| ascii    | string                        | string                | Uses JSON’s `\u` character escape.                                       |
| bigint   | integer, string               | integer               | String must be a valid 64-bit integer.                                   |
| blob     | string                        | string                | String should be `0x` followed by an even number of hex digits.          |
| boolean  | boolean, string               | boolean               | String must be `"true"` or `"false"`.                                    |
| date     | string                        | string                | Date in format `YYYY-MM-DD`, timezone UTC.                               |
| decimal  | integer, float, string        | float                 | May exceed 32- or 64-bit IEEE-754 floating point precision.              |
| double   | integer, float, string        | float                 | String must be a valid integer or float.                                 |
| float    | integer, float, string        | float                 | String must be a valid integer or float.                                 |
| inet     | string                        | string                | IPv4 or IPv6 address.                                                    |
| int      | integer, string               | integer               | String must be a valid 32-bit integer.                                   |
| list     | list, string                  | list                  | Uses JSON’s native list representation.                                  |
| map      | map, string                   | map                   | Uses JSON’s native map representation.                                   |
| smallint | integer, string               | integer               | String must be a valid 16-bit integer.                                   |
| set      | list, string                  | list                  | Uses JSON’s native list representation.                                  |
| text     | string                        | string                | Uses JSON’s `\u` character escape.                                       |
| time     | string                        | string                | Time of day in format `HH-MM-SS[.fffffffff]`.                            |
| timestamp* integer, string               | string                | A timestamp. Strings allow input as dates. Returns format `YYYY-MM-DD HH:MM:SS.SSS`. |
| timeuuid | string                        | string                | Type 1 UUID. See constant for the UUID format.                           |
| tinyint  | integer, string               | integer               | String must be a valid 8-bit integer.                                    |
| tuple    | list, string                  | list                  | Uses JSON’s native list representation.                                  |
| UDT      | map, string                   | map                   | Uses JSON’s native map representation with field names as keys.          |
| uuid     | string                        | string                | See constant for the UUID format.                                        |
| varchar  | string                        | string                | Uses JSON’s `\u` character escape.                                       |
| varint   | integer, string               | integer               | Variable length; may overflow 32- or 64-bit integers in client-side decoder. |

## Функція from_json()
Функція `from_json()` дозволяє використовувати `JSON` для окремих значень стовпців. Її можна використовувати:

у секції `VALUES` запиту `INSERT`,
у значеннях стовпців для запитів `UPDATE`, `DELETE` або `SELECT`.
Приклад:

```sql
INSERT INTO mytable (id, data) VALUES (1, from_json('{ "key": "value" }'));
```

## Функція to_json()
Функція `to_json()` використовується для перетворення окремих значень стовпців у `JSON`.
Її можна використовувати тільки в секції вибору `SELECT`.

Приклад:

```sql
SELECT to_json(data) FROM mytable;
```