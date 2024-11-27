# Тригери

Тригери дозволяють визначати користувацьку логіку, яка автоматично виконується перед запитом DML для таблиці.

Ім'я тригера
Тригер ідентифікується за його назвою:

```bnf
trigger_name ::= identifier
```

## Створення тригеру

Команда `CREATE TRIGGER` використовується для створення нового тригера.

Синтаксис:

```bnf
create_trigger_statement ::= CREATE TRIGGER [ IF NOT EXISTS ] trigger_name
	ON table_name
	USING string
```
Приклад:

```sql
CREATE TRIGGER myTrigger ON myTable USING 'org.apache.cassandra.triggers.InvertedIndex';
```
### Логіка тригера:
Логіка для тригера реалізується на будь-якій мові JVM (наприклад, Java).

Код тригера розміщується в підкаталозі lib/triggers у директорії встановлення Cassandra.
Логіка завантажується під час запуску кластера і має бути доступною на кожному вузлі в кластері.

### Виконання тригера:
Тригери виконуються перед виконанням DML-запиту, забезпечуючи атомарність транзакції.

## Видалення тригеру
Команда `DROP TRIGGER` використовується для видалення тригера.

Синтаксис:

```bnf
drop_trigger_statement ::= DROP TRIGGER [ IF EXISTS ] trigger_nameON table_name
```

Приклад:

```sql
DROP TRIGGER myTrigger ON myTable;
```