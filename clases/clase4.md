# hecho en clase

## metadatos de tabulados

Elijo un SQL que obtiene datos para un tabulado como primer plantilla:
```sql
WITH viviendas AS (
  SELECT v2_2,
         fexp
    FROM eah2015_usuarios_hog
    WHERE nhogar=1)
SELECT v2_2, 
       SUM(fexp)*100.0/(SELECT SUM(fexp) FROM viviendas)
  FROM viviendas
  GROUP BY v2_2
  ORDER BY v2_2;
```

Genero los metadatos de los tabulados:

```sql
create table def_tabulados(
  tabulado text primary key,
  var_corte text,
  filtro text,
  u_a text,
  plantilla text
);

insert into def_tabulados 
  values ('cuadro 1','comuna, v2_2','nhogar=1','hogares','t_2'),
         ('cuadro 2','zona, sexo','edad>=10','personas','t_2');
```

```sql
Genero el SQL de la generación del tabulado a partir de los metadatos
SELECT replace(replace($$
WITH viviendas AS (
  SELECT ${var_corte},
         fexp
    FROM eah2015_usuarios_hog
    WHERE ${filtro})
SELECT ${var_corte}, 
       SUM(fexp)*100.0/(SELECT SUM(fexp) FROM viviendas)
  FROM viviendas
  GROUP BY ${var_corte}
  ORDER BY ${var_corte};
$$,'${var_corte}',var_corte)
,'${filtro}',filtro) as el_sql_del_tabulado
  FROM def_tabulados
  WHERE tabulado='cuadro 1';
```

## abuelos y nietos

Qué porcentaje de nietos que viven con abuelos jefes:

```sql
SELECT sum(CASE WHEN parentes_2=5 and edad<18 THEN fexp ELSE 0 END)*100.0/
       sum(fexp) as cant_nietos
  from miembros;
```
Zonifico las viviendas de acuerdo con su comuna

```sql
alter table viviendas add column zona integer;

update viviendas set zona = 
  CASE WHEN comuna in (2,13,14) THEN 1
     WHEN comuna in (4,8,9,10) THEN 3
     ELSE 2 END;
```
Calculo por zona cantidades, denominadores y proporciones, totales y expandidos:

```sql
SELECT zona,
       sum(CASE WHEN parentes_2=5 and edad<18 THEN v.fexp ELSE 0 END) as c_nietos,
       sum(v.fexp) as cant_total,
       sum(CASE WHEN parentes_2=5 and edad<18 THEN v.fexp ELSE 0 END)*100.0/
       sum(v.fexp) as p_nietos,
       sum(CASE WHEN parentes_2=5 and edad<18 THEN 1 ELSE 0 END) as c_nietosm,
       sum(1) as cant_totalm,
       sum(CASE WHEN parentes_2=5 and edad<18 THEN 1 ELSE 0 END)*100.0/
       sum(1) as p_nietosm
  from miembros AS m JOIN viviendas AS v ON m.id=v.id
  GROUP BY zona;
```
Listar cada hogar con la cantidad de nietos en el hogar:

```sql
SELECT m.id,m.nhogar,
       sum(CASE WHEN parentes_2=5 and edad<18 THEN 1 ELSE 0 END) as c_nietos
  from miembros AS m JOIN viviendas AS v ON m.id=v.id
  GROUP BY m.id,m.nhogar
  ORDER BY 3 DESC, 1,2;
```

Proporcion de hogares con nietos

```sql
WITH hogares_nieto AS (
SELECT m.id,m.nhogar,
       sum(CASE WHEN parentes_2=5 and edad<18 THEN 1 ELSE 0 END) as c_nietos,
       m.fexp,
       v.zona
  from miembros AS m JOIN viviendas AS v ON m.id=v.id
  GROUP BY m.id,m.nhogar,m.fexp,v.zona
)
SELECT zona, 
       sum(CASE WHEN c_nietos>0 THEN fexp ELSE 0 END) as c_h_c_n,
       sum(fexp) as c_h,
       sum(CASE WHEN c_nietos>0 THEN fexp ELSE 0 END)*100.0/
       sum(fexp) as p_h_c_n
  FROM hogares_nieto
  GROUP BY zona
  ORDER BY zona;
```

