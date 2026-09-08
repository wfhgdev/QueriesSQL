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


1.1. Creacion Base de Datos:

```sql
CREATE DATABASE "InmobiliariaDB"
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
```

1.2. Creación de tabla usuario:

```sql
CREATE TABLE "USER"
(
    id_user serial NOT NULL,
    num_docu_user character varying(30) NOT NULL,
    type_docu_user character varying(20) NOT NULL,
    name_user character varying(100) NOT NULL,
    lastname_user character varying(100) NOT NULL,
    birthyear_user integer NOT NULL,
    gender_user character varying(20) NOT NULL,
    mail_user character varying(150) NOT NULL,
    phone_user character varying(20),
    country_resi_user integer,
    status_user boolean NOT NULL,
    CONSTRAINT user_pkey PRIMARY KEY (id_user),
    CONSTRAINT user_mail_user_key UNIQUE (mail_user)
);

```

1.3. Creación de tabla ROLE_USER:

```sql
CREATE TABLE IF NOT EXISTS "ROLE_USER"
(
    id_user_roleuser integer NOT NULL,
    id_role_roleuser integer NOT NULL,
    CONSTRAINT role_user_pkey PRIMARY KEY (id_user_roleuser,id_role_roleuser)
);

```

1.4. Creación de tabla ROLE: