## Interfaz HTML CRUD

### Objetivo

Diseñar e implementar una interfaz web básica que permita a los usuarios interactuar con la aplicación Node.js mediante operaciones CRUD (Crear, Leer, Actualizar y Eliminar) sobre una base de datos MySQL.

Esta interfaz también permite visualizar el servidor que está respondiendo, lo cual es útil para validar el balanceo de carga.

### Ubicación y acceso

La interfaz fue implementada en un archivo llamado `index.html`, ubicado en el siguiente directorio del servidor App1 (donde corre Nginx como proxy inverso):

```bash
/var/www/html/miinfra/index.html

```
Este archivo se sirve directamente desde Nginx como contenido estático y es accesible a través del navegador en:

```bash
http://miinfra.local

```
### Contenido del archivo `index.html`

A continuación se presenta una versión simplificada del código utilizado:

### Contenido del archivo `index.html`

A continuación se presenta una versión simplificada del código utilizado:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>CRUD Simple</title>
  <style>
    body { font-family: Arial; margin: 20px; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; }
    th, td { border: 1px solid #aaa; padding: 8px; text-align: left; }
    th { background-color: #eee; }
  </style>
</head>
<body>

<h2>CRUD de Items</h2>
<h3 id="hostinfo">Cargando información del host...</h3>

<form id="itemForm">
  <input type="text" id="nombre" placeholder="Nombre" required>
  <input type="text" id="descripcion" placeholder="Descripción" required>
  <button type="submit">Agregar</button>
</form>

<table id="itemsTable">
  <thead>
    <tr>
      <th>ID</th><th>Nombre</th><th>Descripción</th><th>Acciones</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script>
const API = '/api/items';

function cargarItems() {
  fetch(API)
    .then(res => res.json())
    .then(data => {
      const tbody = document.querySelector('#itemsTable tbody');
      tbody.innerHTML = '';
      data.forEach(item => {
        tbody.innerHTML += `
          <tr>
            <td>${item.id}</td>
            <td>${item.nombre}</td>
            <td>${item.descripcion}</td>
            <td><button onclick="eliminarItem(${item.id})">Eliminar</button></td>
          </tr>`;
      });
    });
}

function eliminarItem(id) {
  fetch(`/api/items/${id}`, { method: 'DELETE' })
    .then(() => cargarItems());
}

document.getElementById('itemForm').addEventListener('submit', e => {
  e.preventDefault();
  const nombre = document.getElementById('nombre').value;
  const descripcion = document.getElementById('descripcion').value;

  fetch('/api/items', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ nombre, descripcion })
  }).then(() => {
    document.getElementById('itemForm').reset();
    cargarItems();
  });
});

fetch('/api/info')
  .then(res => res.json())
  .then(data => {
    document.getElementById('hostinfo').innerText = data.mensaje;
  });

cargarItems();
</script>

</body>
</html>
```

### Funcionalidad

- `GET /api/items`: Carga y muestra todos los ítems registrados.
- `POST /api/items`: Agrega un nuevo ítem al formulario.
- `DELETE /api/items/:id`: Elimina un ítem seleccionado.
- `GET /api/info`: Consulta qué servidor está respondiendo (App1 o App2), útil para verificar el balanceo de carga en tiempo real.

### Observaciones

- La interfaz consume directamente la API expuesta por el servidor Node.js, accediendo a través del proxy `/api/`.
- Toda la interacción con el backend se realiza utilizando `fetch` y el formato `JSON`.
- El CRUD es funcional y fue probado tanto en App1 como en App2, utilizando una base de datos replicada.
- La visualización del host activo ayuda a evidenciar que el balanceador alterna entre ambos servidores.


