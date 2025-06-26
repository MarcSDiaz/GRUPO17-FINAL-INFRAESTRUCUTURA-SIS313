## Configuración de la Replicación en el Servidor Maestro (app1)

Una vez instalado y verificado el funcionamiento de MariaDB, y creada la base de datos `miappdb` con su respectiva tabla, se procede a configurar el servidor **Maestro** (`app1`) para la replicación de datos hacia un servidor **Esclavo** (`app2`). Esta configuración permitirá que todas las modificaciones realizadas en la base de datos del Maestro sean replicadas en tiempo real al Esclavo.

---

## 1. Configurar MariaDB para habilitar replicación

El primer paso consiste en editar el archivo de configuración de MariaDB para habilitar el registro binario (**binlog**), el cual es necesario para que el esclavo pueda leer los eventos de replicación.

Se edita el siguiente archivo:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
---

Dentro de la sección `[mysqld]`, se agregan o modifican las siguientes líneas:

```ini
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = miappdb
```
`server-id` debe ser un valor único en la red. En este caso, **1** identifica al servidor Maestro.
`log_bin` habilita el binlog, que registra todas las operaciones que modifican datos.
`binlog_do_db` indica cuál base de datos será replicada.

Luego se guarda el archivo y se reinicia el servicio MariaDB:

```bash
sudo systemctl restart mariadb
```
---

## 2. Crear el usuario para replicación

Se debe crear un usuario que será utilizado por el servidor esclavo para conectarse al maestro y leer los eventos del binlog.

Ingresar al cliente MariaDB:

```bash
sudo mysql -u root -p
```
Y ejecutar:

```sql
CREATE USER 'repl'@'%' IDENTIFIED BY '54321';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

`'repl'@'%'` permite la conexión desde cualquier IP (se puede reemplazar `%` por la IP específica del esclavo para mayor seguridad).
`GRANT REPLICATION SLAVE` otorga los permisos necesarios para la replicación.

---

## 3. Obtener la posición actual del binlog

Para que el esclavo comience a replicar correctamente, necesita saber desde qué archivo binlog y en qué posición debe empezar a leer. Esta información se obtiene al ejecutar el siguiente procedimiento:

Bloquear las tablas temporalmente para mantener la consistencia:

```sql
FLUSH TABLES WITH READ LOCK;
```
Obtener la información del binlog:

```sql
SHOW MASTER STATUS;
```
El resultado será algo como:

```makefile
File: mysql-bin.000001
Position: 154
```
Es importante **no cerrar esta sesión** mientras se mantiene el lock. Se puede abrir otra terminal para continuar con los siguientes pasos.

---

## 4. Crear un volcado (dump) de la base de datos

Se debe crear una copia exacta de la base de datos desde el **Maestro** para luego restaurarla en el **Esclavo**:

```bash
mysqldump -u root -p --databases miappdb --master-data > miappdb_dump.sql
```
El parámetro `--master-data` incluye automáticamente en el archivo `.sql` una línea con la posición del binlog, útil para el esclavo.

---

## 5. Transferir el archivo al esclavo

El archivo generado debe copiarse al servidor esclavo para ser restaurado:

```bash
scp miappdb_dump.sql usuario@app2:/home/usuario/
```
Una vez completados estos pasos, el servidor **Maestro** estará correctamente configurado para permitir que un esclavo se conecte y comience a replicar los datos.
