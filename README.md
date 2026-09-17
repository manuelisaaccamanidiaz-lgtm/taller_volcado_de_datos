# 🗄️ Taller Práctico: Volcado y Normalización de Datos en PostgreSQL

Este proyecto contiene el procedimiento paso a paso para la reestructuración, importación y manipulación de datos en PostgreSQL utilizando un modelo relacional de países, ciudades, idiomas y continentes.

---

## 📋 Tabla de Contenidos
- [1. Eliminación de Tablas Previas](#1-eliminación-de-tablas-previas)
- [2. Creación del Esquema Relacional Base](#2-creación-del-esquema-relacional-base)
- [3. Carga de Datos Iniciales](#3-carga-de-datos-iniciales)
- [4. Normalización: Caso de Uso Continentes](#4-normalización-caso-de-uso-continentes)
  - [4.1 Creación de la tabla `continent`](#41-creación-de-la-tabla-continent)
  - [4.2 Consulta y Verificación de Continentes Únicos](#42-consulta-y-verificación-de-continentes-únicos)
  - [4.3 Volcado e Inserción de Datos](#43-volcado-e-inserción-de-datos)
- [5. Resultados Obtenidos](#5-resultados-obtenidos)

---

## 1. Eliminación de Tablas Previas

Antes de iniciar la reestructuración, se eliminan las tablas existentes respetando el orden de integridad referencial y restricciones de llaves foráneas (*Foreign Keys*).

```sql
DROP TABLE IF EXISTS city CASCADE;
DROP TABLE IF EXISTS region CASCADE;
DROP TABLE IF EXISTS country CASCADE;

```

**Verificación de tablas restantes:**

```
               Listado de relaciones
 Esquema |      Nombre      |   Tipo    |  Dueño
---------+------------------+-----------+----------
 public  | empleados        | tabla     | postgres
 public  | empleados_id_seq | secuencia | postgres

```

---

## 2. Creación del Esquema Relacional Base

Se definen las tablas `city`, `country` y `countrylanguage` con sus respectivas restricciones de tipo de datos y comprobaciones (`CHECK`).

```sql
-- Tabla: CIUDADES
CREATE TABLE "public"."city" (
    "id" int4 NOT NULL,
    "name" text NOT NULL,
    "countrycode" bpchar(3) NOT NULL,
    "district" text NOT NULL,
    "population" int4 NOT NULL CHECK (population >= 0),
    PRIMARY KEY ("id")
);

-- Tabla: PAÍSES
CREATE TABLE "public"."country" (
    "code" bpchar(3) NOT NULL,
    "name" text NOT NULL,
    "continent" text NOT NULL CHECK (
        (continent = 'Asia'::text) OR 
        (continent = 'South America'::text) OR 
        (continent = 'North America'::text) OR 
        (continent = 'Oceania'::text) OR 
        (continent = 'Antarctica'::text) OR 
        (continent = 'Africa'::text) OR 
        (continent = 'Europe'::text) OR 
        (continent = 'Central America'::text)
    ),
    "region" text NOT NULL,
    "surfacearea" float4 NOT NULL CHECK (surfacearea >= (0)::double precision),
    "indepyear" int2,
    "population" int4 NOT NULL,
    "lifeexpectancy" float4,
    "gnp" numeric(10,2),
    "gnpold" numeric(10,2),
    "localname" text NOT NULL,
    "governmentform" text NOT NULL,
    "headofstate" text,
    "capital" int4,
    "code2" bpchar(2) NOT NULL,
    PRIMARY KEY ("code")
);

-- Tabla: IDIOMAS POR PAÍS
CREATE TABLE "public"."countrylanguage" (
    "countrycode" bpchar(3) NOT NULL,
    "language" text NOT NULL,
    "isofficial" bool NOT NULL,
    "percentage" float4 NOT NULL CHECK ((percentage >= (0)::double precision) AND (percentage <= (100)::double precision)),
    PRIMARY KEY ("countrycode","language")
);

```

---

## 3. Carga de Datos Iniciales

Los datos poblacionales e información geográfica inicial se obtuvieron del repositorio oficial:
🔗 **Data Inicial:** [Gist Script SQL](https://gist.github.com/454db994cfff87a9ec542a1e92f21ff6.git)

### Vista previa de registros cargados:

* **Tabla `city`:**

* **Tabla `country`:**

* **Tabla `countrylanguage`:**


---

## 4. Normalización: Caso de Uso Continentes

### 4.1 Creación de la tabla `continent`

Con el fin de normalizar la base de datos y evitar la redundancia de texto en la tabla `country`, se creó la tabla catálogo `continent`:

```sql
CREATE TABLE public.continent (
    code serial4 NOT NULL,
    name text NULL,
    CONSTRAINT continent_pk PRIMARY KEY (code)
);

```

---

### 4.2 Consulta y Verificación de Continentes Únicos

Se requiere obtener el listado único de continentes existentes en la tabla `country` sin repeticiones:

```sql
SELECT DISTINCT continent 
FROM country 
ORDER BY continent ASC;

```

---

### 4.3 Volcado e Inserción de Datos

Para poblar automáticamente la tabla `continent` con los registros únicos de la tabla `country`, se ejecutó el siguiente comando de inserción con subconsulta:

```sql
INSERT INTO continent (name)
SELECT DISTINCT continent
FROM country
ORDER BY continent ASC;

```

---

## 5. Resultados Obtenidos

Al realizar la consulta sobre la nueva tabla `continent`, se verifica que los datos se han insertado de forma única, generando llaves primarias numéricas mediante la secuencia `serial4`:

```sql
SELECT code, name FROM continent;

```

### Resultado de la consulta:

**Salida en consola:**

```
 code |     name      
------+---------------
    1 | Africa
    2 | Antarctica
    3 | Asia
    4 | Central America
    5 | Europe
    6 | North America
    7 | Oceania
    8 | South America
(8 rows)

```

```

```

![Tabla City](img/city.png)
![Tabla Country](img/country.png)
![Tabla CountryLanguage](img/countrylanguage.png)
![Resultado Final Continent](img/continent.png)