# Арифметичні оператори

CQL підтримує наступні оператори:

|Оператор | Опис |
|------|---------|
| - (унарний) |	Негує операнд |
| + |	Додавання |
| - |	Віднімання |
| * |	Множення |
| / |	Ділення |
| % |	Повертає залишок від ділення |

## Арифметика чисел
Всі арифметичні операції підтримуються для числових типів і лічильників (`counter`).

Результат операції залежить від типів операндів:

|   Лівий/Правий   |	`tinyint` |	`smallint`    |	`int`     |	`bigint`  |	`counter` |	`float`   |	`double`  |	`varint`  |	`decimal` |
|------------------|--------------|---------------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|
|   `tinyint`    |	tinyint     |	smallint    |	int     |	bigint  |	bigint  |	float   |	double  |	varint  |	decimal |
|   `smallint`   |	smallint    |	smallint    |	int     |	bigint  |	bigint  |	float   |	double  |	varint  |	decimal |
|   `int`        |	int         |	int         |	int     |	bigint  |	bigint  |	float   |	double  |	varint  |	decimal |
|   `bigint`     |	bigint      |	bigint      |	bigint  |	bigint  |	bigint  |	double  |	double  |	varint  |	decimal |
|   `counter`    |	bigint      |	bigint      |	bigint  |	bigint  |	bigint  |	double  |	double  |	varint  |	decimal |
|   `float`      |	float       |	float       |	float   |	double  |	double  |	float   |	double  |	decimal |	decimal |
|   `double`     |	double      |	double      |	double  |	double  |	double  |	double  |	double  |	decimal |	decimal |
|   `varint`     |	varint      |	varint      |	varint  |	varint  |	decimal |	decimal |	decimal |	decimal |	decimal |
|   `decimal`    |	decimal     |	decimal     |	decimal |	decimal |	decimal |	decimal |	decimal |	decimal |	decimal |

## Пріоритет операторів

Оператори `*`, `/` і `%` мають вищий рівень пріоритету, ніж `+` і `-`. Відповідно, вони обчислюються першими. Якщо в одному виразі є кілька операторів з однаковим пріоритетом, обчислення виконується зліва направо відповідно до їхнього розташування.

## Арифметика дат і часу
До `timestamp` або `date` можна додавати (`+`) чи віднімати (`-`) тривалість (`duration`), щоб отримати нове значення `timestamp` або `date`.

#### Наприклад:

```sql
SELECT * FROM myTable WHERE t = '2017-01-01' - 2d;
```
Цей запит вибере всі записи, де значення t відповідає останнім двом дням 2016 року.