# Configuración de Odoo para Múltiples Clientes con Docker

Este tutorial detalla cómo configurar dos instancias separadas de Odoo con PostgreSQL utilizando Docker Compose. El archivo YAML está completamente comentado para facilitar su comprensión.

---

## Archivo `docker-compose.yaml`

```yaml
version: '3.7'

services:

  # Cliente 1: Servicios relacionados con el cliente 1

  odoo_cliente1:
    image: odoo:17.0                  # Imagen oficial de Odoo versión 17
    container_name: odoo_cliente1     # Nombre del contenedor para identificarlo fácilmente
    restart: unless-stopped           # Reinicia el contenedor automáticamente si se detiene
    links:
      - db_cliente1:db               # Vincula este contenedor con el servicio de base de datos "db_cliente1"
    depends_on:
      - db_cliente1                  # Asegura que la base de datos se inicie antes que Odoo
    ports:
      - "8200:8069"                # Mapea el puerto 8069 del contenedor al 8200 del host para acceder a la instancia
    volumes:
      - odoo-data-cliente1:/var/lib/odoo   # Volumen para almacenar datos persistentes de Odoo
      - ./config_cliente1:/etc/odoo        # Monta la configuración personalizada de Odoo para el cliente 1
      - ./addons_cliente1:/mnt/extra-addons # Monta la carpeta de addons personalizados del cliente 1

  db_cliente1:
    image: postgres:16.0              # Imagen oficial de PostgreSQL versión 16
    container_name: db_cliente1       # Nombre del contenedor para identificarlo fácilmente
    restart: unless-stopped           # Reinicia el contenedor automáticamente si se detiene
    environment:
      - POSTGRES_DB=cliente1_db       # Nombre de la base de datos para el cliente 1
      - POSTGRES_USER=cliente1_user   # Usuario de la base de datos
      - POSTGRES_PASSWORD=cliente1_password # Contraseña de la base de datos
      - PGDATA=/var/lib/postgresql/data/pgdata # Ruta de almacenamiento de datos de PostgreSQL
    volumes:
      - db-data-cliente1:/var/lib/postgresql/data # Volumen para almacenar datos persistentes de la base de datos

  # Cliente 2: Servicios relacionados con el cliente 2

  odoo_cliente2:
    image: odoo:17.0                  # Imagen oficial de Odoo versión 17
    container_name: odoo_cliente2     # Nombre del contenedor para identificarlo fácilmente
    restart: unless-stopped           # Reinicia el contenedor automáticamente si se detiene
    links:
      - db_cliente2:db               # Vincula este contenedor con el servicio de base de datos "db_cliente2"
    depends_on:
      - db_cliente2                  # Asegura que la base de datos se inicie antes que Odoo
    ports:
      - "8201:8069"                # Mapea el puerto 8069 del contenedor al 8201 del host para acceder a la instancia
    volumes:
      - odoo-data-cliente2:/var/lib/odoo   # Volumen para almacenar datos persistentes de Odoo
      - ./config_cliente2:/etc/odoo        # Monta la configuración personalizada de Odoo para el cliente 2
      - ./addons_cliente2:/mnt/extra-addons # Monta la carpeta de addons personalizados del cliente 2

  db_cliente2:
    image: postgres:16.0              # Imagen oficial de PostgreSQL versión 16
    container_name: db_cliente2       # Nombre del contenedor para identificarlo fácilmente
    restart: unless-stopped           # Reinicia el contenedor automáticamente si se detiene
    environment:
      - POSTGRES_DB=cliente2_db       # Nombre de la base de datos para el cliente 2
      - POSTGRES_USER=cliente2_user   # Usuario de la base de datos
      - POSTGRES_PASSWORD=cliente2_password # Contraseña de la base de datos
      - PGDATA=/var/lib/postgresql/data/pgdata # Ruta de almacenamiento de datos de PostgreSQL
    volumes:
      - db-data-cliente2:/var/lib/postgresql/data # Volumen para almacenar datos persistentes de la base de datos

volumes:
  odoo-data-cliente1:                 # Volumen para almacenar los datos de Odoo del cliente 1
  db-data-cliente1:                   # Volumen para almacenar los datos de PostgreSQL del cliente 1
  odoo-data-cliente2:                 # Volumen para almacenar los datos de Odoo del cliente 2
  db-data-cliente2:                   # Volumen para almacenar los datos de PostgreSQL del cliente 2
```

