## Balanceador de Carga con Nginx

### Objetivo

Implementar un **balanceador de carga** en la máquina **App1** utilizando **Nginx** como proxy inverso, con el fin de distribuir las solicitudes entre las dos aplicaciones **Node.js** (**App1** y **App2**) que corren en el puerto `3000`. Esta estrategia mejora la **disponibilidad**, la **tolerancia a fallos** y la **escalabilidad** del sistema.

## Infraestructura Utilizada

Rol	IP	Servicios Activos
App1	192.168.25.10	Node.js, Nginx (balanceador), MySQL
App2	192.168.25.118	Node.js, MySQL

---

## Configuración del Balanceador (Nginx)

En **App1** se editó un nuevo archivo de configuración para Nginx:

```bash
sudo nano /etc/nginx/sites-available/miinfra
```

Contenido del archivo:

```nginx
upstream backend_nodes {
    server 192.168.25.10:3000;
    server 192.168.25.118:3000;
}

server {
    listen 80;
    server_name miinfra.local;

    location /api/ {
        proxy_pass http://backend_nodes;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    root /var/www/html/miinfra;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Este archivo define el grupo de servidores backend (backend_nodes) que serán balanceados por Nginx, y establece reglas para enrutar tráfico hacia las rutas /api/.

---

## Activación de la configuración

Para activar la configuración del balanceador:

```bash
sudo ln -s /etc/nginx/sites-available/miinfra /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Estas acciones habilitaron el balanceador de carga y recargaron Nginx para aplicar los cambios.

---

## Endpoint para prueba de balanceo

Cada aplicación Node.js (en App1 y App2) implementa el siguiente endpoint:

```javascript
app.get('/api/info', (req, res) => {
  const hostname = os.hostname();
  res.json({ mensaje: `Hola desde app de ${hostname}` });
});
```

Este endpoint permite verificar visualmente desde qué servidor se está respondiendo la petición.

---

## Verificación del balanceo

Desde cualquier navegador o terminal, se pueden ejecutar múltiples peticiones:

```bash
curl http://miinfra.local/api/info
```

### Resultado esperado

Una respuesta alternada como:

```nginx
Hola desde app de app1
Hola desde app de app2
```

Esto demuestra que el balanceador está utilizando un algoritmo de round-robin para distribuir las peticiones entre ambas aplicaciones.

### Observaciones

- Nginx escucha en el puerto 80 y actúa como punto de entrada para el sistema.
- Las dos instancias de Node.js están corriendo en el puerto 3000 de cada máquina.
- El balanceador permite tolerancia a fallos básica: si una instancia deja de responder, la otra puede seguir funcionando.
- Se configuró una interfaz HTML que, al consultar `/api/info`, muestra dinámicamente desde qué host se responde la petición.

