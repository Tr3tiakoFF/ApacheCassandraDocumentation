# CAP та гарантії

Apache Cassandra — це високомасштабована та надійна база даних, яку використовують для веб-додатків, що обслуговують велику кількість клієнтів і оперують великими обсягами даних на рівні веб-масштабів (петабайти даних). Cassandra надає певні гарантії щодо своєї масштабованості, доступності та надійності. Щоб повністю зрозуміти обмеження системи зберігання даних в умовах, де очікуються певні збої мережевого розподілу, і врахувати це при проєктуванні системи, важливо спочатку коротко розглянути теорему CAP.

## Що таке CAP?

Теорема CAP (Consistency, Availability, Partition tolerance) стверджує, що розподілена система не може забезпечити всі три характеристики одночасно:

- Консистентність (Consistency) — всі вузли мають однакові дані після виконання операції запису. Кожен запит на читання отримує найновіший запис або помилку.
- Доступність (Availability) — кожен запит до системи завершується відповіддю (успішною або невдалою).
- Толерантність до розподілу (Partition tolerance) — система продовжує працювати навіть у випадку розподілу мережі, який розриває комунікацію між частинами системи.

Теорема CAP вказує на те, що в умовах мережевого розподілу, при ризику виникнення розподілу, потрібно вибирати між послідовністю та доступністю, оскільки обидва гаранти не можуть бути забезпечені одночасно.

Висока доступність є пріоритетом для веб-додатків, тому Cassandra вибирає доступність та терпимість до розподілу з гарантів теореми CAP, поступаючись частково даними послідовності.Розробники Cassandra зробили свідомий вибір забезпечити доступність і толерантність до розподілу за рахунок певної консистентності. Це означає, що Cassandra підтримує високу доступність і стійкість до збоїв, навіть у розподіленому середовищі з можливими мережевими розривами.

Cassandra надає наступні гарантії:

- Висока масштабованість
- Висока доступність
- Довговічність
- Кінцева послідовність записів в одній таблиці
- Легкі транзакції з лінійною послідовністю
- Пакетні записи через кілька таблиць гарантовано виконуються повністю або зовсім не виконуються
- Вторинні індекси гарантують узгодженість із даними їхніх локальних реплік.

### Висока масштабованість
Cassandra є високома scalesованою системою зберігання даних, у якій вузли можуть додаватися або видалятися за потребою. Використовуючи протокол на основі gossip, єдина та послідовна інформація про членів зберігається на кожному вузлі.

### Висока доступність
Cassandra гарантує високу доступність даних, реалізуючи систему зберігання, стійку до відмов. Виявлення відмови вузла здійснюється за допомогою протоколу на основі gossip.

### Довговічність
Cassandra гарантує довговічність даних, використовуючи репліки. Репліки — це кілька копій даних, збережених на різних вузлах у кластері. У середовищі з кількома дата-центрами репліки можуть бути збережені в різних дата-центрах. Якщо одна репліка втрачається через незворотну відмову вузла або дата-центру, дані не втрачаються повністю, оскільки репліки залишаються доступними.

### Кінцева послідовність
Для досягнення вимог до продуктивності, надійності, масштабованості та високої доступності в умовах виробництва Cassandra є сховищем з кінцевою послідовністю. Кінцева послідовність означає, що всі оновлення зрештою досягнуть усіх реплік. Тимчасово можуть існувати різні версії тих самих даних, але вони в кінцевому підсумку приводяться до консистентного стану. Кінцева послідовність — це компроміс для досягнення високої доступності, що включає певні затримки на читання та запис.

### Легкі транзакції з лінійною послідовністю
Дані повинні читатися та записуватися в послідовному порядку. Для реалізації легких транзакцій використовується консенсусний протокол Paxos. Протокол Paxos реалізує легкі транзакції, здатні обробляти одночасні операції за допомогою лінійної послідовності. Лінійна послідовність — це послідовність з реальними часовими обмеженнями, що забезпечує ізоляцію транзакцій за допомогою операцій порівняння та встановлення значення (CAS). За допомогою CAS репліковані дані порівнюються, і дані, що виявилися застарілими, встановлюються на найбільш консистентне значення. Читання з лінійною послідовністю дозволяє читати поточний стан даних, який може бути незатвердженим, без додавання або оновлення.

### Пакетні записи
Гарантія для пакетних записів через кілька таблиць полягає в тому, що вони зрештою успішно виконуються, або жоден запис не буде виконано. Пакетні дані спочатку записуються в систему даних batchlog, і коли пакетні дані успішно зберігаються в кластері, дані batchlog видаляються. Пакет реплікується на інший вузол, щоб забезпечити завершення всього пакету у разі відмови координуючого вузла.

### Вторинні індекси
Вторинний індекс — це індекс на стовпці, і він використовується для запиту таблиці, яку зазвичай не можна запитувати. Вторинні індекси, коли вони будуються, гарантують узгодженість з їхніми локальними репліками.