# clase 1

## ejemplos SQL usados en la clase

Para seguir el ejemplo:

   1. Abrir el pdf de la clase 1.
   2. Instalar el postgresql y el 7zip (para descomprimir el archivo .zip). [ver software](../software.md)
   3. Bajarse la base a usuario de la EAH 2017 de la página de la [DGEyC-CABA](https://www.estadisticaciudad.gob.ar/eyc/?cat=93)
   4. Descomprimir el archivo en un carpeta local
   5. Ir a [txt-to-sql](https://codenautas.com/txt-to-sql) para convertir el .txt en un archivo .sql
      1. Seleccionar el archivo eah2015_usuarios_hog.txt
      2. Presionar el botón **generar**
      3. Hacer click en el link **descargar**
   6. Abrir el programa llamado pgAdmin (puede ser el pgAdmin III o pgAdmin 4). 
      1. Abrir la herramienta para ejecutar SQL ![ejecutarSQL](../imagenes/ejecutarSQL.png)
      2. Abrir el .sql descargado
      3. Ejecutar las instrucciones
      4. Abrir una nueva ventana y correr los comandos que acá se muestran

## pruebas del pdf

Contando todos los registros de la tabla desagregado (agrupados) por tipo de vivienda
```sql
select v2_2, count(*)
  from eah2015_usuarios_hog
  group by v2_2;
```

Contando todos los registros de la tabla sin desagregar (para obtener el denominador de la proporción)
```sql
select count(*)
  from eah2015_usuarios_hog;
```

Obteniendo la proporción. _(**Cuidado** el denominador está puesto a mano, debería calcularse ahí mismo)_
```sql
select v2_2, count(*)*100/6136
  from eah2015_usuarios_hog
  group by v2_2;
```

Select dentro de Select (para calcular el denominador dentro). _(**Cuidado** el valor devuelto es un número entero porque todos los operandos son enteros)_
```sql
select v2_2, count(*)*100/(select count(*) from eah2015_usuarios_hog)
  from eah2015_usuarios_hog
  group by v2_2;
```

Indicando explícitamente (poniendo **100.0**) que queremos que haga la cuenta con decimales
```sql
select v2_2, count(*)*100.0/(select count(*) from eah2015_usuarios_hog)
  from eah2015_usuarios_hog
  group by v2_2;
```

Lo queremos ordenado por tipo de vivienda
```sql
select v2_2, count(*)*100.0/(select count(*) from eah2015_usuarios_hog)
  from eah2015_usuarios_hog
  group by v2_2
  order by v2_2;
```

Utilizando el **factor de expansión** (ahora en vez de contar registros se suma el factor de expansión de cada registro)
```sql
select v2_2, sum(fexp)*100.0/
  (select sum(fexp)
    from eah2015_usuarios_hog)
  from eah2015_usuarios_hog
  group by v2_2
  order by v2_2;
```

Agregando un filtro para obtener solo un registro por vivienda (en este caso indicando que el número de hogar es 1)
```sql
select v2_2, sum(fexp)*100.0/
  (select sum(fexp)
    from eah2015_usuarios_hog
    where nhogar=1)
  from eah2015_usuarios_hog
  where nhogar=1
  group by v2_2
  order by v2_2;
```

## Tareas

   1. Reproducir la cuenta para el 2016
   2. Elegir otro cuadro y pensar si con los conocimientos actuales se puede confeccionar
   3. Mirar esta página de ejercicios de postgresql: [https://pgexercises.com/](https://pgexercises.com/)