## SQL Queries - Plataforma Web de alquiler de habitaciones por William Hernández

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

#### Tablas independientes

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

### Tablas dependientes de Town (Municipio)

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

### Usuario (tabla central)
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

### Preference: relación 1 a 1 con User (PK = FK)

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

### Role_User: Tabla pivote con relación Muchos a Muchos entre User y Role

```sql
CREATE TABLE roleuser (
    id_user_roleuser  INTEGER NOT NULL REFERENCES "user"(id_user),
    id_role_roleuser  INTEGER NOT NULL REFERENCES role(id_role),
    PRIMARY KEY (id_user_roleuser, id_role_roleuser)
);
```

### Enrollment (Matricula): Relación Muchos a Muchos entre User y Edu_Center

```sql
CREATE TABLE enrollment (
    id_enroll            SERIAL PRIMARY KEY,
    id_user_enroll       INTEGER NOT NULL REFERENCES "user"(id_user),
    id_educenter_enroll  INTEGER NOT NULL REFERENCES edu_center(id_edu),
    attach_enroll        VARCHAR(255),
    exp_date_enroll      DATE
);
```

### Room (Habitación)

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

### Post (Publicación)

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

### Booking (Reserva)

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

### User_Review y Room_Review (dependen de Booking)

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

### Índices
Se agregan para acelerar la búsqueda y recuperación de datos en una tabla

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

---

## Insert de registros


### 1. TABLAS INDEPENDIENTES Y CATÁLOGOS (10 registros c/u)

```sql
INSERT INTO country (name_country) VALUES
('España'), ('Colombia'), ('México'), ('Argentina'), ('Chile'),
('Perú'), ('Ecuador'), ('Uruguay'), ('Bolivia'), ('Paraguay');

INSERT INTO role (name_role) VALUES
('Owner'), ('Student'), ('Admin'), ('Agent'), ('Maintenance'),
('Guest'), ('Moderator'), ('Auditor'), ('Support'), ('Supervisor');

INSERT INTO pay_method (name_pay, type_pay, status_pay) VALUES
('Tarjeta de Crédito Visa', 'Card', 'active'),
('Tarjeta de Débito Mastercard', 'Card', 'active'),
('Transferencia Bancaria BBVA', 'Transfer', 'active'),
('Bizum', 'Mobile', 'active'),
('PayPal', 'Digital Wallet', 'active'),
('Efectivo', 'Cash', 'active'),
('Stripe', 'Digital Wallet', 'active'),
('Domiciliación SEPA', 'Direct Debit', 'active'),
('Criptomoneda (USDT)', 'Crypto', 'inactive'),
('Cheque Bancario', 'Check', 'inactive');

INSERT INTO town (name_town, province_town) VALUES
('Madrid', 'Madrid'),
('Barcelona', 'Barcelona'),
('Valencia', 'Valencia'),
('Sevilla', 'Sevilla'),
('Alicante', 'Alicante'),
('Granada', 'Granada'),
('Salamanca', 'Salamanca'),
('Zaragoza', 'Zaragoza'),
('Málaga', 'Málaga'),
('Murcia', 'Murcia');
```

### 2. TABLAS DEPENDIENTES DE TOWN (10 registros c/u)

```sql
INSERT INTO neighborhood (id_town_neighbor, name_neighbor) VALUES
(1, 'Malasaña'),
(2, 'Eixample'),
(3, 'Ruzafa'),
(4, 'Triana'),
(5, 'Centro'),
(6, 'Albaicín'),
(7, 'Van Dyck'),
(8, 'Delicias'),
(9, 'Teatinos'),
(10, 'El Carmen');

INSERT INTO edu_center (town_edu, name_edu, address_edu, type_edu) VALUES
(1, 'Universidad Complutense de Madrid', 'Av. Séneca 2', 'Universidad Publica'),
(2, 'Universitat de Barcelona', 'Gran Via de les Corts Catalanes 585', 'Universidad Publica'),
(3, 'Universitat Politècnica de València', 'Camí de Vera s/n', 'Universidad Publica'),
(4, 'Universidad de Sevilla', 'Calle San Fernando 4', 'Universidad Publica'),
(5, 'Universidad de Alicante', 'Carretera de San Vicente s/n', 'Universidad Publica'),
(6, 'Universidad de Granada', 'Av. del Hospicio s/n', 'Universidad Publica'),
(7, 'Universidad de Salamanca', 'Patio de Escuelas 1', 'Universidad Publica'),
(8, 'Universidad de Zaragoza', 'Calle Pedro Cerbuna 12', 'Universidad Publica'),
(9, 'Universidad de Málaga', 'Av. Cervantes 2', 'Universidad Publica'),
(10, 'Universidad de Murcia', 'Av. Teniente Flomesta 5', 'Universidad Publica');
```

