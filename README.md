### Queries SQL - plataforma de alquiler de habitaciones by William Hernández

Objetivo general

Implementar el modelo relacional de la clínica veterinaria en PostgreSQL y consultar la información con sentencias SQL.

Objetivos específicos de aprendizaje:

Convertir el DER a modelo relacional implementable.
Crear tablas con PK, FK, restricciones y tipos de datos adecuados.
Insertar datos consistentes respetando reglas de integridad.
Resolver consultas SQL: simple, filtro, join, group by y subconsulta.
Aplicar operaciones de actualización y eliminación de forma segura.
Entregable esperado

1. El script de sus consultas SQL (todo debe hacerse con lenguaje SQL, no de manera manual)
2. Respuesta escrita a las preguntas de análisis.

---

### Creación de base de datos: DB_Inmobiliaria
### Motor: PostgreSQL

## 1. Creacion Base de Datos:

```sql
CREATE DATABASE "InmobiliariaDB"
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_CTYPE = 'es-ES'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
```

## 2. Creación de tablas:

### Tablas independientes

```sql
CREATE TABLE country (
    id_country     SERIAL PRIMARY KEY,
    name_country  VARCHAR(100) NOT NULL
);

CREATE TABLE role (
    id_role    SERIAL PRIMARY KEY,
    name_role  VARCHAR(30) NOT NULL
);

CREATE TABLE pay_method (
    id_pay      SERIAL PRIMARY KEY,
    name_pay    VARCHAR(100) NOT NULL,
    type_pay    VARCHAR(50),
    status_pay  VARCHAR(20) DEFAULT 'active'
);

CREATE TABLE town (
    id_town        SERIAL PRIMARY KEY,
    name_town      VARCHAR(100) NOT NULL,
    province_town  VARCHAR(100)
);
```

## Tablas dependientes de Town (Municipio)

```sql
CREATE TABLE neighborhood (
    id_neighbor       SERIAL PRIMARY KEY,
    id_town_neighbor  INTEGER NOT NULL REFERENCES town(id_town),
    name_neighbor     VARCHAR(100) NOT NULL
);

CREATE TABLE edu_center (
    id_edu       SERIAL PRIMARY KEY,
    town_edu     INTEGER REFERENCES town(id_town),
    name_edu     VARCHAR(150) NOT NULL,
    address_edu  VARCHAR(200),
    type_edu     VARCHAR(50)
);
```

## Usuario (tabla central)
Nota: "user" es palabra reservada en PostgreSQL, se usa comillas dobles

```sql
CREATE TABLE "user" (
    id_user            SERIAL PRIMARY KEY,
    num_docu_user      VARCHAR(30) NOT NULL,
    type_docu_user     VARCHAR(20),
    name_user          VARCHAR(100) NOT NULL,
    lastname_user      VARCHAR(100) NOT NULL,
    birthyear_user     INTEGER,
    gender_user        VARCHAR(20),
    mail_user          VARCHAR(150) NOT NULL UNIQUE,
    phone_user         VARCHAR(20),
    country_resi_user  INTEGER REFERENCES country(id_country),
    status_user        VARCHAR(20) DEFAULT 'active'
);
```

## Preference: relación 1 a 1 con User (PK = FK)

```sql
CREATE TABLE preference (
    id_user_pref          INTEGER PRIMARY KEY REFERENCES "user"(id_user),
    pet_owner_pref        BOOLEAN DEFAULT FALSE,
    smoker_pref           BOOLEAN DEFAULT FALSE,
    aircon_pref           BOOLEAN DEFAULT FALSE,
    wifi_pref             BOOLEAN DEFAULT FALSE,
    private_bathroom_pref BOOLEAN DEFAULT FALSE,
    closet_pref           BOOLEAN DEFAULT FALSE,
    kitchen_pref          BOOLEAN DEFAULT FALSE,
    balcony_pref          BOOLEAN DEFAULT FALSE,
    visit_allow_pref      BOOLEAN DEFAULT FALSE,
    utilities_incl_pref   BOOLEAN DEFAULT FALSE
);
```

## Role_User: Tabla pivote con relación Muchos a Muchos entre User y Role

```sql
CREATE TABLE roleuser (
    id_user_roleuser  INTEGER NOT NULL REFERENCES "user"(id_user),
    id_role_roleuser  INTEGER NOT NULL REFERENCES role(id_role),
    PRIMARY KEY (id_user_roleuser, id_role_roleuser)
);
```

## Enrollment (Matricula): Relación Muchos a Muchos entre User y Edu_Center

```sql
CREATE TABLE enrollment (
    id_enroll            SERIAL PRIMARY KEY,
    id_user_enroll       INTEGER NOT NULL REFERENCES "user"(id_user),
    id_educenter_enroll  INTEGER NOT NULL REFERENCES edu_center(id_edu),
    attach_enroll        VARCHAR(255),
    exp_date_enroll      DATE
);
```

## Room (Habitación)

