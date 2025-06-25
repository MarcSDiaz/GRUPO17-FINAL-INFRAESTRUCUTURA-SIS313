# Configuración de Replicación Maestro-Esclavo en MariaDB

La replicación Maestro-Esclavo es una técnica que permite mantener una copia exacta de una base de datos en un servidor esclavo, replicando en tiempo real los cambios realizados en el servidor maestro. Esta arquitectura es muy útil para mejorar la disponibilidad, distribuir cargas de lectura y realizar respaldos sin afectar la base de datos principal.

---

## 1. Preparativos Iniciales

Antes de iniciar la configuración, es necesario tener dos servidores MariaDB instalados y en funcionamiento: 

- **Maestro (app1):** Servidor principal que recibirá todas las escrituras.
- **Esclavo (app2):** Servidor que replicará los datos del maestro y responderá a consultas de solo lectura.

Ambos servidores deben tener conexión de red y poder comunicarse entre sí.

---

## 2. Configuración en el Servidor Maestro (app1)

### 2.1. Habilitar el registro binario (binlog)

El registro binario es fundamental para la replicación, ya que almacena todas las modificaciones realizadas en la base de datos.

Editar el archivo de configuración de MariaDB, usualmente ubicado en:

## 2.2 Crear un usuario para la replicación en el Maestro (app1)

Para que el servidor esclavo pueda conectarse y obtener los cambios en tiempo real, es necesario crear un usuario exclusivo para la replicación con los permisos adecuados.

Ingresamos a MariaDB en el maestro:

```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'tu_contraseña_segura';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;


## 2.3 Obtener la posición actual del binlog en el Maestro

Para que el servidor esclavo pueda iniciar la replicación desde un punto exacto y evitar la pérdida o duplicación de datos, es fundamental conocer la posición actual del registro binario (binlog) en el servidor maestro.

### ¿Qué es el binlog?

El binlog es un archivo que registra todas las operaciones que modifican la base de datos (INSERT, UPDATE, DELETE, etc.). La replicación se basa en la lectura de estos logs para reproducir los cambios en el esclavo.

### Pasos para obtener la posición actual:

1. **Bloquear las tablas para mantener la consistencia**

   Ejecutamos el siguiente comando para bloquear las tablas y evitar modificaciones mientras se realiza el volcado de la base de datos:

   ```sql
   FLUSH TABLES WITH READ LOCK;
