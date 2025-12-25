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

3. **Create a Database:**
   - **Master Password:** `admin` (defined in `config/odoo.conf`)
   - Fill in the database name, email, and password for your admin user.
   - Click "Create Database".

4. **Install Apps (Important):**
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

## Pasos iniciales para un nuevo cliente (rápido)

A continuación tienes los pasos y comandos para crear la base de datos, el rol y el usuario inicial de Odoo en este entorno Dockerizado. Reemplaza los nombres/contraseñas donde indico.

1) Fijar la contraseña maestra (admin database password)

- Edita `config/odoo.conf` y asegura la línea:
   - `admin_passwd = TU_CONTRASENA_MAESTRA`
- Ejemplo desde el host:
```bash
sed -i "s/^admin_passwd.*/admin_passwd = SuperSecreta123/" config/odoo.conf || echo "admin_passwd = SuperSecreta123" >> config/odoo.conf
```

2) Crear el rol (usuario) y la base de datos PostgreSQL

Reemplaza `odoo_user`, `odoo_pass`, `odoo_db` por tus valores:
```bash
docker compose exec -T db psql -U postgres -c "CREATE USER odoo_user WITH PASSWORD 'odoo_pass';"
docker compose exec -T db psql -U postgres -c "CREATE DATABASE odoo_db OWNER odoo_user;"
docker compose exec -T db psql -U postgres -d odoo_db -c "GRANT ALL PRIVILEGES ON DATABASE odoo_db TO odoo_user;"
```

3) Inicializar la base de datos Odoo (instalar `base`)

Esto crea las tablas principales (ir_module_module, ir.http, etc.):
```bash
docker compose exec -T web odoo -d odoo_db -i base --without-demo=all --stop-after-init -c /etc/odoo/odoo.conf
```

4) Crear el usuario interno de Odoo

Opción A — Interfaz web: abre `http://localhost:8069` y crea el usuario desde Configuración → Usuarios.

Opción B — `odoo shell` no interactivo:
```bash
docker compose exec -T web odoo shell -d odoo_db -c /etc/odoo/odoo.conf <<'PY'
from odoo import api, SUPERUSER_ID, registry
db = 'odoo_db'
with registry(db).cursor() as cr:
      env = api.Environment(cr, SUPERUSER_ID, {})
      user = env['res.users'].create({
            'name': 'Cliente Demo',
            'login': 'cliente@example.com',
            'password': 'ClienteP4ss!',
      })
      cr.commit()
print('Usuario creado:', user.login)
PY
```

5) Corregir el warning de `addons` (si aparece: "invalid addons directory '/mnt/extra-addons'")

```bash
sudo chown -R 1000:1000 addons
sudo chmod -R u+rwX,go+rX addons
docker compose restart web
```

6) Verificaciones útiles

```bash
docker compose exec -T db psql -U odoo_user -d odoo_db -c "SELECT count(*) FROM ir_module_module;"
curl -I http://localhost:8069
```

Notas:
- Guarda `admin_passwd` en `config/odoo.conf` de forma segura.
- Para producción: usa contraseñas robustas y backups regulares.
