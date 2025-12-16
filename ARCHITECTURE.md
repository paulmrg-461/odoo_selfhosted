# Architecture Documentation

## Project Overview
This project is a local development environment for Odoo using Docker. It aims to provide a robust, scalable, and easy-to-manage setup for running Odoo and its dependencies.

## Architecture Pattern
**Containerized Architecture**: The application and its dependencies are encapsulated in Docker containers to ensure consistency across different environments.

## Technology Stack
- **Application**: Odoo Community Edition
  - Version: 19.0
  - Image: `odoo:19.0`
- **Database**: PostgreSQL
  - Version: 15
  - Image: `postgres:15`
- **Orchestration**: Docker Compose

## Infrastructure
- **Docker Compose**: Defines the services, networks, and volumes.
- **Volumes**:
  - `odoo-web-data`: Persists Odoo session data and filestore.
  - `odoo-db-data`: Persists PostgreSQL database data.
  - `./config`: Host-mounted directory for Odoo configuration files.
  - `./addons`: Host-mounted directory for custom Odoo addons.

## Development Rules
1. **Configuration**: All Odoo configuration changes should be made in `config/odoo.conf` or via environment variables in `docker-compose.yml`.
2. **Data Persistence**: Never store critical data inside containers without volumes.
3. **Addons**: Custom modules must be placed in the `addons` directory.