### 3. USUARIOS Y SU ENTORNO (10 registros c/u)

```sql

INSERT INTO "user" (num_docu_user, type_docu_user, name_user, lastname_user, birthyear_user, gender_user, mail_user, phone_user, country_resi_user, status_user) VALUES
('12345678A','DNI','Carlos','Gómez',1978,'Male','carlos.gomez@mail.com','+34600111222',1,'active'),
('23456789B','DNI','Ana','Martínez',1985,'Female','ana.martinez@mail.com','+34600222333',1,'active'),
('34567890C','DNI','Pedro','Sánchez',1980,'Male','pedro.sanchez@mail.com','+34600333444',1,'active'),
('45678901D','NIE','Laura','López',2001,'Female','laura.lopez@student.com','+34600444555',2,'active'),
('56789012E','DNI','Mateo','Fernández',2002,'Male','mateo.fernandez@student.com','+34600555666',1,'active'),
('67890123F','Passport','Sofia','Rossi',2000,'Female','sofia.rossi@student.com','+34600666777',3,'active'),
('78901234G','DNI','David','García',1999,'Male','david.garcia@student.com','+34600777888',1,'active'),
('89012345H','NIE','Elena','Torres',2003,'Female','elena.torres@student.com','+34600888999',4,'active'),
('90123456I','DNI','Javier','Ruiz',2001,'Male','javier.ruiz@student.com','+34600999000',1,'active'),
('01234567J','Passport','Lucía','Benítez',2002,'Female','lucia.benitez@student.com','+34600000111',5,'active'),
('55573366W','DNI','William','Hernández',1986,'Male','william_dev@gmail.com','+34614555668',2,'active');

INSERT INTO preference (id_user_pref, pet_owner_pref, smoker_pref, aircon_pref, wifi_pref, private_bathroom_pref, closet_pref, kitchen_pref, balcony_pref, visit_allow_pref, utilities_incl_pref) VALUES
(1, false, false, true,  true,  true,  true,  true,  true,  true,  true),
(2, true,  false, true,  true,  true,  true,  true,  false, true,  false),
(3, false, true,  false, true,  false, true,  true,  true,  false, true),
(4, false, false, true,  true,  true,  true,  true,  true,  true,  true),
(5, true,  false, false, true,  false, true,  true,  false, true,  true),
(6, false, false, true,  true,  true,  true,  true,  true,  true,  true),
(7, false, true,  true,  true,  false, true,  false, true,  false, false),
(8, true,  false, true,  true,  true,  true,  true,  false, true,  true),
(9, false, false, false, true,  false, true,  true,  true,  true,  false),
(10,false, false, true,  true,  true,  true,  true,  true,  true,  true);

INSERT INTO roleuser (id_user_roleuser, id_role_roleuser) VALUES
(1, 1), -- Carlos: Owner
(2, 1), -- Ana: Owner
(3, 1), -- Pedro: Owner
(4, 2), -- Laura: Student
(5, 2), -- Mateo: Student
(6, 2), -- Sofia: Student
(7, 2), -- David: Student
(8, 2), -- Elena: Student
(9, 2), -- Javier: Student
(10, 2), -- Lucía: Student
(11, 3); -- William: Agent

INSERT INTO enrollment (id_user_enroll, id_educenter_enroll, attach_enroll, exp_date_enroll) VALUES
(4, 1, 'enroll_laura_ucm.pdf', '2025-06-30'),
(5, 2, 'enroll_mateo_ub.pdf', '2025-06-30'),
(6, 3, 'enroll_sofia_upv.pdf', '2025-06-30'),
(7, 4, 'enroll_david_us.pdf', '2025-06-30'),
(8, 5, 'enroll_elena_ua.pdf', '2025-06-30'),
(9, 6, 'enroll_javier_ugr.pdf', '2025-06-30'),
(10, 7, 'enroll_lucia_usal.pdf', '2025-06-30'),
(4, 8, 'enroll_laura_unizar.pdf', '2026-06-30'),
(5, 9, 'enroll_mateo_uma.pdf', '2026-06-30'),
(6, 10, 'enroll_sofia_um.pdf', '2026-06-30');
```

### 4. HABITACIONES Y PUBLICACIONES (10 registros c/u)

