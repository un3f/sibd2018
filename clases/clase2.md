# clase 2

## ejemplos SQL usados en la clase

Para empezar hay que crear una base de datos llamada **clase2_db**


### select

Primero nos bajamos los datos de la [EAH](https://www.estadisticaciudad.gob.ar/eyc/?p=86240) y
usamos [txt-to-sql](http://codenautas.com/txt-to-sql) para pasarlo a SQL. Luego metemos todo en la base de datos

Vamos a buscar qué hogares están en viviendas tipo 5. 

```sql
SELECT * 
  FROM eah2017_usuarios_hog
  WHERE v2_2=5
  ORDER BY id;
```

Vamos a contar las viviendas de cada tipo en la muestra

```sql
SELECT v2_2, count(*) as cantidad_viviendas
  FROM eah2017_usuarios_hog
  WHERE nhogar=1
  GROUP BY v2_2
  ORDER BY v2_2;
```

Cuando no hay cláusula `GROUP BY` se obtiene una fila por cada registro de la tabla que cumpla la condición del `WHERE` 
(cuando no hay `WHERE` son todos los registros de la tabla).

La presencia del `GROUP BY` determina que se obtendrán datos agrupados. 
Las columnas que aparecen listadas en `GROUP BY` son las únicas que pueden estar libres en la lista de campos del `SELECT`, 
el resto tiene que estar dentro de una función de agregación `SUM`, `COUNT`, `AVG`, etc

### select con subqueries

Ahora vamos a obtener el % del total, además queremos reclasificar los tipos de vivienda en 1=casa, 2=dept y resto=otros
```sql
WITH viviendas AS (
  SELECT CASE v2_2 WHEN 1 THEN 'casa' 
                   WHEN 2 THEN 'dept' 
                   ELSE 'otros' 
         END AS tipo_viv,
         fexp
    FROM eah2017_usuarios_hog
    WHERE nhogar=1)
SELECT tipo_viv, 
       SUM(fexp)*100.0/(SELECT SUM(fexp) FROM viviendas)
  FROM viviendas
  GROUP BY tipo_viv
  ORDER BY tipo_viv;
```

### select con joins

Ahora vamos a obtener el % de personas que viven en casa o departamento
```sql
WITH personas AS (
  SELECT CASE v2_2 WHEN 1 THEN 'casa' 
                   WHEN 2 THEN 'dept' 
                   ELSE 'otros' 
         END AS tipo_viv,
         i.fexp
    FROM eah2017_usuarios_hog h
      INNER JOIN eah2017_usuarios_ind i ON h.id=i.id
)
SELECT tipo_viv, 
       SUM(fexp)*100.0/(SELECT SUM(fexp) FROM personas),
       SUM(fexp)
  FROM personas
  GROUP BY tipo_viv
  ORDER BY tipo_viv;
```

Siempre hay que verificar con los totales:

```sql
SELECT sum(i.fexp)
  FROM eah2017_usuarios_ind
```

## Otros ejemplos vistos en clase

Select no agrupado

```sql
SELECT id, nhogar, comuna, v4, h3, v4-h3 as hab_compartidas
  FROM eah2017_usuarios_hog
  WHERE v4-h3>0
  ORDER BY comuna, id;

```

Select agrupado

```sql
SELECT comuna, count(*) as muestral, sum(fexp) as expandido
  FROM eah2017_usuarios_hog
  WHERE v4-h3>0
  GROUP BY comuna
  ORDER BY comuna;
```

select c/with

```sql
WITH hogares as (
     SELECT id, nhogar, comuna, v4, h3, v4-h3 as hab_compartidas
       FROM eah2017_usuarios_hog
)
SELECT *
  FROM hogares
  WHERE hab_compartidas>0
  ORDER BY id;
```

Porcentaje

```sql
WITH viviendas as (
  SELECT *, CASE v2_2 
              WHEN 1 THEN 'casa' 
              WHEN 2 THEN 'dto' 
              ELSE 'otro' 
            END as tipov
    FROM eah2017_usuarios_hog
    WHERE nhogar=1
)
SELECT tipov, round(sum(fexp)*100.0/(SELECT sum(fexp) FROM viviendas),1)
  FROM viviendas
  GROUP BY tipov
  ORDER BY tipov;
```