Incluimos nietos con el siguiente criterio: Hijos del jefe donde en el hogar vive también el padre, madre o suegro del jefe
```sql
WITH hogares_nieto AS (
SELECT m.id,m.nhogar,
       sum(CASE WHEN 
             (  parentes_2=5 -- nieto del jefe
             OR 
                parentes_2=3 -- hijo del jefe
                AND (
                  SELECT count(*)
                    FROM miembros cpms -- candidato a padre, madre suegro
                    WHERE cpms.id = m.id
                      AND cpms.nhogar = m.nhogar
                      AND cpms.parentes_2=6 -- es padre, madre o suegro
                    )>0
             )
           and edad<18 THEN 1 ELSE 0 END) as c_nietos,
       m.fexp,
       v.zona
  from miembros AS m JOIN viviendas AS v ON m.id=v.id
  GROUP BY m.id,m.nhogar,m.fexp,v.zona
)
SELECT zona, 
       sum(CASE WHEN c_nietos>0 THEN fexp ELSE 0 END) as c_h_c_n,
       sum(fexp) as c_h,
       sum(CASE WHEN c_nietos>0 THEN fexp ELSE 0 END)*100.0/
       sum(fexp) as p_h_c_n
  FROM hogares_nieto
  GROUP BY zona
  ORDER BY zona;
```

### la manera de los lunes

```sql
WITH hogares_ni as (
 SELECT id, nhogar, 
    sum(
      CASE WHEN parentes_2=5 and edad<18 THEN 1 ELSE 0 END
    ) cant_nietos
  from miembros
  group by id, nhogar
 UNION distinct
 SELECT m1.id, m1.nhogar, count(distinct m1.miembro) as cant_nietos
  from miembros m1 
    inner join miembros m2 on m1.id=m2.id and m1.nhogar=m2.nhogar
  where m1.parentes_2=3 and m2.parentes_2=6
  group by m1.id, m1.nhogar
)
SELECT sum(case when cant_nietos>0 then fexp else 0 end)*100.0/sum(fexp)
  from hogares_ni hn 
    inner join viviendas v on hn.id=v.id;
```

### ----------

```sql
select * -- count(*), sum(fexp)
  from miembros m
  where parentes_2 = 5 -- nieto
     OR parentes_2 = 3 -- hijo
       AND EXISTS (SELECT * 
                     FROM miembros p
                     WHERE p.id = m.id
                       AND p.nhogar = m.nhogar -- del mismo hogar
                       AND p.parentes_2 = 6)-- que sea padre
                      -- otro registro 

```

```sql
select n.id, n.nhogar, n.miembro, avg(a.edad - n.edad) as promedio
  from miembros n
    join miembros a
      on n.id = a.id AND n.nhogar = a.nhogar
        AND a.parentes_2 = 
          CASE WHEN n.parentes_2 = 5 THEN 1
               WHEN n.parentes_2 = 3 THEN 6
          END
  where n.parentes_2 = 5 -- nieto
     OR n.parentes_2 = 3 -- hijo
  GROUP BY n.id, n.nhogar, n.miembro

```

```sql
select avg(promedio), avg(promedio) - 54.7538461538461538
from (
select n.id, n.nhogar, n.miembro, avg(a.edad - n.edad) as promedio
  from miembros n
    join miembros a
      on n.id = a.id AND n.nhogar = a.nhogar
        AND a.parentes_2 = 
          CASE WHEN n.parentes_2 = 5 THEN 1
               WHEN n.parentes_2 = 3 THEN 6
          END
  where n.parentes_2 = 5 -- nieto
     OR n.parentes_2 = 3 -- hijo
  GROUP BY n.id, n.nhogar, n.miembro
) promedio_por_nieto```

