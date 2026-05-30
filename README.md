# Granja Verde BI — Metabase Self-Hosted

Metabase para análisis de datos de Granja Verde, expuesto via Cloudflare Tunnel en `granjaverde-bi.dbarnet.app`.

## Prerequisitos

- Docker y Docker Compose instalados en el servidor
- Nginx corriendo (para proxy en producción)

## Setup local (desarrollo)

```bash
cp .env.example .env
# Editar .env con contraseñas seguras
docker compose up -d
```

Esperar ~2 minutos y abrir `http://localhost:3001`.

El archivo `docker-compose.override.yml` se aplica automáticamente en local y bindea el puerto a `0.0.0.0` para acceder desde el browser sin necesitar Nginx.

## Deploy en servidor

```bash
git clone <repo> granjaverde_metabase
cd granjaverde_metabase
cp .env.example .env
nano .env  # poner contraseñas reales

# Copiar config de Nginx
sudo cp nginx/granjaverde-bi.conf /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/granjaverde-bi.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# Levantar Metabase (sin override en producción)
docker compose -f docker-compose.yml up -d
```

Agregar en Cloudflare Tunnel el hostname `granjaverde-bi.dbarnet.app` apuntando a `localhost:80`.

## Conexión a la DB analítica de Granja Verde

Una vez arriba Metabase, desde el admin panel:

1. Settings → Admin → Databases → Add database
2. Tipo: PostgreSQL
3. Host: IP del servidor de DB
4. User: `gverde_bi` (read-only)
5. Base de datos: la DB de producción de Granja Verde

## Backup de la app DB de Metabase

Agregar al cron del servidor:

```bash
# Backup diario a las 3am
0 3 * * * docker exec metabase-db pg_dump -U metabase metabaseappdb > /backups/metabase_$(date +\%Y\%m\%d).sql
```
