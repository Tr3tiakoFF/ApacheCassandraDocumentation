# Визначення схеми бази даних

Після того як ви завершили оцінку та вдосконалення фізичної моделі, ви готові реалізувати схему в CQL. Ось схема для простору ключів hotel, з використанням функції коментарів CQL для документування шаблону запитів, підтримуваного кожною таблицею:

```sql
CREATE KEYSPACE hotel WITH replication =
  {'class': 'SimpleStrategy', 'replication_factor' : 3};

CREATE TYPE hotel.address (
  street text,
  city text,
  state_or_province text,
  postal_code text,
  country text );

CREATE TABLE hotel.hotels_by_poi (
  poi_name text,
  hotel_id text,
  name text,
  phone text,
  address frozen<address>,
  PRIMARY KEY ((poi_name), hotel_id) )
  WITH comment = 'Q1. Знайти готелі поблизу заданого POI'
  AND CLUSTERING ORDER BY (hotel_id ASC) ;

CREATE TABLE hotel.hotels (
  id text PRIMARY KEY,
  name text,
  phone text,
  address frozen<address>,
  pois set<text> )
  WITH comment = 'Q2. Знайти інформацію про готель';

CREATE TABLE hotel.pois_by_hotel (
  poi_name text,
  hotel_id text,
  description text,
  PRIMARY KEY ((hotel_id), poi_name) )
  WITH comment = 'Q3. Знайти POI поблизу готелю';

CREATE TABLE hotel.available_rooms_by_hotel_date (
  hotel_id text,
  date date,
  room_number smallint,
  is_available boolean,
  PRIMARY KEY ((hotel_id), date, room_number) )
  WITH comment = 'Q4. Знайти доступні кімнати за готелем та датою';

CREATE TABLE hotel.amenities_by_room (
  hotel_id text,
  room_number smallint,
  amenity_name text,
  description text,
  PRIMARY KEY ((hotel_id, room_number), amenity_name) )
  WITH comment = 'Q5. Знайти зручності для кімнати';
```

Зверніть увагу, що елементи ключа партиції обгорнуті в дужки, навіть якщо ключ партиції складається з одного стовпця poi_name. Це є кращою практикою, яка робить вибір ключа партиції більш явним для тих, хто читає ваш CQL.

Аналогічно, ось схема для простору ключів reservation:

```sql
CREATE KEYSPACE reservation WITH replication = {'class':
  'SimpleStrategy', 'replication_factor' : 3};

CREATE TYPE reservation.address (
  street text,
  city text,
  state_or_province text,
  postal_code text,
  country text );

CREATE TABLE reservation.reservations_by_confirmation (
  confirm_number text,
  hotel_id text,
  start_date date,
  end_date date,
  room_number smallint,
  guest_id uuid,
  PRIMARY KEY (confirm_number) )
  WITH comment = 'Q6. Знайти бронювання за номером підтвердження';

CREATE TABLE reservation.reservations_by_hotel_date (
  hotel_id text,
  start_date date,
  end_date date,
  room_number smallint,
  confirm_number text,
  guest_id uuid,
  PRIMARY KEY ((hotel_id, start_date), room_number) )
  WITH comment = 'Q7. Знайти бронювання за готелем і датою';

CREATE TABLE reservation.reservations_by_guest (
  guest_last_name text,
  hotel_id text,
  start_date date,
  end_date date,
  room_number smallint,
  confirm_number text,
  guest_id uuid,
  PRIMARY KEY ((guest_last_name), hotel_id) )
  WITH comment = 'Q8. Знайти бронювання за прізвищем гостя';

CREATE TABLE reservation.guests (
  guest_id uuid PRIMARY KEY,
  first_name text,
  last_name text,
  title text,
  emails set,
  phone_numbers list,
  addresses map<text, frozen<address>>,
  confirm_number text )
  WITH comment = 'Q9. Знайти гостя за ID';
```

Тепер у вас є повна схема Cassandra для зберігання даних для додатка для готелів.