```sql
CREATE TABLE room (
    id_room                SERIAL PRIMARY KEY,
    id_owner_room          INTEGER NOT NULL REFERENCES "user"(id_user),
    address_room           VARCHAR(200) NOT NULL,
    postalcode_room        VARCHAR(20),
    floor_number_room      INTEGER,
    town_room              INTEGER REFERENCES town(id_town),
    neighborhood_room      INTEGER REFERENCES neighborhood(id_neighbor),
    area_room              NUMERIC(6,2),
    bed_qty_room           INTEGER,
    capacity_room          INTEGER,
    allowed_gender_room    VARCHAR(10) CHECK (allowed_gender_room IN ('Male', 'Female', 'Any')),
    closet_room            BOOLEAN DEFAULT FALSE,
    private_bathroom_room  BOOLEAN DEFAULT FALSE,
    shared_bathroom_room   BOOLEAN DEFAULT FALSE,
    balcony_room           BOOLEAN DEFAULT FALSE,
    aircon_room            BOOLEAN DEFAULT FALSE,
    wifi_room              BOOLEAN DEFAULT FALSE,
    allowed_kitchen_room   BOOLEAN DEFAULT FALSE,
    visit_allowed_room     BOOLEAN DEFAULT FALSE,
    smoker_room            BOOLEAN DEFAULT FALSE,
    pet_allowed_room       BOOLEAN DEFAULT FALSE,
    utilities_incl_room    BOOLEAN DEFAULT FALSE,
    status_room            VARCHAR(20) DEFAULT 'available'
);
```

## Post (Publicación)

```sql
CREATE TABLE post (
    id_post              SERIAL PRIMARY KEY,
    id_publisher_post    INTEGER NOT NULL REFERENCES "user"(id_user),
    id_room_post         INTEGER NOT NULL REFERENCES room(id_room),
    timestamp_post       TIMESTAMPTZ NOT NULL DEFAULT now(),
    minimum_months_post  INTEGER CHECK (minimum_months_post > 0),
    monthly_price_post   NUMERIC(10,2) NOT NULL CHECK (monthly_price_post >= 0),
    deposit_price_post   NUMERIC(10,2) CHECK (deposit_price_post >= 0),
    status_post          VARCHAR(10) DEFAULT 'active' CHECK (status_post IN ('active', 'paused', 'closed', 'expired'))
);
```

## Booking (Reserva)

```sql
CREATE TABLE booking (
    id_booking           SERIAL PRIMARY KEY,
    id_user_booking      INTEGER NOT NULL REFERENCES "user"(id_user),
    id_post_booking      INTEGER NOT NULL REFERENCES post(id_post),
    datetime_booking     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    pay_method_booking   INTEGER REFERENCES pay_method(id_pay),
    start_date_booking   DATE NOT NULL,
    end_date_booking     DATE NOT NULL,
    pay_confirm_booking  BOOLEAN DEFAULT FALSE,
    status_booking       VARCHAR(20) DEFAULT 'pending' CHECK (status_booking IN ('pending', 'confirmed', 'cancelled', 'completed')),
    CONSTRAINT chk_dates CHECK (end_date_booking > start_date_booking)
);
```

## User_Review y Room_Review (dependen de Booking)

```sql
CREATE TABLE user_review (
    id_userreview           SERIAL PRIMARY KEY,
    id_booking_userreview   INTEGER NOT NULL REFERENCES booking(id_booking),
    id_reviewed_user        INTEGER NOT NULL REFERENCES "user"(id_user),
    rate_userreview         NUMERIC(2,1) CHECK (rate_userreview BETWEEN 1.0 AND 5.0),
    desc_userreview         TEXT,
    author_userreview       INTEGER NOT NULL REFERENCES "user"(id_user)
);

CREATE TABLE room_review (
    id_roomreview           SERIAL PRIMARY KEY,
    id_booking_roomreview   INTEGER NOT NULL REFERENCES booking(id_booking),
    rate_room_roomreview    NUMERIC(2,1) CHECK (rate_room_roomreview BETWEEN 1.0 AND 5.0),
    rate_owner_roomreview   NUMERIC(2,1) CHECK (rate_owner_roomreview BETWEEN 1.0 AND 5.0),
    desc_roomreview         TEXT,
    author_roomreview       INTEGER NOT NULL REFERENCES "user"(id_user)
);
```

## Índices (Se agregan para acelerar la búsqueda y recuperación de datos en una tabla)

```sql  
CREATE INDEX idx_room_owner ON room(id_owner_room);
CREATE INDEX idx_room_town ON room(town_room);
CREATE INDEX idx_post_room ON post(id_room_post);
CREATE INDEX idx_post_publisher ON post(id_publisher_post);
CREATE INDEX idx_booking_user ON booking(id_user_booking);
CREATE INDEX idx_booking_post ON booking(id_post_booking);
CREATE INDEX idx_enrollment_user ON enrollment(id_user_enroll);
CREATE INDEX idx_enrollment_edu ON enrollment(id_educenter_enroll);
CREATE INDEX idx_roleuser_user ON roleuser(id_user_roleuser);
CREATE INDEX idx_roleuser_role ON roleuser(id_role_roleuser);
```