# Configuración de Odoo para Múltiples Clientes con Docker

Este tutorial te guía paso a paso para configurar múltiples instancias de Odoo con Docker, utilizando una configuración personalizada para cada cliente.

## Estructura del Proyecto

Primero, crea la estructura de carpetas necesaria:

```bash
mkdir odoo_multiclientes
cd odoo_multiclientes
mkdir -p config_cliente1 addons_cliente1 config_cliente2 addons_cliente2
```

Tu estructura de directorios será:

```
odoo_multiclientes/
├── config_cliente1/
├── addons_cliente1/
├── config_cliente2/
├── addons_cliente2/
```

## Archivos de Configuración

### Archivo `config_cliente1/odoo.conf`

```ini
[options]
db_host = db_cliente1
db_port = 5432
db_user = cliente1_user
db_password = cliente1_password
addons_path = /mnt/extra-addons
data_dir = /var/lib/odoo
```

### Archivo `config_cliente2/odoo.conf`

```ini
[options]
db_host = db_cliente2
db_port = 5432
db_user = cliente2_user
db_password = cliente2_password
addons_path = /mnt/extra-addons
data_dir = /var/lib/odoo
```

## Archivo `docker-compose.yaml`

Guarda el siguiente contenido en el archivo `docker-compose.yaml` dentro del directorio `odoo_multiclientes`:

```yaml
version: '3.7'

services:
  # Cliente 1
  odoo_cliente1:
    image: odoo:17.0
    container_name: odoo_cliente1
    restart: unless-stopped
    ports:
      - "8200:8069"
    volumes:
      - odoo-data-cliente1:/var/lib/odoo
      - ./config_cliente1:/etc/odoo
      - ./addons_cliente1:/mnt/extra-addons
    depends_on:
      - db_cliente1

  db_cliente1:
    image: postgres:16.0
    container_name: db_cliente1
    restart: unless-stopped
    environment:
      - POSTGRES_DB=cliente1_db
      - POSTGRES_USER=cliente1_user
      - POSTGRES_PASSWORD=cliente1_password
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - db-data-cliente1:/var/lib/postgresql/data

  # Cliente 2
  odoo_cliente2:
    image: odoo:17.0
    container_name: odoo_cliente2
    restart: unless-stopped
    ports:
      - "8201:8069"
    volumes:
      - odoo-data-cliente2:/var/lib/odoo
      - ./config_cliente2:/etc/odoo
      - ./addons_cliente2:/mnt/extra-addons
    depends_on:
      - db_cliente2

  db_cliente2:
    image: postgres:16.0
    container_name: db_cliente2
    restart: unless-stopped
    environment:
      - POSTGRES_DB=cliente2_db
      - POSTGRES_USER=cliente2_user
      - POSTGRES_PASSWORD=cliente2_password
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - db-data-cliente2:/var/lib/postgresql/data

volumes:
  odoo-data-cliente1:
  db-data-cliente1:
  odoo-data-cliente2:
  db-data-cliente2:
```

## Iniciar los Contenedores

1. Inicia los servicios con Docker Compose:
   ```bash
   docker-compose up -d
   ```

2. Verifica que los contenedores estén corriendo:
   ```bash
   docker ps
   ```

   Deberías ver los contenedores de Odoo y PostgreSQL para ambos clientes.

## Configurar las Instancias de Odoo

1. Accede a cada instancia en tu navegador:
   - Cliente 1: [http://localhost:8200](http://localhost:8200)
   - Cliente 2: [http://localhost:8201](http://localhost:8201)

2. En cada instancia, crea la base de datos correspondiente:
   - Nombre de la base de datos: `cliente1_db` o `cliente2_db`.
   - Configura el idioma, país y usuario administrador desde la interfaz de Odoo.

## Añadir Addons Personalizados

1. Coloca los módulos adicionales en las carpetas correspondientes:
   - `addons_cliente1` para Cliente 1.
   - `addons_cliente2` para Cliente 2.

2. Reinicia los servicios si es necesario:
   ```bash
   docker-compose restart
   ```

## Logs y Resolución de Problemas

1. Si encuentras problemas, revisa los logs de los contenedores:
   ```bash
   docker logs odoo_cliente1
   docker logs odoo_cliente2
   ```

2. Verifica las conexiones a la base de datos o posibles errores de configuración.

---

¡Listo! Ahora tienes múltiples instancias de Odoo configuradas para diferentes clientes con sus propias bases de datos y addons personalizados.