```sql
INSERT INTO room (id_owner_room, address_room, postalcode_room, floor_number_room, town_room, neighborhood_room, area_room, bed_qty_room, capacity_room, allowed_gender_room, closet_room, private_bathroom_room, shared_bathroom_room, balcony_room, aircon_room, wifi_room, allowed_kitchen_room, visit_allowed_room, smoker_room, pet_allowed_room, utilities_incl_room, status_room) VALUES
(1, 'Calle Pez 12, 3A', '28004', 3, 1, 1, 18.50, 1, 1, 'Any',    true, true,  false, true,  true,  true, true, true,  false, false, true,  'available'),
(1, 'Calle Pez 12, 3B', '28004', 3, 1, 1, 15.00, 1, 1, 'Female', true, false, true,  false, true,  true, true, true,  false, false, true,  'available'),
(2, 'Carrer de Mallorca 234', '08008', 2, 2, 2, 22.00, 2, 2, 'Any',    true, true,  false, true,  true,  true, true, true,  false, true,  false, 'available'),
(2, 'Carrer de Mallorca 236', '08008', 4, 2, 2, 12.00, 1, 1, 'Male',   true, false, true,  false, false, true, true, false, true,  false, false, 'available'),
(3, 'Carrer de Cadis 15', '46006', 1, 3, 3, 20.00, 1, 1, 'Female', true, true,  false, true,  true,  true, true, true,  false, false, true,  'available'),
(3, 'Calle Betis 45', '41010', 2, 4, 4, 16.50, 1, 1, 'Any',    true, false, true,  true,  false, true, true, true,  false, false, true,  'available'),
(1, 'Av. Alfonso El Sabio 10', '03002', 5, 5, 5, 25.00, 2, 2, 'Any',    true, true,  false, true,  true,  true, true, true,  false, true,  true,  'available'),
(2, 'Carrera del Darro 8', '18010', 1, 6, 6, 14.00, 1, 1, 'Female', true, false, true,  false, false, true, true, true,  false, false, false, 'available'),
(3, 'Calle Toro 50', '37002', 3, 7, 7, 19.00, 1, 1, 'Male',   true, true,  false, true,  true,  true, true, false, false, false, true,  'available'),
(1, 'Paseo Independencia 18', '50004', 4, 8, 8, 17.50, 1, 1, 'Any',    true, false, true,  false, true,  true, true, true,  false, false, true,  'available');

-- Nota: La Habitación 1 tiene 2 publicaciones (id_post 1 cerrada y id_post 2 activa)
INSERT INTO post (id_publisher_post, id_room_post, timestamp_post, minimum_months_post, monthly_price_post, deposit_price_post, status_post) VALUES
(1, 1, '2023-09-01 10:00:00+01', 5, 450.00, 450.00, 'closed'),  -- Histórico (Habitación 1)
(1, 1, '2024-01-15 09:30:00+01', 6, 480.00, 480.00, 'active'),  -- Actual (Habitación 1)
(1, 2, '2024-01-16 11:00:00+01', 6, 400.00, 400.00, 'active'),
(2, 3, '2024-01-20 12:15:00+01', 10,550.00, 1100.00,'active'),
(2, 4, '2024-01-22 15:45:00+01', 3, 350.00, 350.00, 'active'),
(3, 5, '2024-02-01 08:00:00+01', 6, 420.00, 420.00, 'active'),
(3, 6, '2024-02-05 16:20:00+01', 5, 380.00, 380.00, 'active'),
(1, 7, '2024-02-10 18:00:00+01', 12,600.00, 600.00, 'active'),
(2, 8, '2024-02-12 14:10:00+01', 6, 390.00, 390.00, 'active'),
(3, 9, '2024-02-15 19:30:00+01', 9, 410.00, 410.00, 'paused');
```

### 5. RESERVAS Y RESEÑAS (10 registros c/u)

