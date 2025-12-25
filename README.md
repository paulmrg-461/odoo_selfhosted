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

Reemplaza `odoo_user`, `odoo_pass`, `odoo_db` por tus valores. Usa el superusuario definido en `.env` (variable `POSTGRES_USER`) en lugar de `postgres`:
```bash
# Reemplaza valores: <db_user> <db_pass> <db_name>
docker compose exec -T db psql -U ${POSTGRES_USER} -d postgres -c "CREATE USER <db_user> WITH PASSWORD '<db_pass>';"
docker compose exec -T db psql -U ${POSTGRES_USER} -d postgres -c "CREATE DATABASE <db_name> OWNER <db_user>;"
docker compose exec -T db psql -U ${POSTGRES_USER} -d <db_name> -c "GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <db_user>;"
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

## Guía paso a paso: configurar un nuevo cliente en un equipo nuevo

Estos pasos asumen que estás desplegando este repositorio en una máquina nueva y quieres preparar un nuevo cliente (base de datos + usuario Odoo). Sustituye los valores entre `<>` por los reales.

1) Requisitos en la máquina nueva

- Instala Docker y Docker Compose.
- Clona este repositorio y sitúate en la carpeta del proyecto.

```bash
git clone <repo-url>
cd odoo_selfhosted
```

2) Revisar y preparar variables de entorno

- Crea o actualiza `.env` con las credenciales PostgreSQL. Ejemplo:

```
POSTGRES_DB=modware_odoo
POSTGRES_USER=modware_user
POSTGRES_PASSWORD=modware_password
```

3) Ajustar `config/odoo.conf`

- Edita `config/odoo.conf` y fija la contraseña maestra (válida para crear/gestionar DB desde la UI):

```
[options]
admin_passwd = <TU_CONTRASENA_MAESTRA>
db_host = db
db_port = 5432
db_user = ${POSTGRES_USER}
db_password = ${POSTGRES_PASSWORD}
addons_path = /mnt/extra-addons
```

4) Asegurar el directorio `addons` y `config`

- Crea la carpeta `addons/` si no existe y asigna permisos que el contenedor Odoo pueda leer (UID 1000 normalmente):

```bash
mkdir -p addons
sudo chown -R 1000:1000 addons config
sudo chmod -R u+rwX,go+rX addons config
```

5) Levantar los servicios Docker

```bash
docker compose up -d
```

6) (Opcional) Crear rol y base de datos PostgreSQL para el cliente

- En una instalación nueva debes crear el rol/DB si no existen. Usa el superusuario definido en `.env` (por ejemplo `modware_user`) para ejecutar estos comandos:

```bash
# Reemplaza valores: <db_user> <db_pass> <db_name>
docker compose exec -T db psql -U ${POSTGRES_USER} -d postgres -c "CREATE USER <db_user> WITH PASSWORD '<db_pass>';"
docker compose exec -T db psql -U ${POSTGRES_USER} -d postgres -c "CREATE DATABASE <db_name> OWNER <db_user>;"
docker compose exec -T db psql -U ${POSTGRES_USER} -d postgres -c "GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <db_user>;"
```

7) Inicializar la base de datos Odoo (instalar `base`)

- Si la base está vacía, inicialízala instalando el módulo `base`:

```bash
docker compose exec -T web odoo -d <db_name> -i base --without-demo=all --stop-after-init -c /etc/odoo/odoo.conf
```

8) Crear el usuario administrativo de Odoo

- Opción A — Interfaz web (recomendada):
   - Abre `http://<host>:8069`, usa la contraseña maestra cuando el asistente de creación de DB la pida y crea el administrador.

- Opción B — `odoo shell` (no interactivo):

```bash
docker compose exec -T web odoo shell -d <db_name> -c /etc/odoo/odoo.conf <<'PY'
from odoo import api, SUPERUSER_ID, registry
db = '<db_name>'
with registry(db).cursor() as cr:
      env = api.Environment(cr, SUPERUSER_ID, {})
      user = env['res.users'].create({
            'name': 'Admin Cliente',
            'login': 'admin@cliente.local',
            'password': 'AdminP4ss!'
      })
      cr.commit()
print('Usuario creado:', user.login)
PY
```

9) Verificaciones y resolución de problemas

- Ver logs de `web` y `db`:

```bash
docker compose logs -f web
docker compose logs -f db
```

- Comprobar tablas principales en la DB:

```bash
docker compose exec -T db psql -U ${POSTGRES_USER} -d <db_name> -c "SELECT count(*) FROM ir_module_module;"
```

- Si ves el warning "invalid addons directory '/mnt/extra-addons'", asegúrate de que `addons/` existe y es legible por UID 1000 (paso 4).

---

### Nota importante: uso correcto de `psql` y error "role -d does not exist"

Si al ejecutar comandos dentro del contenedor `db` ves un error del tipo `role "-d" does not exist` o `role "postgres" does not exist`, normalmente se debe a:

- Orden incorrecta de los flags/argumentos a `psql` (las opciones como `-U` y `-d` deben ir antes del `-c` o del comando). 
- Problemas de comillas cuando se usa `docker compose exec` (la shell del host interpreta partes del comando).

Regla práctica: siempre pasar las opciones de `psql` primero, luego la base de datos (si aplica) y por último `-c "<SQL o meta-comando>"`.

Ejemplos corregidos (usa `${POSTGRES_USER}` definido en `.env`):

```bash
# Listar roles
docker compose exec -T db psql -U "${POSTGRES_USER}" -d postgres -c "\du"

# Listar bases de datos
docker compose exec -T db psql -U "${POSTGRES_USER}" -d postgres -c "\l"

# Crear rol y base de datos (reemplaza <db_user> <db_pass> <db_name>)
docker compose exec -T db psql -U "${POSTGRES_USER}" -d postgres -c "CREATE USER <db_user> WITH PASSWORD '<db_pass>';"
docker compose exec -T db psql -U "${POSTGRES_USER}" -d postgres -c "CREATE DATABASE <db_name> OWNER <db_user>;"
docker compose exec -T db psql -U "${POSTGRES_USER}" -d <db_name> -c "GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <db_user>;"

# Conectar (ejemplo) para listar tablas en una BD
docker compose exec -T db psql -U "${POSTGRES_USER}" -d <db_name> -c "\dt"
```

Consejo: si tu `.env` define `POSTGRES_USER=modware_user`, puedes usar `-U modware_user` directamente. Evita ejecutar `psql` sin `-U` si el rol `postgres` no existe en tu imagen.


10) Buenas prácticas

- No uses `admin_passwd` débil en producción.
- Haz backups regulares (`pg_dump`) y prueba restauraciones.
- Controla los permisos de `addons/` cuando subas módulos externos.

Si quieres, puedo adaptar esta guía a tus valores concretos y ejecutar los comandos aquí mismo (crear DB/usuario, inicializar y crear admin). Indícame los nombres/contraseñas que quieres usar o confirma que prefieres solo la guía.