```

```sql
select sexo, avg(promedio), avg(promedio) - 54.7538461538461538
from (
select n.id, n.nhogar, n.miembro, a.sexo, avg(a.edad - n.edad) as promedio
  from miembros n
    join miembros a
      on n.id = a.id AND n.nhogar = a.nhogar
        AND a.parentes_2 = 
          CASE WHEN n.parentes_2 = 5 THEN 1
               WHEN n.parentes_2 = 3 THEN 6
          END
  where n.parentes_2 = 5 -- nieto
     OR n.parentes_2 = 3 -- hijo
  GROUP BY n.id, n.nhogar, n.miembro, a.sexo
) promedio_por_nieto
GROUP BY sexo
```

```sql
select a.sexo, avg(a.edad - n.edad) as p_dif
       , avg(a.edad) as p_edad_abuelo
       , avg(n.edad) as p_edad_nieto
       , count(*) as cantidad
  from miembros n
    join miembros a
      on n.id = a.id AND n.nhogar = a.nhogar
        AND a.parentes_2 = 
          CASE WHEN n.parentes_2 = 5 THEN 1
               WHEN n.parentes_2 = 3 THEN 6
          END
  where n.parentes_2 = 5 -- nieto
     OR n.parentes_2 = 3 -- hijo
  GROUP BY a.sexo
```

```sql
CREATE TABLE comunas
  AS SELECT DISTINCT comuna 
       FROM viviendas
       ORDER BY comuna;
       
ALTER TABLE comunas
  ADD PRIMARY KEY (comuna);
  
ALTER TABLE comunas
  ADD zona INTEGER;  
  
UPDATE comunas SET zona = 
  CASE WHEN comuna in (2,13,14) THEN 1
     WHEN comuna in (4,8,9,10) THEN 3
     ELSE 2 END;
```

```sql
select a.sexo, zona
       , 1.0*sum((a.edad - n.edad)*n.fexp)/sum(n.fexp) as p_dif
       , 1.0*sum(a.edad*n.fexp)/sum(n.fexp) as p_edad_abuelo
       , 1.0*sum(n.edad*n.fexp)/sum(n.fexp) as p_edad_nieto
       , 1.0*sum(n.fexp) as cantidad
  from miembros n
    join miembros a
      on n.id = a.id AND n.nhogar = a.nhogar
        AND a.parentes_2 = 
          CASE WHEN n.parentes_2 = 5 THEN 1
               WHEN n.parentes_2 = 3 THEN 6
          END
    join comunas c on c.comuna = n.comuna
  where n.parentes_2 = 5 -- nieto
     OR n.parentes_2 = 3 -- hijo
  GROUP BY a.sexo, zona
  order by zona, a.sexo
```

```sql
select count(distinct array[n.id, n.nhogar, n.miembro])
       , 1.0*sum(n.fexp) as cantidad
  from miembros n
    join miembros a
      on n.id = a.id AND n.nhogar = a.nhogar
        AND a.parentes_2 = 1
    join comunas c on c.comuna = n.comuna
  where n.parentes_2 = 5 -- nieto
    AND NOT EXISTS (
      SELECT *
        FROM miembros o
        WHERE o.id = n.id AND o.nhogar = n.nhogar
          AND o.parentes_2 IN (3,4,7,8,9)
    )
```

```sql
```
```sql
```

# otras maneras

```sql
SELECT nieto.id, nieto.nhogar
  FROM miembros as jefe 
    inner join miembros as nieto 
       on jefe.id=nieto.id and jefe.nhogar=nieto.nhogar
  WHERE jefe.parentes_2=1 
    and nieto.parentes_2=5
  ORDER BY nieto.id, nieto.nhogar
  
  
SELECT nieto.id, nieto.nhogar
  FROM miembros as nieto 
  WHERE nieto.parentes_2=5
  ORDER BY nieto.id, nieto.nhogar

