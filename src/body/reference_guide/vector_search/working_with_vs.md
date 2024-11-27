# Робота з векторним пошуком

## Створення простору ключів для векторів

Створіть простір ключів, який буде використовуватись для таблиці пошуку векторів. У цьому прикладі використовується назва простору ключів `cycling`:

```sql
CREATE KEYSPACE IF NOT EXISTS cycling
   WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : '1' };
```
## Використання простору ключів для векторів
Виберіть простір ключів, який буде використовуватись для таблиці пошуку векторів. У цьому прикладі це `cycling`:

```sql
USE cycling;
```

## Створення таблиці для векторів
Створіть нову таблицю у просторі ключів, включивши стовпець `comments_vector` для зберігання векторів. Код нижче створює вектор з п’яти значень:

```sql
CREATE TABLE IF NOT EXISTS cycling.comments_vs (
  record_id timeuuid,
  id uuid,
  commenter text,
  comment text,
  comment_vector VECTOR <FLOAT, 5>,
  created_at timestamp,
  PRIMARY KEY (id, created_at)
)
WITH CLUSTERING ORDER BY (created_at DESC);
```
За бажанням можна змінити існуючу таблицю, щоб додати стовпець для зберігання векторів:

```sql
ALTER TABLE cycling.comments_vs
   ADD comment_vector VECTOR <FLOAT, 5>
```

## Створення індексу для векторів
Створіть спеціальний індекс за допомогою Storage Attached Indexing (SAI):

```sql
CREATE INDEX IF NOT EXISTS ann_index
  ON cycling.comments_vs(comment_vector) USING 'sai';
```

Для більш детальної інформації про SAI перегляньте документацію про Storage Attached Indexing.

*Індекс можна створити з опціями, які визначають функцію подібності:*
```sql
CREATE INDEX IF NOT EXISTS ann_index
    ON vsearch.com(item_vector) USING 'sai'
WITH OPTIONS = { 'similarity_function': 'DOT_PRODUCT' };
```
*Допустимі значення для функції подібності: `DOT_PRODUCT`, `COSINE` або `EUCLIDEAN`.*

## Завантаження векторних даних у базу
Вставте дані у таблицю, використовуючи новий тип:

```sql
INSERT INTO cycling.comments_vs (record_id, id, created_at, comment, commenter, comment_vector)
   VALUES (
      now(),
      e7ae5cf3-d358-4d99-b900-85902fda9bb0,
      '2017-02-14 12:43:20-0800',
      'Raining too hard should have postponed',
      'Alex',
      [0.45, 0.09, 0.01, 0.2, 0.11]
);
```
Додаткові приклади вставки:

```sql
INSERT INTO cycling.comments_vs (record_id, id, created_at, comment, commenter, comment_vector)
   VALUES (
      now(),
      c7fceba0-c141-4207-9494-a29f9809de6f,
      '2017-03-22 5:16:59.001+0400',
      'Great snacks at all reststops',
      'Amy',
      [0.1, 0.4, 0.1, 0.52, 0.09]
);
```
## Запити до векторних даних за допомогою CQL
Для виконання пошуку векторів використовуйте запит SELECT:
```sql
SELECT * FROM cycling.comments_vs
    ORDER BY comment_vector ANN OF [0.15, 0.1, 0.1, 0.35, 0.55]
    LIMIT 3;
```
Щоб отримати результат із розрахунком подібності:
```sql
SELECT comment, similarity_cosine(comment_vector, [0.2, 0.15, 0.3, 0.2, 0.05])
    FROM cycling.comments_vs
    ORDER BY comment_vector ANN OF [0.1, 0.15, 0.3, 0.12, 0.05]
    LIMIT 1;
```
Підтримувані функції для цього типу запитів:

- `similarity_dot_product`
- `similarity_cosine`
- `similarity_euclidean`
Параметри: `<vector_column>`, `<embedding_value>`. Обидва параметри представляють вектори.

*Ліміт повинен бути не більше 1,000. Пошук векторів використовує метод Approximate Nearest Neighbor (ANN), який у більшості випадків забезпечує результати, близькі до точного збігу. Масштабованість вищає порівняно з Exact Nearest Neighbor (KNN). Пошук найменш подібних векторів не підтримується. Оптимальна продуктивність досягається у таблицях без змін чи видалень у стовпці item_vector. У разі змін можливе уповільнення пошуку.*