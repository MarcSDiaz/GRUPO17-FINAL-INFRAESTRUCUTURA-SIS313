# Configuración en el Servidor Esclavo (app2)

La configuración del servidor esclavo es esencial para implementar la replicación Maestro-Esclavo en MariaDB. El esclavo se encargará de mantener una copia sincronizada de la base de datos del maestro, replicando en tiempo real los cambios que se realicen. Este proceso permite distribuir la carga de consultas y aumentar la disponibilidad del sistema.

---

## 4.1 Restaurar la base de datos desde el dump

Antes de configurar la replicación, es indispensable que el esclavo tenga una copia exacta y consistente de la base de datos del maestro en el momento de iniciar la replicación.

### Proceso para restaurar la base de datos:

1. **Transferir el dump desde el maestro al esclavo**

   Desde el maestro, utilizando una herramienta segura como `scp` o `rsync`, se copia el archivo del volcado generado:

   ```bash
   scp miappdb_dump.sql usuario@app2:/home/usuario/