SELECT hijo.id, hijo.nhogar
  FROM miembros as padre 
    inner join miembros as hijo 
       on padre.id=hijo.id and padre.nhogar=hijo.nhogar
  WHERE padre.parentes_2=6 
    and hijo.parentes_2=3
  ORDER BY hijo.id, hijo.nhogar  

-- hay repetidos

SELECT DISTINCT hijo.id, hijo.nhogar
  FROM miembros as padre 
    inner join miembros as hijo 
       on padre.id=hijo.id and padre.nhogar=hijo.nhogar
  WHERE padre.parentes_2=6 
    and hijo.parentes_2=3
  ORDER BY hijo.id, hijo.nhogar  

WITH hogares_AN as (
SELECT DISTINCT hijo.id, hijo.nhogar
  FROM miembros as padre 
    inner join miembros as hijo 
       on padre.id=hijo.id and padre.nhogar=hijo.nhogar
  WHERE padre.parentes_2=6 
    and hijo.parentes_2=3
UNION 
SELECT DISTINCT nieto.id, nieto.nhogar
  FROM miembros as nieto 
  WHERE nieto.parentes_2=5
ORDER BY id, nhogar) 
SELECT sum(fexp)*100.0/
     (SELECT SUM(fexp) FROM hogares AS h INNER JOIN viviendas AS v ON v.id=h.id)
  FROM hogares_AN AS h inner join viviendas AS v ON v.id=h.id
  
  
-- por comuna

WITH hogares_AN as (
SELECT DISTINCT hijo.id, hijo.nhogar
  FROM miembros as padre 
    inner join miembros as hijo 
       on padre.id=hijo.id and padre.nhogar=hijo.nhogar
  WHERE padre.parentes_2=6 
    and hijo.parentes_2=3
UNION 
SELECT DISTINCT nieto.id, nieto.nhogar
  FROM miembros as nieto 
  WHERE nieto.parentes_2=5
ORDER BY id, nhogar) 
SELECT comuna, sum(fexp)*100.0/
     (SELECT SUM(fexp) 
        FROM hogares AS h 
        INNER JOIN viviendas AS v2 ON v2.id=h.id
        WHERE v1.comuna=v2.comuna
      )
  FROM hogares_AN AS h inner join viviendas AS v1 ON v1.id=h.id
  GROUP BY comuna
  ORDER BY comuna
  
  
CREATE TABLE comunas
  AS SELECT DISTINCT comuna 
       FROM viviendas
       ORDER BY comuna;
       
ALTER TABLE comunas
  ADD PRIMARY KEY (comuna);
  
ALTER TABLE comunas
  ADD zona INTEGER;  
  
UPDATE comunas SET zona = 
  CASE WHEN comuna in (2,13,14) THEN 1
     WHEN comuna in (4,8,9,10) THEN 3
     ELSE 2 END;
     
WITH hogares_AN as (
SELECT DISTINCT hijo.id, hijo.nhogar
  FROM miembros as padre 
    inner join miembros as hijo 
       on padre.id=hijo.id and padre.nhogar=hijo.nhogar
  WHERE padre.parentes_2=6 
    and hijo.parentes_2=3
UNION 
SELECT DISTINCT nieto.id, nieto.nhogar
  FROM miembros as nieto 
  WHERE nieto.parentes_2=5
ORDER BY id, nhogar) 
, vivZ AS (
  select v.*, zona 
    FROM viviendas AS v 
    INNER JOIN comunas AS c ON v.comuna=c.comuna
)
SELECT zona, sum(fexp)*100.0/
     (SELECT SUM(fexp) 
        FROM hogares AS h 
        INNER JOIN vivZ AS v2 ON v2.id=h.id
        WHERE v1.zona=v2.zona
      )
  FROM hogares_AN AS h inner join vivZ AS v1 ON v1.id=h.id
  GROUP BY zona
  ORDER BY zona     
```
