## Servidores de Aplicaciones

### Instalación de Node.js y npm

Para implementar la aplicación backend, se procedió a instalar **Node.js** y su gestor de paquetes **npm** en ambas máquinas (**App1** y **App2**), asegurando así entornos equivalentes para distribución de carga y tolerancia a fallos.

Primero se actualizó el sistema y se instalaron los paquetes necesarios:

```bash
sudo apt update
sudo apt install nodejs npm -y
```
---

Posteriormente, se verificó la instalación con:

```bash
node -v
npm -v
```
Ambas máquinas quedaron preparadas para ejecutar aplicaciones desarrolladas con Node.js.

---

## Desarrollo de la aplicación CRUD con Node.js y Express

Se desarrolló una aplicación básica que implementa un **CRUD** (Create, Read, Update, Delete) utilizando el framework **Express**. La lógica principal se encuentra en el archivo `index.js`, donde se definen rutas para interactuar con una base de datos MySQL.

Código base (`index.js`):

```javascript
const express = require('express');
const os = require('os');
const mysql = require('mysql2');

const app = express();
const port = 3000;

const db = mysql.createConnection({
  host: '192.168.25.118',
  user: 'user',
  password: '12345',
  database: 'miappdb'
});

db.connect(err => {
  if (err) {
    console.error('Error al conectar a la base de datos:', err.message);
  } else {
    console.log('Conectado exitosamente a la base de datos');
  }
});

app.use(express.json());

app.get('/api/info', (req, res) => {
  const hostname = os.hostname();
  res.json({ mensaje: `Hola desde app de ${hostname}` });
});

app.get('/api/items', (req, res) => {
  db.query('SELECT * FROM items', (err, results) => {
    if (err) return res.status(500).send('Error al obtener los items');
    res.json(results);
  });
});

app.post('/api/items', (req, res) => {
  const { nombre, descripcion } = req.body;
  db.query('INSERT INTO items (nombre, descripcion) VALUES (?, ?)', [nombre, descripcion], (err, result) => {
    if (err) return res.status(500).send('Error al insertar el item');
    res.status(201).json({ id: result.insertId, nombre, descripcion });
  });
});

app.put('/api/items/:id', (req, res) => {
  const { id } = req.params;
  const { nombre, descripcion } = req.body;
  db.query('UPDATE items SET nombre = ?, descripcion = ? WHERE id = ?', [nombre, descripcion, id], (err) => {
    if (err) return res.status(500).send('Error al actualizar el item');
    res.json({ message: 'Item actualizado' });
  });
});

app.delete('/api/items/:id', (req, res) => {
  const { id } = req.params;
  db.query('DELETE FROM items WHERE id = ?', [id], (err) => {
    if (err) return res.status(500).send('Error al eliminar el item');
    res.json({ message: 'Item eliminado' });
  });
});

app.listen(port, '0.0.0.0', () => {
  console.log(`App escuchando en el puerto ${port}`);
});
```
Esta aplicación responde a solicitudes HTTP en formato JSON para operaciones de lectura, creación, actualización y eliminación de registros en la tabla items.

---

### Conexión a la base de datos MySQL

Se utilizó el módulo **mysql2** para establecer la conexión entre **Node.js** y la base de datos **MySQL**. El servidor **App2** se configuró como base de datos principal, con la IP `192.168.25.118`.

---

Para instalar el módulo en ambos servidores, se ejecutó:

```bash
npm install mysql2
```

---

## Configuración de Nginx como proxy inverso

Se configuró **Nginx** como proxy inverso en **App1**, para exponer la aplicación **Node.js** al cliente final. El objetivo fue enrutar todas las solicitudes a la ruta `/api/` hacia las instancias de **Node.js** que corren en el puerto `3000` de **App1** y **App2**.

Archivo de configuración del Server Block: `/etc/nginx/sites-available/miinfra`

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

Para aplicar los cambios se ejecutaron los siguientes comandos:

---

```bash
sudo ln -s /etc/nginx/sites-available/miinfra /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

De esta forma, cualquier petición enviada a http://miinfra.local/api/items se distribuye automáticamente entre ambas instancias de la aplicación, garantizando disponibilidad y balanceo de carga.