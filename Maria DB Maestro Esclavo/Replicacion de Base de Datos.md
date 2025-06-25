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

```ini
/etc/mysql/mariadb.conf.d/50-server.cnf