# clase 5

## preparación

Hoy vamos a intentar poner en línea una mini aplicación web basada en base de datos. 

Primero hay que levantar el backup de la base de datos de la clase3 [clase3_db](dbs/clase3_db.backup)

Luego hay que instalar el backend.

### instrucciones para instalar el backend

Presionar simultáneamente las teclas de windows ![win](imagenes/win-key.png) y ![tecla R](imagenes/R-key.png), escribir **`CMD`** y dar *Enter*.
Se abrirá una pantalla negra llamada intérprete de comandos. Escribir ahí:

```sh
CD\
MKDIR hecho
```

Puede pasar que el `MKDIR` diga que el directorio ya existía, eso no es problema, seguimos adelante.

```sh
CD hecho
MKDIR npm
CD npm
git clone https://gitlab.com/untref/sibd2017-ejemplo-t.git
CD sibd2017-ejemplo-t
COPY example-config-db.json local-config-db.json
```

Se debe ver en la pantalla: `C:\hecho\npm\ejemplo-t`

Instalamos los paquetes de node y ponemos a correr el servidor:

```sh
npm install
npm start
```

Luego revisamos (editamos) el archivo de configuración llamado `local-config-db` para que coincidan el nombre de la base de datos y demás parámetros.

### Intentando hackear

   1. [http://localhost:3000/datos?regimen=1 AND h3=1](http://localhost:3000/datos?regimen=1%20AND%20h3=1)
   2. [http://localhost:3000/datos?regimen=1 UNION select usesuper::text,0,1 from pg_catalog.pg_user where usename=user](http://localhost:3000/datos?regimen=1%20UNION%20select%20usesuper::text,0,1%20from%20pg_catalog.pg_user%20where%20usename=user)

Analicemos los resultados:  
   1. En el primer caso logarmos cortar por una variable más. 
      El backend no debería permitirlo porque de esa manera se podrían estar dando resultados estadísticamente no significativos o se podría estar violando el secreto estadístico
   2. En el segundo caso obtenemos información del sistema, 
      podemos ver si el usuario que está usándose para acceder a la base de datos es "superusuario" o no.

## Solución del problema de seguridad

El problema es que en la línea del backend:

        return client.query(sql.replace('$1', req.query.regimen)).then(function(result){
        
se reemplazaba dentro del código *SQL* los ingresado por el usario en la posición *$1*. 
Cualquier cosa que ingresara el usuario se metía dentro de la instrucción *SQL*.
Así se logra obtener información que podría ser sensible. 

Una solución es reemplazar esa línea por:
        
        return client.query(sql, [req.query.regimen]).then(function(result){

donde lo que se hace es mandar a la base de datos la sentencia *SQL* por un lado y 
lo ingresado por el usuario (en la variable `regimen`) por separado 
y delegar en la base de datos el control de lo ingresado. 
      
## Alternativas de SQL

```sql
select comuna,
       sum(case when e2=3 then fexp else 0 end) as numerador,
       sum(fexp) as denominador
  from miembros
  where edad>=3
  group by comuna
  order by comuna;

select comuna, sum(fexp) as numerador,
  (select sum(fexp) from miembros z where edad>=3 and z.comuna=x.comuna) as denominador
  from miembros as x
  where edad>=3 and e2=3
  group by comuna
  order by comuna;
```
