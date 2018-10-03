# clase 2

## ejemplos SQL usados en la clase

Para empezar hay que crear una base de datos llamada **clase6_db**

### transacciones

Imaginemos en una primera instancia una contabilidad sencilla de una entidad financiera que tiene una sucursal y tres clientes. Al empezar todos los clientes tienen depositados en la entidad 1000$. 

```sql
create table cuentas(
  cuenta integer primary key,
  nombre text
);

insert into cuentas(cuenta, nombre) values 
  (11, 'Sucursal 1'),
  (1, 'Cliente 1'),
  (2, 'Cliente 2'),
  (3, 'Cliente 3');

create table contabilidad(
  renglon serial primary key,
  cuenta integer,
  fecha date,
  importe decimal
);

insert into contabilidad (cuenta, fecha, importe) values 
  (1, '2018-09-01', 1000),
  (2, '2018-09-01', 1000),
  (3, '2018-09-01', 1000),
  (11, '2018-09-01', -3000);
```

Podemos revisar los saldos de las cuentas de contabilidad (en la sucursal hay 3000$ que es de los clientes), y la suma de toda la contabilidad da 0

```sql
select cuenta, sum(importe)
  from contabilidad
  group by cuenta
union
select null, sum(importe)
  from contabilidad
  order by cuenta;
```

Imaginemos luego que el cliente 1 transfiere 200$ al cliente 2. Para ello se registra el siguiente movimiento:

```sql
BEGIN TRANSACTION
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (1, '2018-09-02', -200);
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (2, '2018-09-02', 200);
COMMIT;
```

Queremos ver qué ocurre entre una instrucción y otra si no hubiéramos puesto el begin transaction. 

Para poder experimentar con la bifurcación del tiempo en la base de datos (coherencia y atomicidad) 
hay que ejecutar estas instrucciones desde la línea de comandos con `psql`.
Para eso hay que abrir una consola de comandos presionando ![win](../imagenes/win-key.png)![R](../imagenes/R-key.png)
y luego escribiendo **CMD** y presionando el botón **Ejecutar**. Luedo dentro de la pantalla escribir

```sh
cd c:\Program Files\PostgreSQL\9.5\bin
psql clase6_db postgres
```

Si todo está bien veremos que delante del cursor se ve **clase6_db=#**. 
Eso significa que estamos dentro de la base de datos.

Ejecutemos de a una las instrucciones que siguen, entre una y otra vayamos al pgAdmin para pedir el saldo:

```sql
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (1, '2018-09-03', -300);
-- Pedir los saldos desde el pgAdmin en este momento. Se ve la incoherencia
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (2, '2018-09-03', 300);
```

Al pedir el saldo entre ambas instrucciones hay una incoherencia (el dinero ya no está en el cliente 1 pero todavía no está en el 2).

```sql
select cuenta, sum(importe)
  from contabilidad
  group by cuenta
union
select null, sum(importe)
  from contabilidad
  order by cuenta;
```

Al colocarlo dentro de la transacción

```sql
BEGIN TRANSACTION; -- no en pgAdminIII ni en SQLFiddle 
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (1, '2018-09-04', -400);
-- Pedir los saldos desde el pgAdmin en este momento. No se ve el insert hasta que no haya commit. 
-- Hacer otra transferencia en las mismas cuentas
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (2, '2018-09-04', 400);
COMMIT; -- no en pgAdminIII ni en SQLFiddle 
```

Si por alguna razón quisiéramos que no se puedan realizar simultáneamente dos transacciones sobre la misma cuenta de origen podemos indicarlo explícitamente:

```sql
BEGIN TRANSACTION; -- no en pgAdminIII ni en SQLFiddle 
SELECT * FROM cuentas WHERE cuenta=1 FOR UPDATE;
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (1, '2018-09-04', -400);
-- Pedir los saldos desde el pgAdmin en este momento. No se ve el insert hasta que no haya commit. 
-- Hacer otra transferencia en las mismas cuentas
INSERT INTO contabilidad (cuenta, fecha, importe) VALUES (2, '2018-09-04', 400);
COMMIT; -- no en pgAdminIII ni en SQLFiddle 
```