-- Reservas hechas únicamente por usuarios con rol 'Student' (id_user 4 al 10)
```sql
INSERT INTO booking (id_user_booking, id_post_booking, datetime_booking, pay_method_booking, start_date_booking, end_date_booking, pay_confirm_booking, status_booking) VALUES
(4, 1,  '2023-09-02 11:00:00+01', 1, '2023-09-15', '2024-01-15', true,  'completed'),
(5, 2,  '2024-01-18 10:30:00+01', 4, '2024-02-01', '2024-07-31', true,  'confirmed'),
(6, 3,  '2024-01-19 14:20:00+01', 2, '2024-02-01', '2024-07-31', true,  'confirmed'),
(7, 4,  '2024-01-25 09:00:00+01', 3, '2024-03-01', '2024-12-31', true,  'confirmed'),
(8, 5,  '2024-01-26 17:40:00+01', 4, '2024-02-15', '2024-05-15', true,  'confirmed'),
(9, 6,  '2024-02-02 12:00:00+01', 5, '2024-03-01', '2024-08-31', true,  'confirmed'),
(10, 7, '2024-02-06 18:30:00+01', 1, '2024-03-01', '2024-07-31', true,  'pending'),
(4, 8,  '2024-02-11 08:45:00+01', 4, '2024-09-01', '2025-08-31', true,  'confirmed'),
(5, 9,  '2024-02-14 13:15:00+01', 3, '2024-03-01', '2024-08-31', false, 'cancelled'),
(6, 10, '2024-02-16 16:00:00+01', 2, '2024-04-01', '2024-12-31', false, 'pending');

-- Reseñas entre Usuarios (Estudiantes evalúan a Propietarios y viceversa)
INSERT INTO user_review (id_booking_userreview, id_reviewed_user, rate_userreview, desc_userreview, author_userreview) VALUES
(1, 1, 5.0, 'Carlos fue un excelente propietario, siempre atento a todo.', 4),
(1, 4, 4.8, 'Laura cuidó la habitación perfectamente durante su estancia.', 1),
(2, 1, 4.5, 'Todo perfecto en la entrega de llaves y comunicación.', 5),
(3, 1, 4.0, 'La estancia fue muy agradable y rápida gestión.', 6),
(4, 2, 5.0, 'Ana es muy amable y comprensiva con las fechas.', 7),
(5, 2, 4.2, 'Trato cercano y formal por parte del arrendador.', 8),
(6, 3, 4.9, 'Pedro resolvió una pequeña duda con el agua al instante.', 9),
(7, 1, 3.8, 'Proceso correcto aunque tardó un poco en responder.', 10),
(8, 3, 5.0, 'Ubicación y comunicación inmejorables con el propietario.', 4),
(9, 2, 3.5, 'La reserva se canceló pero el trato fue respetuoso.', 5);

-- Reseñas de la Habitación / Inmueble
INSERT INTO room_review (id_booking_roomreview, rate_room_roomreview, rate_owner_roomreview, desc_roomreview, author_roomreview) VALUES
(1, 4.8, 5.0, 'La habitación es muy luminosa y el escritorio es grande para estudiar.', 4),
(2, 4.5, 4.5, 'Muy bien equipada y cerca de la parada de metro.', 5),
(3, 4.0, 4.2, 'Zona tranquila para estudiar, wifi de buena velocidad.', 6),
(4, 5.0, 5.0, 'Espaciosa y con baño privado implacable. 10/10.', 7),
(5, 3.9, 4.0, 'Buena relación calidad-precio en el centro.', 8),
(6, 4.7, 4.8, 'Cerca de la universidad y con balcón muy bonito.', 9),
(7, 4.0, 4.0, 'Buena climatización en invierno y cama cómoda.', 10),
(8, 5.0, 5.0, 'A 5 minutos a pie del campus, recomendada.', 4),
(9, 3.0, 3.5, 'Habitación correcta según las fotos publicadas.', 5),
(10, 4.2, 4.5, 'Instalaciones modernas y piso reformado.', 6);
```
### Queries tipo SELECT

```sql
SELECT * FROM booking;
SELECT * FROM country;
SELECT * FROM edu_center;
SELECT * FROM enrollment;
SELECT * FROM neighborhood;
SELECT * FROM pay_method;
SELECT * FROM post;
SELECT * FROM preference;
SELECT * FROM "role";
SELECT * FROM roleuser;
SELECT * FROM room;
SELECT * FROM room_review;
SELECT * FROM town;
SELECT * FROM "user";
SELECT * FROM user_review;

SELECT name_user, mail_user FROM "user" INNER JOIN roleuser ON id_user=id_user_roleuser
WHERE id_role_roleuser=1;

SELECT address_room, postalcode_room, name_town FROM room INNER JOIN town ON town_room=id_town;

SELECT name_user, phone_user FROM "user" INNER JOIN roleuser ON id_user=id_user_roleuser
WHERE id_role_roleuser=2;
```

### Queries tipo Update

```sql
UPDATE "user" SET phone_user = 612345678
WHERE id_user = 11;

UPDATE booking SET status_booking = 'confirmed'
WHERE id_booking = 1;

UPDATE post SET monthly_price_post = 1600
WHERE id_post = 1;

```

### Queries tipo Delete

```sql
DELETE FROM booking
WHERE id_booking = 1;

DELETE FROM room
WHERE id_room = 9;
-- Aparece error y ocurre porque estás intentando eliminar un registro de la tabla room (con id_room = 9) que está siendo utilizado o referenciado por uno o más registros en la tabla post.

DELETE FROM "user"
WHERE id_user = 6;
-- Aparece error y ocurre porque estás intentando eliminar un registro de la tabla user (con id_user = 6) que está siendo utilizado o referenciado por uno o más registros en la tabla booking.
```
### Queries con filtros WHERE

