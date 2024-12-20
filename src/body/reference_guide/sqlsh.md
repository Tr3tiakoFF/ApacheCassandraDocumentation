# Інтерфейс командного рядка мови запитів Apache Cassandra

`cqlsh` — це командний інтерфейс для роботи з Cassandra, що використовує Cassandra Query Language (CQL). Він входить до складу кожного пакету Cassandra і розташовується в каталозі bin/ поряд із виконуваним файлом cassandra. `cqlsh` реалізований із використанням протоколу Python native protocol driver і підключається до вказаного вузла.

## Сумісність
Загалом кожна версія `cqlsh` гарантовано працює лише з тією версією Cassandra, з якою вона була випущена. У деяких випадках `cqlsh` може працювати зі старішими чи новішими версіями Cassandra, але це офіційно не підтримується.

## Необов’язкові залежності
`cqlsh` постачається з усіма необхідними залежностями. Проте є кілька необов’язкових бібліотек, які можна встановити для розширення можливостей `cqlsh`.

### pytz
За замовчуванням `cqlsh` відображає всі часові мітки у часовій зоні UTC.
Для Python 3.9 або новіше можна змінити часову зону, налаштувавши параметр `timezone` у файлі `cqlshrc` або через змінну середовища `TZ`. Для Python 3.8 або старіше також потрібно встановити бібліотеку `pytz`.

### cython
Продуктивність операцій `COPY` у cqlsh може бути покращена шляхом встановлення [cython](https://cython.org). Це дозволить компілювати модулі Python, які критично впливають на продуктивність `COPY`.

## cqlshrc
Файл `cqlshrc` містить конфігураційні параметри для `cqlsh`. За замовчуванням цей файл розташований у домашній директорії користувача за шляхом `~/.cassandra/cqlshrc`. Але можна вказати інше розташування за допомогою опції `--cqlshrc`.

Приклад конфігураційних параметрів та документація знаходяться у файлі `conf/cqlshrc.sample` у tarball-інсталяції. Також можна переглянути останню версію файлу [cqlshrc онлайн](https://github.com/apache/cassandra/blob/trunk/conf/cqlshrc.sample).

## Історія CQL
Усі виконані команди CQL записуються у файл історії. За замовчуванням історія CQL записується у файл `~/.cassandra/cql_history`.

Для зміни цього розташування можна встановити змінну середовища CQL_HISTORY на інший шлях, наприклад, `~/some/other/path/to/cqlsh_history`, де `cqlsh_history` — це ім'я файлу. Якщо батьківські каталоги до файлу історії відсутні, вони будуть створені автоматично.

Якщо історію не потрібно зберігати, встановіть значення `CQL_HISTORY` на `/dev/null`. Ця функція підтримується з версії Cassandra 4.1.

## Опції командного рядка
Використання:
`cqlsh.py [опції] [хост [порт]]`

CQL Shell для Apache Cassandra

Опції:
- `--version`
Показати номер версії та завершити роботу.

- `-h`, `--help`
Показати довідку та завершити роботу.

- `-C`, `--color`
Завжди використовувати кольоровий вивід.

- `--no-color`
Ніколи не використовувати кольоровий вивід.

- `--browser=BROWSER`
Вказати браузер для відображення довідки CQL, де BROWSER може бути одним із підтримуваних браузерів за посиланням docs.python.org. Приклад: шлях до браузера з параметром %s, наприклад: /usr/bin/google-chrome-stable %s.

- `--ssl`
Використовувати SSL.

- `-u USERNAME`, `--username=USERNAME`
Автентифікуватися як вказаний користувач.

- `-p PASSWORD`, `--password=PASSWORD`
Використовувати вказаний пароль для автентифікації.

- `-k KEYSPACE`, `--keyspace=KEYSPACE`
Підключитися до вказаного простору ключів.

- `-f FILE`, `--file=FILE`
Виконати команди з файлу та завершити роботу.

- `--debug`
Показати додаткову інформацію для налагодження.

- `--coverage`
Збирати дані покриття.

- `--encoding=ENCODING`
Вказати кодування для виводу (за замовчуванням: UTF-8).

- `--cqlshrc=CQLSHRC`
Вказати альтернативне розташування файлу конфігурації cqlshrc.

- `--credentials=CREDENTIALS`
Вказати альтернативне розташування файлу з обліковими даними.

- `--cqlversion=CQLVERSION`
Вибрати конкретну версію CQL. За замовчуванням використовується найновіша версія, що підтримується сервером.
Приклад: "3.0.3", "3.1.0".

- `--protocol-version=PROTOCOL_VERSION`
Вказати конкретну версію протоколу. Інакше клієнт вибере відповідну версію автоматично.

- `-e EXECUTE`, `--execute=EXECUTE`
Виконати вказаний запит та завершити роботу.

- `--connect-timeout=CONNECT_TIMEOUT`
Вказати час очікування з’єднання в секундах (за замовчуванням: 5 секунд).

- `--request-timeout=REQUEST_TIMEOUT`
Вказати час очікування виконання запиту в секундах (за замовчуванням: 10 секунд).

- `-t`, `--tty`
Примусово використовувати режим TTY (командний рядок).

- `-v`, `--v`
Вивести поточну версію cqlsh.