---

## Estructura del Proyecto

1. **Directorio principal:** `odoo_multiclientes/`
   - Carpeta base donde estará el archivo `docker-compose.yaml`.
2. **Subdirectorios de configuración y addons:**
   ```
   odoo_multiclientes/
   ├── config_cliente1/   # Configuración personalizada para el cliente 1
   ├── addons_cliente1/   # Addons personalizados para el cliente 1
   ├── config_cliente2/   # Configuración personalizada para el cliente 2
   ├── addons_cliente2/   # Addons personalizados para el cliente 2
   ```

---

## Archivos de Configuración

### `config_cliente1/odoo.conf`
```ini
[options]
db_host = db_cliente1                # Servicio de base de datos para el cliente 1
db_port = 5432                       # Puerto estándar de PostgreSQL
db_user = cliente1_user              # Usuario de conexión a la base de datos del cliente 1
db_password = cliente1_password      # Contraseña del usuario de la base de datos del cliente 1
addons_path = /mnt/extra-addons      # Ruta para los addons personalizados
```

### `config_cliente2/odoo.conf`
```ini
[options]
db_host = db_cliente2                # Servicio de base de datos para el cliente 2
db_port = 5432                       # Puerto estándar de PostgreSQL
db_user = cliente2_user              # Usuario de conexión a la base de datos del cliente 2
db_password = cliente2_password      # Contraseña del usuario de la base de datos del cliente 2
addons_path = /mnt/extra-addons      # Ruta para los addons personalizados
```

---

## Instrucciones para Implementar

### 1. Crear la Estructura de Carpetas
Ejecuta los siguientes comandos para crear las carpetas necesarias:
```bash
mkdir odoo_multiclientes
cd odoo_multiclientes
mkdir -p config_cliente1 addons_cliente1 config_cliente2 addons_cliente2
```

### 2. Crear y Configurar los Archivos
- Guarda el archivo `docker-compose.yaml` en el directorio `odoo_multiclientes/`.
- Crea los archivos `odoo.conf` dentro de `config_cliente1/` y `config_cliente2/` con el contenido proporcionado.

### 3. Iniciar los Servicios
Ejecuta el siguiente comando para levantar los contenedores:
```bash
docker-compose up -d
```

### 4. Verificar los Contenedores
Confirma que todos los contenedores estén en ejecución:
```bash
docker ps
```