```sql
SELECT * FROM room WHERE private_bathroom_room = true;
SELECT * FROM room WHERE shared_bathroom_room = true;
SELECT * FROM booking WHERE status_booking = 'pending';
SELECT * FROM booking WHERE status_booking = 'active';
SELECT * FROM post WHERE (monthly_price_post/30) > 120000;
SELECT * FROM post WHERE timestamp_post > '2024-01-01';
SELECT * FROM "user" WHERE name_user='Carlos';
```

### Queries con filtros LIKE

```sql
SELECT * FROM "user" FROM "user" INNER JOIN roleuser ON id_user=id_user_roleuser
WHERE id_role_roleuser=1 AND name_user LIKE 'L%';

SELECT * FROM "user" FROM "user" INNER JOIN roleuser ON id_user=id_user_roleuser
WHERE id_role_roleuser=2 AND name_user LIKE 'M%';

SELECT * FROM room FROM room WHERE address_room LIKE '%Centro%';

SELECT * FROM preference WHERE wifi_preference=true;

SELECT * FROM room INNER JOIN town ON town_room=id_town
WHERE name_town LIKE '%a%';
```
### Queries de ordenamiento

```sql
SELECT * FROM room ORDER BY address_room ASC;

SELECT r.*, p.monthly_price_post, (p.monthly_price_post / 30) AS price_per_night 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
ORDER BY price_per_night DESC;

SELECT * 
FROM booking 
ORDER BY start_date_booking ASC;

SELECT DISTINCT u.* 
FROM "user" u 
JOIN roleuser ru ON u.id_user = ru.id_user_roleuser 
JOIN role r ON ru.id_role_roleuser = r.id_role 
WHERE r.name_role = 'Owner' 
ORDER BY u.name_user DESC;
```

### Rangos y listas

```sql
SELECT r.*, (p.monthly_price_post / 30) AS price_per_night 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
WHERE (p.monthly_price_post / 30) BETWEEN 80000 AND 200000;

SELECT * 
FROM room 
WHERE private_bathroom_room = true OR shared_bathroom_room = true;

SELECT * 
FROM booking 
WHERE status_booking IN ('pending', 'confirmed');

SELECT r.*, t.name_town 
FROM room r 
JOIN town t ON r.town_room = t.id_town 
WHERE t.name_town IN ('Bogotá', 'Medellín', 'Madrid', 'Barcelona');
```

### Relaciones con JOIN

```sql
SELECT r.address_room AS titulo_habitacion, u.name_user || ' ' || u.lastname_user AS anfitrion 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user;

SELECT 
    r.address_room AS titulo_habitacion, 
    CASE WHEN r.private_bathroom_room THEN 'Privada' ELSE 'Compartida' END AS tipo_habitacion, 
    u.name_user || ' ' || u.lastname_user AS anfitrion 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user;

SELECT b.*, u.name_user || ' ' || u.lastname_user AS huesped 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user;

SELECT b.*, r.address_room AS titulo_habitacion 
FROM booking b 
JOIN post p ON b.id_post_booking = p.id_post 
JOIN room r ON p.id_room_post = r.id_room;

SELECT 
    b.id_booking, 
    u.name_user || ' ' || u.lastname_user AS huesped, 
    r.address_room AS habitacion, 
    b.start_date_booking AS check_in, 
    b.end_date_booking AS check_out, 
    b.status_booking AS estado 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
JOIN post p ON b.id_post_booking = p.id_post 
JOIN room r ON p.id_room_post = r.id_room;

SELECT b.id_booking, b.pay_confirm_booking, pm.name_pay, u.name_user || ' ' || u.lastname_user AS huesped 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
LEFT JOIN pay_method pm ON b.pay_method_booking = pm.id_pay;

SELECT id_room, address_room, wifi_room, aircon_room, balcony_room, closet_room, private_bathroom_room 
FROM room;

SELECT 
    r.address_room AS habitacion, 
    'Wi-Fi / Aire Acondicionado' AS servicio, 
    p.deposit_price_post AS costo_adicional, 
    r.status_room AS disponibilidad 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post;

SELECT 
    r.address_room AS habitacion, 
    u.name_user || ' ' || u.lastname_user AS anfitrion, 
    rr.desc_roomreview AS resena, 
    rr.rate_room_roomreview AS calificacion 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
JOIN room_review rr ON b.id_booking = rr.id_booking_roomreview;

SELECT 
    b.id_booking, 
    u_huesped.name_user || ' ' || u_huesped.lastname_user AS huesped, 
    u_anfitrion.name_user || ' ' || u_anfitrion.lastname_user AS anfitrion, 
    r.address_room AS habitacion 
FROM booking b 
JOIN "user" u_huesped ON b.id_user_booking = u_huesped.id_user 
JOIN post p ON b.id_post_booking = p.id_post 
JOIN room r ON p.id_room_post = r.id_room 
JOIN "user" u_anfitrion ON r.id_owner_room = u_anfitrion.id_user;
```

