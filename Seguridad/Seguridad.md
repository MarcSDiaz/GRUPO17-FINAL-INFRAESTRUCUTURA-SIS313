### Hardening y Seguridad

El proceso de *hardening* o endurecimiento se aplicó sobre los servicios críticos del sistema: `SSH`, `Apache`, `Nginx` y `Node.js`, con el objetivo de:

- Reducir la superficie de ataque.
- Proteger la información sensible.
- Controlar el acceso al sistema.

### Cambio de puertos por defecto en servicios críticos

Se modificaron los puertos por defecto para dificultar ataques automatizados que escanean puertos comunes.

#### SSH

El puerto por defecto `22` fue cambiado por otro personalizado (por ejemplo `1026`) en el archivo de configuración:

```bash
/etc/ssh/sshd_config

```
```bash
Port 1026
PermitRootLogin no
PasswordAuthentication no
```
Luego, se reinició el servicio SSH:

```bash
sudo systemctl restart ssh
```
Y se permitió el nuevo puerto en el firewall UFW:

```bash
sudo ufw allow 1026/tcp
```

### Autenticación por clave pública y deshabilitación del acceso root

Para proteger el acceso remoto:

- Se generó un par de claves con `ssh-keygen` en la máquina cliente.
- Se copió la clave pública al servidor con el siguiente comando:

```bash
ssh-copy-id -p 1026 usuario@ip_del_servidor
```

En el servidor, se deshabilitó el acceso root y la autenticación por contraseña en el archivo `/etc/ssh/sshd_config`:

```bash
PermitRootLogin no
PasswordAuthentication no
```

Esto garantiza que solo usuarios autorizados con clave pública puedan ingresar.

### Configuración del firewall (UFW)

Se configuró el firewall UFW (*Uncomplicated Firewall*) para permitir solo los servicios necesarios en cada máquina.

```bash
sudo ufw allow 1026/tcp      # Puerto SSH personalizado  
sudo ufw allow 80/tcp        # Puerto HTTP para Apache/Nginx  
sudo ufw allow 3000/tcp      # Puerto de la app Node.js  
sudo ufw enable              # Activar UFW  
sudo ufw status              # Verificar reglas activas
```

Esto limita el acceso únicamente a los servicios esenciales.

### Cabeceras de seguridad en servidores web

Se configuraron cabeceras de seguridad HTTP tanto en `Apache` como en `Nginx` para proteger a los clientes ante ataques comunes como *Clickjacking* y *Sniffing*.

Estas cabeceras ayudan a reforzar la política de seguridad del sitio web, evitando la exposición de información sensible y mejorando la privacidad del usuario.

En Nginx (`/etc/nginx/sites-available/miinfra`), se añadieron las siguientes cabeceras de seguridad:

```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
```

Opcionalmente, también se puede incluir la siguiente cabecera en la configuración de Nginx para reforzar aún más la seguridad:

```nginx
add_header Content-Security-Policy "default-src 'self'";
```

Estas directivas impiden que tu aplicación sea embebida en sitios externos, bloquean la interpretación incorrecta de tipos MIME y restringen el origen del contenido cargado, fortaleciendo así la política de seguridad del navegador.

### Observaciones

- Se aplicó un esquema mínimo de privilegios y exposición de puertos.
- Todos los cambios fueron probados y validados en ambas máquinas.
- En caso de fallo de una app o servicio, los mecanismos de *failover* y las reglas del firewall permiten que la operación continúe de forma segura.
