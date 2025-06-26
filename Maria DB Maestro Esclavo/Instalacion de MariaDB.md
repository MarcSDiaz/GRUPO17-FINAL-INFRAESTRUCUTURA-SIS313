# Instalación y Verificación de MariaDB, Creación de la Base de Datos

En el marco de la implementación de una arquitectura de replicación Maestro-Esclavo con MariaDB, es fundamental comenzar por la correcta instalación del sistema gestor de bases de datos en ambos servidores (**Maestro** y **Esclavo**).

---

## 1. Instalación de MariaDB

El primer paso consiste en instalar MariaDB en ambos servidores. Se utilizó la distribución Ubuntu Server 24.04 LTS, por lo que la instalación se realizó mediante `apt`.

En ambos servidores (**app1** como Maestro, **app2** como Esclavo) se ejecutaron los siguientes comandos:

```bash
sudo apt update
sudo apt install mariadb-server -y
```

Una vez finalizada la instalación, se inicia el servicio y se habilita para que arranque automáticamente con el sistema:

```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
---

## 2. Verificación del estado del servicio

Para asegurarse de que el servidor MariaDB esté funcionando correctamente, se ejecuta el siguiente comando:

```bash
sudo systemctl status mariadb
```
El estado debe indicar que el servicio está `"active (running)"`.

Adicionalmente, se puede verificar la versión instalada con:

```bash
mysql --version
```
---

## 3. Acceso inicial al cliente MariaDB

Para ingresar al cliente de línea de comandos de MariaDB se ejecuta:

```bash
sudo mysql -u root -p
```
La contraseña puede estar vacía en la instalación inicial si no se ha configurado aún.

---

## 4. Creación de la base de datos y tabla inicial

Dentro del cliente MariaDB en el servidor maestro (**app1**), se procede a crear la base de datos que será replicada y su tabla correspondiente:

```sql
CREATE DATABASE miappdb;
USE miappdb;
CREATE TABLE items (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100),
  descripcion TEXT
);
```
Esta base de datos miappdb y su tabla items serán utilizadas para comprobar el funcionamiento de la replicación una vez configurada la arquitectura Maestro-Esclavo.

---

## 5. Verificación de la tabla creada

Se puede verificar la existencia de la tabla y su estructura con los siguientes comandos:

```sql
SHOW TABLES;
DESCRIBE items;
```
Esto debe mostrar que existe una tabla llamada items con las columnas id, nombre y descripcion, donde id es una clave primaria autoincremental.