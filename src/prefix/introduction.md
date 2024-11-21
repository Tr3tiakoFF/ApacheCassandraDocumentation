# Вступне слово

Ця книга присвячена опису документації розподіленої NoSQL бази даних Apache Cassandra українською мовою.

Погляньте на [авторів книги](./../suffix/authors_contributors.md)!  
А також не соромтесь й відвідайте сторінку [GitHub проєкту](https://github.com/Tr3tiakoFF/ApacheCassandraDocumentation)!

## *Важливе уточнення*

*Всі дані відповідають дійсності на момент 12го листопада 2024р. відносно офіційної документації Apache Cassandra.*  
*Збраження, що наявні у книзі частково запозичені з офіційної документації Apache Cassandra.*

*Офіційна документація станом на 12те листопада 2024р. : <https://cassandra.apache.org/doc/4.1/index.html>*
https://github.com/Tr3tiakoFF/ApacheCassandraDocumentation
*Офіційна документація найновішої версії : <https://cassandra.apache.org/doc/latest>*

# Встановлення Cassandra як Debian-пакета

- Додавання Apache-репозиторію Cassandra
Додайте репозиторій Apache Cassandra до файлу cassandra.sources.list.
Остання основна версія позначена як {50_version}, а назва дистрибутиву — 50x. Для старіших версій використовуйте:
    - 50x для серії {50_version}
    - 41x для серії {41_version}
    - 40x для серії 4.0
Наприклад, щоб додати репозиторій для версії {50_version} (50x), виконайте:
```bash
echo "deb https://www.apache.org/dist/cassandra/debian 50x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
```
- Оновіть списки пакетів
Виконайте команду для оновлення репозиторіїв:
```bash
sudo apt update
```
- Встановіть Apache Cassandra
Після додавання репозиторію та оновлення списків пакетів встановіть Cassandra:
```bash
sudo apt install cassandra
```
- Перевірка встановлення
Після встановлення переконайтеся, що Cassandra працює, виконавши:
```bash
sudo systemctl status cassandra
```

Тепер Cassandra має бути встановлена і готова до використання! 