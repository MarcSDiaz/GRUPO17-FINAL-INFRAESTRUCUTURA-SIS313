# Instalación y Verificación de MariaDB

MariaDB es un sistema de gestión de bases de datos relacional de código abierto, derivado de MySQL. Es ampliamente utilizado para aplicaciones que requieren una base de datos confiable, escalable y de alto rendimiento. MariaDB es compatible con MySQL, por lo que suele ser una opción preferida en muchos proyectos por su licencia libre y su comunidad activa.

## Instalación de MariaDB

Para instalar MariaDB en un sistema basado en Ubuntu (como Ubuntu Server 24.04 LTS), utilizamos el gestor de paquetes `apt`, que permite descargar e instalar fácilmente el software desde los repositorios oficiales.

Los pasos para la instalación son:

```bash
sudo apt update
sudo apt install mariadb-server
