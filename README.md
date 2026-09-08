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


1. Creacion Base de Datos:

```sql
CREATE DATABASE "InmobiliariaDB"
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
```
2. 