## Configuración de la Replicación en el Servidor Esclavo (app2)

Con el servidor **Maestro** ya configurado y con el archivo dump de la base de datos transferido, se procede a preparar el servidor **Esclavo** (`app2`) para recibir y aplicar los cambios provenientes del Maestro. El objetivo es lograr una sincronización continua y automática de la base de datos `miappdb`.

---

## 1. Configurar MariaDB en el Esclavo

Primero se debe configurar el archivo de MariaDB para asignar un identificador único al servidor esclavo y prepararlo para la replicación.

Editar el archivo de configuración:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
---

Y dentro de la sección `[mysqld]` se agrega o modifica:

```ini
server-id = 2
```

El `server-id` debe ser diferente al del Maestro. En este caso, el valor **2** identifica al servidor Esclavo.
No se debe usar `log_bin` en el esclavo si este no va a actuar también como maestro (por ejemplo, en replicación en cadena).

Guardar los cambios y reiniciar MariaDB:

```bash
sudo systemctl restart mariadb
```
---

## 2. Restaurar la base de datos desde el dump

Es necesario importar el volcado de la base de datos que se generó en el **Maestro**. Esto asegura que el esclavo comience con una copia exacta del estado del maestro en el momento de iniciar la replicación.

Ejecutar:

```bash
sudo mysql -u root -p < /home/usuario/miappdb_dump.sql
```

Después de ingresar la contraseña, se restaurará la base de datos `miappdb`.

Es recomendable ingresar a MariaDB para verificar que la base se haya restaurado correctamente:

```bash
sudo mysql -u root -p
```

Dentro del cliente:

```sql
SHOW DATABASES;
USE miappdb;
SHOW TABLES;
```

---

## 3. Configurar la conexión con el Maestro

Con la base de datos ya restaurada, se procede a indicar al esclavo cómo conectarse al Maestro para iniciar la replicación.

Acceder al cliente MariaDB como root:

```bash
sudo mysql -u root -p
```

---

Detener cualquier replicación previa (por precaución):

```sql
STOP SLAVE;
```

---

Configurar la replicación (reemplazando los valores con los obtenidos desde el Maestro):

```sql
CHANGE MASTER TO
  MASTER_HOST='192.168.2.10',
  MASTER_USER='repl',
  MASTER_PASSWORD='54321',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;
```

`MASTER_HOST` debe ser la IP del Maestro (`app1`).
`MASTER_USER` y `MASTER_PASSWORD` son las credenciales del usuario creado en el Maestro para replicación.
`MASTER_LOG_FILE` y `MASTER_LOG_POS` se obtuvieron con `SHOW MASTER STATUS` en el Maestro o están incluidos en el dump si se usó `--master-data`.

---

Iniciar el proceso de replicación:

```sql
START SLAVE;
```

---

## 4. Verificar el estado de la replicación

Para comprobar que la replicación está funcionando correctamente, ejecutar:

```sql
SHOW SLAVE STATUS\G
```

En la salida, verificar que:

* `Slave_IO_Running: Yes`
* `Slave_SQL_Running: Yes`
* `Seconds_Behind_Master` muestra `0` o un número bajo
* `Last_Error` esté vacío

Si ambos hilos (IO y SQL) están activos y no hay errores, la replicación está funcionando correctamente.

---

## Prueba de Replicación

Para comprobar que la replicación se realiza correctamente:

En el servidor **Maestro**, insertar un nuevo dato:

```sql
USE miappdb;
INSERT INTO items (nombre, descripcion) VALUES ('Producto A', 'Este producto fue insertado desde el maestro');
```

---

En el servidor **Esclavo**, verificar que el dato se haya replicado:

```sql
SELECT * FROM items;
```

El nuevo registro debería estar presente en la tabla `items` del esclavo, confirmando que la replicación **Maestro-Esclavo** funciona correctamente.