### 5. Configurar las Instancias
Accede a las instancias desde el navegador:
- Cliente 1: [http://localhost:8200](http://localhost:8200)
- Cliente 2: [http://localhost:8201](http://localhost:8201)

Crea las bases de datos `cliente1_db` y `cliente2_db` según corresponda.

---

## Resolución de Problemas

1. **Inicializar Base de Datos Manualmente:**
   Si encuentras problemas con la base de datos, puedes inicializarla manualmente con el siguiente comando:
   ```bash
   docker exec -it odoo_cliente1 odoo --init=base --database=cliente1_db
   ```
   Para el cliente 2, usa:
   ```bash
   docker exec -it odoo_cliente2 odoo --init=base --database=cliente2_db
   ```

2. **Verificar Logs de Contenedores:**
   ```bash
   docker logs odoo_cliente1
   docker logs odoo_cliente2
   ```

3. **Reiniciar Servicios:**
   Si algún servicio falla, reinícialo:
   ```bash
   docker-compose restart
   ```

---

¡Listo! Ahora tienes dos instancias independientes de Odoo configuradas con Docker para diferentes clientes.


# Continuación del Tutorial: Configuración Avanzada y Resolución de Problemas

En esta sección, se incluyen pasos adicionales para personalizar la configuración de Odoo, resolver problemas comunes y optimizar el flujo de trabajo para múltiples clientes.

---

## Personalización Adicional

### 1. Cargar Addons Personalizados

1. Coloca los módulos adicionales en las carpetas correspondientes:
   - `addons_cliente1` para Cliente 1.
   - `addons_cliente2` para Cliente 2.

2. Reinicia los servicios para cargar los nuevos módulos:
   ```bash
   docker-compose restart
   ```

3. Entra en la instancia de Odoo correspondiente y activa el modo desarrollador:
   - Ve a **Configuración** → **Usuarios y Compañías** → **Administración** → **Activar Modo Desarrollador**.

4. Actualiza la lista de aplicaciones desde la interfaz de Odoo:
   - Ve a **Aplicaciones** → **Actualizar Lista de Aplicaciones**.

5. Busca e instala los addons personalizados desde la lista de aplicaciones.

---

### 2. Configuración del Correo Electrónico

Si necesitas configurar el correo electrónico para cada cliente, sigue estos pasos:

1. Accede a la interfaz de Odoo:
   - Cliente 1: [http://localhost:8200](http://localhost:8200).
   - Cliente 2: [http://localhost:8201](http://localhost:8201).

2. Ve a **Configuración** → **Técnico** → **Correo** → **Servidores de Correo Saliente**.

3. Agrega la configuración del servidor SMTP:
   - Servidor SMTP: por ejemplo, `smtp.gmail.com`.
   - Puerto: `587` para conexión segura (TLS).
   - Usuario: correo electrónico del cliente.
   - Contraseña: contraseña del correo electrónico.

4. Prueba la configuración enviando un correo de prueba.

---

## Optimización del Flujo de Trabajo

### 1. Automatizar la Creación de Bases de Datos

Para evitar la configuración manual de bases de datos en cada instancia, puedes usar un script de inicialización automatizado:

1. Crea un archivo llamado `init_db.sh` en el directorio principal:
   ```bash
   touch init_db.sh
   ```

2. Agrega el siguiente contenido al archivo:
   ```bash
   #!/bin/bash
   echo "Inicializando bases de datos..."

   docker exec -it odoo_cliente1 odoo --init=base --database=cliente1_db
   docker exec -it odoo_cliente2 odoo --init=base --database=cliente2_db

   echo "Bases de datos inicializadas correctamente."
   ```

3. Haz el script ejecutable:
   ```bash
   chmod +x init_db.sh
   ```

4. Ejecuta el script:
   ```bash
   ./init_db.sh
   ```

---

## Solución de Problemas Avanzada

### 1. Diagnóstico de Problemas en PostgreSQL

Si las bases de datos no se conectan correctamente:

1. Ingresa al contenedor de PostgreSQL para inspeccionar las bases de datos:
   ```bash
   docker exec -it db_cliente1 bash
   ```

2. Usa el cliente `psql` para verificar las bases de datos:
   ```bash
   psql -U cliente1_user cliente1_db
   ```

3. Ejecuta consultas para verificar las tablas existentes:
   ```sql
   \dt
   ```

4. Sal del cliente `psql`:
   ```bash
   \q
   ```

---

### 2. Debugging en Odoo

Para habilitar logs detallados en Odoo, edita el archivo de configuración de cada cliente (`odoo.conf`):

1. Agrega las siguientes líneas:
   ```ini
   log_level = debug
   logfile = /var/log/odoo/odoo.log
   ```

2. Reinicia el contenedor:
   ```bash
   docker-compose restart odoo_cliente1
   ```

3. Consulta los logs:
   ```bash
   docker logs odoo_cliente1
   ```

---

¡Con estos pasos adicionales, podrás gestionar y personalizar fácilmente múltiples instancias de Odoo! Si necesitas más ayuda o tienes dudas, no dudes en pedirlo. 😊
