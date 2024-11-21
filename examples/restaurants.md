CREATE KEYSPACE restaurants
    WITH REPLICATION = { 
      'class' : 'SimpleStrategy', 
      'replication_factor' : 1 
    } AND durable_writes = true;

USE restaurants;

CREATE TABLE Owners (
	first_name TEXT,
	middle_name TEXT,
	last_name TEXT,
	
	passport TEXT PRIMARY KEY,
	
	birth_date DATE
);

CREATE TABLE Owners_dynasty (
    id UUID PRIMARY KEY,

	child_passport TEXT,

	child_first_name TEXT,
	child_middle_name TEXT,
	child_last_name TEXT,
    child_birth_date DATE,
	
	father_passport TEXT,

	father_first_name TEXT,
	father_middle_name TEXT,
	father_last_name TEXT,
    father_birth_date DATE,

	mother_passport TEXT,

	mother_first_name TEXT,
	mother_middle_name TEXT,
	mother_last_name TEXT,
	mother_birth_date DATE
);

CREATE TABLE Restaurants (
    id UUID PRIMARY KEY,

	name TEXT,
	adress TEXT,
	open_date DATE
);

CREATE TABLE Restaurants_Owners (
    id UUID PRIMARY KEY,

	name TEXT,
	adress TEXT,
	open_date DATE,

    owner_passport TEXT,

	owner_first_name TEXT,
	owner_middle_name TEXT,
	owner_last_name TEXT,
    owner_birth_date DATE
);