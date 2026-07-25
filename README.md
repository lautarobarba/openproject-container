# OpenProject (Docker Compose)

Levanta una instancia de [OpenProject](https://www.openproject.org/) usando la
imagen **all-in-one** oficial (`openproject/openproject:17`), que incluye la
app, Postgres, Memcached y el resto de dependencias en un único contenedor.

## Requisitos

- Docker
- Docker Compose (plugin `docker compose`)

## Puesta en marcha

1. Copiá el archivo de variables de entorno de ejemplo:

   ```bash
   cp .env.example .env
   ```

2. Generá una `SECRET_KEY_BASE` propia (se usa para firmar sesiones y cookies)
   y pegala en `.env`:

   ```bash
   openssl rand -hex 64
   ```

3. Ajustá el resto de variables de `.env` si hace falta (puerto, host, idioma,
   etc.). Ver detalle en la sección [Variables de entorno](#variables-de-entorno).

4. Levantá el contenedor:

   ```bash
   docker compose up -d
   ```

5. Seguí los logs del primer arranque:

   ```bash
   docker compose logs -f openproject
   ```

   La primera vez que se levanta tarda unos minutos (crea la base de datos,
   corre migraciones, precompila assets, etc.). Vas a ver un mensaje de éxito
   como este:

   > This will take a bit of time the first time you launch it, but after a
   > few minutes you should see a success message indicating the default
   > administration password (login: admin, password: admin).

   ### Credenciales por defecto

   | Usuario | Contraseña |
   | ------- | ---------- |
   | `admin` | `admin`    |

   OpenProject va a pedirte cambiar esta contraseña en el primer login. Hacelo
   apenas entres, sobre todo si el puerto queda expuesto fuera de tu red local.

6. Accedé a la app en `http://localhost:8000` (o el puerto/host que hayas
   configurado en `OPENPROJECT_PORT` / `OPENPROJECT_HOST_NAME`).

## Variables de entorno

Definidas en `.env` (ver `.env.example` como plantilla):

| Variable                | Descripción                                                                   |
| ----------------------- | ----------------------------------------------------------------------------- |
| `OPENPROJECT_PORT`      | Puerto local mapeado al puerto 80 del contenedor (la web de OpenProject).     |
| `OPENPROJECT_HTTPS`     | `true`/`false`. Si OpenProject corre detrás de HTTPS (proxy/terminación SSL). |
| `OPENPROJECT_HOST_NAME` | Host (y puerto si aplica) con el que se accede a la instancia.                |
| `SECRET_KEY_BASE`       | Clave para firmar sesiones y cookies. Generarla con `openssl rand -hex 64`.   |

> **Nota sobre `DB_USER` / `DB_PASSWORD` / `DB_NAME`:** la imagen all-in-one
> trae Postgres embebida y su propio entrypoint la inicializa con credenciales
> **fijas** (usuario, contraseña y base = `openproject`). Estas tres variables
> no se pasan a ningún lado todavía y no cambian esas credenciales; quedan
> definidas como referencia para el día que se quiera separar Postgres en su
> propio servicio dentro de `compose.yaml` (ahí sí se usarían para configurar
> `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` y armar el
> `DATABASE_URL`). Mientras tanto, para conectarte a la base usá las
> credenciales reales que se detallan abajo.

## Acceder a la base de datos con un cliente externo

Por defecto el puerto de Postgres **no** está expuesto al host. Si algún día
necesitás conectarte con DBeaver (u otro cliente) a la base interna:

1. En `compose.yaml`, dentro del servicio `openproject`, descomentá la línea:

   ```yaml
   ports:
     - ${OPENPROJECT_PORT}:80
     - ${DB_PORT}:5432
   ```

2. Reiniciá el contenedor:

   ```bash
   docker compose up -d
   ```

3. Conectate con estos datos (credenciales fijas de la imagen, no las de
   `.env`):

   | Campo         | Valor                                   |
   | ------------- | --------------------------------------- |
   | Host          | `localhost`                             |
   | Puerto        | valor de `DB_PORT` (por defecto `5432`) |
   | Base de datos | `openproject`                           |
   | Usuario       | `openproject`                           |
   | Contraseña    | `openproject`                           |

   No olvides volver a comentar la línea del puerto (y reiniciar) cuando ya no
   la necesites expuesta.

## Datos persistentes

- `./openproject/pgdata` → datos de la base Postgres interna.
- `./openproject/assets` → adjuntos, avatares y demás archivos subidos a OpenProject.

Ambos están mapeados como volúmenes en `compose.yaml`, así que sobreviven a
`docker compose down` / recreaciones del contenedor. Borrarlos implica perder
esa información.

## Comandos útiles

```bash
# Ver estado
docker compose ps

# Ver logs en vivo
docker compose logs -f openproject

# Parar
docker compose down

# Parar y actualizar a una imagen nueva
docker compose pull
docker compose up -d
```