### Consultas de negocio con JOIN

```sql
SELECT r.* 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user 
WHERE u.name_user = 'Carlos' AND u.lastname_user = 'Gómez';

SELECT b.* 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
WHERE u.name_user = 'Laura' AND u.lastname_user = 'López';

SELECT address_room, wifi_room, aircon_room, balcony_room, closet_room, private_bathroom_room 
FROM room 
WHERE address_room = 'Calle Pez 12, 3A';

SELECT u.name_user || ' ' || u.lastname_user AS huesped 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
JOIN post p ON b.id_post_booking = p.id_post 
JOIN room r ON p.id_room_post = r.id_room 
WHERE r.address_room = 'Calle Pez 12, 3A';

SELECT DISTINCT r.id_room, r.address_room 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
JOIN room_review rr ON b.id_booking = rr.id_booking_roomreview;

SELECT r.id_room, r.address_room 
FROM room r 
LEFT JOIN post p ON r.id_room = p.id_room_post 
LEFT JOIN booking b ON p.id_post = b.id_post_booking 
LEFT JOIN room_review rr ON b.id_booking = rr.id_booking_roomreview 
WHERE rr.id_roomreview IS NULL;

SELECT id_room, address_room 
FROM room 
WHERE wifi_room = true OR aircon_room = true OR private_bathroom_room = true;

SELECT id_room, address_room 
FROM room 
WHERE wifi_room = false AND aircon_room = false AND private_bathroom_room = false;
```
### Funciones de agregación

```sql
SELECT COUNT(DISTINCT id_user_roleuser) AS total_anfitriones 
FROM roleuser ru 
JOIN role r ON ru.id_role_roleuser = r.id_role 
WHERE r.name_role = 'Owner';

SELECT COUNT(DISTINCT id_user_roleuser) AS total_huespedes 
FROM roleuser ru 
JOIN role r ON ru.id_role_roleuser = r.id_role 
WHERE r.name_role = 'Student';

SELECT COUNT(DISTINCT id_room_post) AS habitaciones_publicadas 
FROM post;

SELECT COUNT(*) AS total_reservas 
FROM booking;

SELECT AVG(monthly_price_post / 30) AS precio_promedio_noche 
FROM post;

SELECT r.address_room, MAX(p.monthly_price_post / 30) AS max_precio_noche 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
GROUP BY r.address_room 
ORDER BY max_precio_noche DESC 
LIMIT 1;

SELECT r.address_room, MIN(p.monthly_price_post / 30) AS min_precio_noche 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
GROUP BY r.address_room 
ORDER BY min_precio_noche ASC 
LIMIT 1;

SELECT SUM(p.monthly_price_post) AS total_ingresos 
FROM booking b 
JOIN post p ON b.id_post_booking = p.id_post 
WHERE b.pay_confirm_booking = true;
```

### Group by

```sql
SELECT 
    CASE WHEN private_bathroom_room THEN 'Baño Privado' ELSE 'Baño Compartido' END AS tipo, 
    COUNT(*) AS cantidad 
FROM room 
GROUP BY tipo;

SELECT u.name_user || ' ' || u.lastname_user AS anfitrion, COUNT(r.id_room) AS total_habitaciones 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user 
GROUP BY u.id_user, u.name_user, u.lastname_user;

SELECT u.name_user || ' ' || u.lastname_user AS huesped, COUNT(b.id_booking) AS total_reservas 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
GROUP BY u.id_user, u.name_user, u.lastname_user;

SELECT r.address_room AS habitacion, COUNT(b.id_booking) AS total_reservas 
FROM booking b 
JOIN post p ON b.id_post_booking = p.id_post 
JOIN room r ON p.id_room_post = r.id_room 
GROUP BY r.id_room, r.address_room;

SELECT 
    address_room, 
    (CASE WHEN wifi_room THEN 1 ELSE 0 END + 
     CASE WHEN aircon_room THEN 1 ELSE 0 END + 
     CASE WHEN balcony_room THEN 1 ELSE 0 END + 
     CASE WHEN closet_room THEN 1 ELSE 0 END + 
     CASE WHEN private_bathroom_room THEN 1 ELSE 0 END) AS total_servicios 
FROM room;

SELECT t.name_town AS ciudad, AVG(p.monthly_price_post / 30) AS precio_promedio 
FROM room r 
JOIN town t ON r.town_room = t.id_town 
JOIN post p ON r.id_room = p.id_room_post 
GROUP BY t.name_town;
```

