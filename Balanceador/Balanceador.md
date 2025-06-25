# Balanceador de Carga con Nginx

## Objetivo

Distribuir las peticiones HTTP entre dos aplicaciones Node.js (App1 y App2) utilizando **Nginx como proxy inverso**, mejorando así la disponibilidad, tolerancia a fallos y escalabilidad del servicio web.

---

## Infraestructura

| Rol      | IP              | Servicios Activos             |
|----------|------------------|-------------------------------|
| App1     | 192.168.2.10     | Node.js, Nginx, MySQL (esclavo) |
| App2     | 192.168.2.118    | Node.js, MySQL (maestro)      |

Ambas máquinas ejecutan una instancia idéntica de una aplicación Node.js CRUD en el puerto `3000`.

---

## Configuración del Balanceador (Nginx)

Archivo: `/etc/nginx/sites-available/miinfra`

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

## Verificación del Balanceo

Ambas apps incluyen el siguiente endpoint en `index.js`:

```js
app.get('/api/info', (req, res) => {
  const hostname = os.hostname();
  res.json({ mensaje: `Hola desde app de ${hostname}` });
});
