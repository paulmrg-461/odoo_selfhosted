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
- `requirements.txt`: Python dependencies for development (debugging, linting, analysis).
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
- **Install Python Requirements (inside container):**
  If you need to use the debug tools or extra libraries:
  ```bash
  docker compose exec web pip install -r /mnt/extra-addons/requirements.txt
  ```
  *(Note: You might need to mount the requirements file or copy it depending on your exact volume setup. For quick use, just run `pip install <package>` inside the container)*

## Development

To add custom modules:
1. Place your module folder inside the `addons/` directory.
2. Restart the Odoo container: `docker compose restart web`.
3. Go to Apps in Odoo, click "Update Apps List", and install your module.