### GROUP BY + HAVING

```sql
SELECT u.name_user || ' ' || u.lastname_user AS anfitrion, COUNT(r.id_room) AS total_habitaciones 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user 
GROUP BY u.id_user, u.name_user, u.lastname_user 
HAVING COUNT(r.id_room) > 1;

SELECT 
    CASE WHEN r.private_bathroom_room THEN 'Baño Privado' ELSE 'Baño Compartido' END AS tipo, 
    COUNT(p.id_post) AS publicaciones 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
GROUP BY tipo 
HAVING COUNT(p.id_post) > 1;

SELECT u.name_user || ' ' || u.lastname_user AS huesped, COUNT(b.id_booking) AS total_reservas 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
GROUP BY u.id_user, u.name_user, u.lastname_user 
HAVING COUNT(b.id_booking) > 1;

SELECT address_room 
FROM room 
WHERE (CASE WHEN wifi_room THEN 1 ELSE 0 END + 
       CASE WHEN aircon_room THEN 1 ELSE 0 END + 
       CASE WHEN balcony_room THEN 1 ELSE 0 END + 
       CASE WHEN closet_room THEN 1 ELSE 0 END + 
       CASE WHEN private_bathroom_room THEN 1 ELSE 0 END) > 1;
```

### Subconsultas

```sql
SELECT r.address_room, (p.monthly_price_post / 30) AS precio_noche 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
WHERE (p.monthly_price_post / 30) > (
    SELECT AVG(monthly_price_post / 30) FROM post
);

SELECT b.id_booking, p.monthly_price_post 
FROM booking b 
JOIN post p ON b.id_post_booking = p.id_post 
WHERE p.monthly_price_post > (
    SELECT AVG(p2.monthly_price_post) 
    FROM booking b2 
    JOIN post p2 ON b2.id_post_booking = p2.id_post
);

SELECT * 
FROM room 
WHERE id_room IN (
    SELECT p.id_room_post 
    FROM post p 
    JOIN booking b ON p.id_post = b.id_post_booking
);

SELECT * 
FROM room 
WHERE id_room NOT IN (
    SELECT p.id_room_post 
    FROM post p 
    JOIN booking b ON p.id_post = b.id_post_booking
);

SELECT * 
FROM "user" 
WHERE id_user IN (
    SELECT DISTINCT id_owner_room FROM room
);

SELECT * 
FROM "user" 
WHERE id_user IN (
    SELECT id_user_booking 
    FROM booking 
    WHERE status_booking = 'confirmed'
);

SELECT r.address_room 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
GROUP BY r.id_room, r.address_room 
HAVING COUNT(b.id_booking) = (
    SELECT MAX(cant) FROM (
        SELECT COUNT(b2.id_booking) AS cant 
        FROM booking b2 
        JOIN post p2 ON b2.id_post_booking = p2.id_post 
        GROUP BY p2.id_room_post
    ) AS sub
);

SELECT r.address_room, (p.monthly_price_post / 30) AS precio_noche 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
WHERE (p.monthly_price_post / 30) = (
    SELECT MAX(monthly_price_post / 30) FROM post
);
```

### LEFT JOIN y análisis de datos faltantes

```sql
SELECT u.* 
FROM "user" u 
JOIN roleuser ru ON u.id_user = ru.id_user_roleuser 
JOIN role r ON ru.id_role_roleuser = r.id_role 
LEFT JOIN room rm ON u.id_user = rm.id_owner_room 
WHERE r.name_role = 'Owner' AND rm.id_room IS NULL;

SELECT r.* 
FROM room r 
LEFT JOIN post p ON r.id_room = p.id_room_post 
LEFT JOIN booking b ON p.id_post = b.id_post_booking 
WHERE b.id_booking IS NULL;

SELECT * 
FROM room 
WHERE wifi_room = false AND aircon_room = false AND private_bathroom_room = false AND balcony_room = false AND closet_room = false;

SELECT u.* 
FROM "user" u 
JOIN roleuser ru ON u.id_user = ru.id_user_roleuser 
JOIN role r ON ru.id_role_roleuser = r.id_role 
LEFT JOIN booking b ON u.id_user = b.id_user_booking 
WHERE r.name_role = 'Student' AND b.id_booking IS NULL;

SELECT r.* 
FROM room r 
LEFT JOIN post p ON r.id_room = p.id_room_post 
LEFT JOIN booking b ON p.id_post = b.id_post_booking 
LEFT JOIN room_review rr ON b.id_booking = rr.id_booking_roomreview 
WHERE rr.id_roomreview IS NULL;
```

