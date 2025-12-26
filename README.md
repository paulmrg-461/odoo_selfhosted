# Odoo Local Environment

This project provides a Dockerized environment for running Odoo locally, following the architectural guidelines defined in `ARCHITECTURE.md`.

## Prerequisites

- [Docker](https://www.docker.com/products/docker-desktop) installed and running.

## Quick Start

1. **Start the services:**
   Open a terminal in this directory and run:
   ```bash
   docker compose up -d
   ```

2. **Access Odoo:**
   Open your browser and navigate to: [http://localhost:8069](http://localhost:8069)

3. **Install Apps (Important):**
   When you first log in, Odoo is a "blank canvas". To get functionality like POS, Inventory, or Invoicing:
   - Go to the **Apps** menu on the main dashboard.
   - Click **Activate** on the apps you need (e.g., *Point of Sale*, *Inventory*, *Accounting*).
   - Odoo will install them and refresh the page.

## Project Structure

- `docker-compose.yml`: Defines the Odoo and PostgreSQL services.
- `config/odoo.conf`: Odoo configuration file (Master password, DB settings, etc.).
- `addons/`: Directory for custom Odoo modules.
- `ARCHITECTURE.md`: Project architecture and rules.

## Management Commands

- **Stop services:**
  ```bash
  docker compose down
  ```
- **View logs:**
  ```bash
  docker compose logs -f
  ```
- **Restart services:**
  ```bash
  docker compose restart
  ```

## Development

To add custom modules:
1. Place your module folder inside the `addons/` directory.
2. Restart the Odoo container: `docker compose restart web`.
3. Go to Apps in Odoo, click "Update Apps List", and install your module.

## Guía paso a paso: configurar un nuevo cliente

A continuación tienes los pasos y comandos para crear la base de datos, el rol y el usuario inicial de Odoo en este entorno Dockerizado. 

**IMPORTANTE:** Si ves un error de "role/user does not exist", es porque Odoo intenta conectarse a la base de datos con un usuario que aún no ha sido creado en PostgreSQL.

### 1. Levantar los servicios
Asegúrate de que los contenedores estén corriendo:
```bash
docker compose up -d
```

### 2. Crear el usuario en la Base de Datos (PostgreSQL)
Debemos entrar al contenedor de base de datos (`db`) y crear el usuario que Odoo usará. Usaremos las credenciales del superusuario definidas en `.env`.

Ejecuta el siguiente comando (puedes copiar y pegar todo el bloque en tu terminal):

```bash
# Carga las variables del .env (si estás en PowerShell, omite esta línea y asegúrate de que el contenedor esté usando las vars)
# En este proyecto, docker-compose ya inyecta las variables, así que usamos el usuario definido en .env

# Crear el usuario de base de datos para Odoo
docker compose exec db psql -U modware_user -d postgres -c "CREATE USER modware_user WITH PASSWORD 'modware_password' SUPERUSER;"
```

*Nota: Si el comando falla diciendo que el rol ya existe, puedes ignorarlo.*

### 3. Crear la Base de Datos
Ahora creamos la base de datos vacía propiedad de ese usuario:

```bash
docker compose exec db psql -U modware_user -d postgres -c "CREATE DATABASE modware_odoo OWNER modware_user;"
```

### 4. Inicializar Odoo
Ahora le decimos a Odoo que inicialice esa base de datos (instalará las tablas base):

```bash
docker compose exec web odoo -d modware_odoo -i base --stop-after-init
```

### 5. Acceder y crear el primer usuario Administrador
1. Abre [http://localhost:8069](http://localhost:8069).
2. Si te pide crear una base de datos, usa la **Master Password** que está en `config/odoo.conf` (por defecto `admin`).
3. Si ya ves la pantalla de Login, entra con `admin` / `admin` (o las credenciales que hayas definido al crear la DB).

### Solución de Problemas Comunes

**Error: `FATAL: role "modware_user" does not exist`**
- Significa que PostgreSQL no tiene ese usuario creado. Ejecuta el paso 2 de esta guía.

**Error: `database "modware_odoo" does not exist`**
- Significa que la base de datos no se creó. Ejecuta el paso 3.

**Permisos de carpetas (Linux/Mac)**
Si tienes problemas con los módulos, asegura los permisos:
```bash
sudo chown -R 1000:1000 addons config
sudo chmod -R u+rwX,go+rX addons config
```