###  Consultas de reto

```sql
SELECT 
    r.address_room AS habitacion, 
    CASE WHEN r.private_bathroom_room THEN 'Privada' ELSE 'Compartida' END AS tipo, 
    u_anfitrion.name_user || ' ' || u_anfitrion.lastname_user AS anfitrion, 
    u_huesped.name_user || ' ' || u_huesped.lastname_user AS huesped, 
    CASE WHEN r.wifi_room THEN 'WiFi Incluido' ELSE 'Sin WiFi' END AS servicio 
FROM booking b 
JOIN post p ON b.id_post_booking = p.id_post 
JOIN room r ON p.id_room_post = r.id_room 
JOIN "user" u_anfitrion ON r.id_owner_room = u_anfitrion.id_user 
JOIN "user" u_huesped ON b.id_user_booking = u_huesped.id_user;

SELECT 
    CASE WHEN private_bathroom_room THEN 'Baño Privado' ELSE 'Baño Compartido' END AS tipo, 
    COUNT(*) AS total 
FROM room 
GROUP BY tipo 
HAVING COUNT(*) >= 2;

SELECT u.name_user || ' ' || u.lastname_user AS anfitrion, COUNT(r.id_room) AS total_habitaciones 
FROM room r 
JOIN "user" u ON r.id_owner_room = u.id_user 
GROUP BY u.id_user, u.name_user, u.lastname_user 
ORDER BY total_habitaciones DESC 
LIMIT 1;

SELECT r.address_room, COUNT(b.id_booking) AS total_reservas 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
GROUP BY r.id_room, r.address_room 
ORDER BY total_reservas DESC 
LIMIT 1;

SELECT u.name_user || ' ' || u.lastname_user AS huesped, COUNT(b.id_booking) AS total_reservas 
FROM booking b 
JOIN "user" u ON b.id_user_booking = u.id_user 
GROUP BY u.id_user, u.name_user, u.lastname_user 
ORDER BY total_reservas DESC;

SELECT DISTINCT r.id_room, r.address_room 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
WHERE (r.wifi_room = true OR r.aircon_room = true OR r.private_bathroom_room = true);

SELECT DISTINCT r.id_room, r.address_room 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
WHERE (r.wifi_room = false AND r.aircon_room = false AND r.private_bathroom_room = false);

SELECT 'Servicios Generales' AS servicio_no_asignado 
WHERE NOT EXISTS (
    SELECT 1 FROM room WHERE wifi_room = true OR aircon_room = true
);

SELECT r.address_room, SUM(p.monthly_price_post) AS ingreso_total 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
WHERE b.pay_confirm_booking = true 
GROUP BY r.id_room, r.address_room;

SELECT r.address_room, p.monthly_price_post AS valor_reserva 
FROM room r 
JOIN post p ON r.id_room = p.id_room_post 
JOIN booking b ON p.id_post = b.id_post_booking 
WHERE b.pay_confirm_booking = true 
ORDER BY valor_reserva DESC 
LIMIT 1;
```

### Preguntas de análisis

¿Cómo identificarías si una relación 1:1 entre reservas y pagos está bien implementada?

```text
Se identifica verificando que la clave foránea en la tabla secundaria de pagos tenga una restricción UNIQUE asignada, garantizando que una reserva solo pueda vincularse como máximo a un único pago.
```
¿Qué consulta usarías para demostrar que la relación entre habitaciones y servicios es muchos a muchos?

```text
Se demuestra evaluando que una misma habitación posee múltiples flags de servicios habilitados y que cada tipo de servicio está activo en múltiples habitaciones:
```
```sql
SELECT id_room, wifi_room, aircon_room, private_bathroom_room FROM room;
```

¿Qué diferencia hay entre usar JOIN y LEFT JOIN?

```text
JOIN (INNER JOIN): Retorna únicamente las filas donde existe coincidencia en ambas tablas.

LEFT JOIN: Retorna todas las filas de la tabla izquierda y, si no existen coincidencias en la tabla derecha, completa las columnas con valores NULL.
```

¿Cuándo usarías una subconsulta en lugar de un JOIN?

```text
Se utiliza una subconsulta cuando:

Se necesita comparar un valor individual contra agregaciones calculadas previamente (ej. buscar elementos cuyo valor sea mayor al AVG()).

Se requiere verificar existencia mediante operadores IN, NOT IN, EXISTS o NOT EXISTS sin duplicar filas del conjunto de datos resultante.
```