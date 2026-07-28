# NocoDB — Recipe v1.0

## 🎯 Propósito

Base de datos visual tipo Airtable. **Capa de datos accesible sin SQL.** Algunos workflows de n8n dependen de sus tablas.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `nocodb/nocodb:latest` |
| Puerto interno | `8080` |
| Reinicio | `always` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Backend de almacenamiento | **Sí** |
| redis | Caché | **Sí** |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `NOCO_PUBLIC_URL` | **Sí** | URL pública |
| `NOCO_ADMIN_EMAIL` | **Sí** | Email del admin inicial |
| `NOCO_ADMIN_PASSWORD` | **Sí** | Contraseña del admin |
| `NOCO_DB_NAME` | **Sí** | Base de datos en postgres |
| `NOCO_DB_USER` | **Sí** | Usuario postgres |
| `NOCO_DB_PW` | **Sí** | Contraseña postgres |
| `NOCO_REDIS_URL` | **Sí** | URL de conexión a Redis |
| `NOCO_GOOGLE_CLIENT` | No | OAuth Google |
| `NOCO_GOOGLE_SECRET` | No | Secreto OAuth Google |
| `DOMINIO` | **Sí** | Dominio base |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `nocodb-data` | `/usr/app/data` | Archivos adjuntos, config UI |

> Los datos de las tablas viven en Postgres, no en este volumen.

---

## 🚀 Deploy

```bash
docker compose up -d nocodb
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://nocodb.${DOMINIO}
# Esperado: 200

# Ver que las tablas persisten
curl -s https://nocodb.${DOMINIO}/api/v1/db/data/noco_db/ -H "xc-auth: ${NOCO_API_TOKEN}"
```

---

## 🧹 Backup

```bash
# Los datos están en postgres. Respaldar la base noco_db:
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres noco_db | gzip > /backups/nocodb-$(date +%Y%m%d).sql.gz"
```

---

## 🔄 Restore

```bash
docker compose stop nocodb
gunzip < nocodb-YYYYMMDD.sql.gz | docker compose exec -T postgres psql -U postgres -d noco_db
docker compose up -d nocodb
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| 500 Internal Error | Postgres no accesible | `docker compose ps postgres` |
| Tablas vacías tras restart | Conexión a postgres mal configurada | Revisar `NC_DB` en compose |
| Login no funciona | Credenciales de admin cambiadas | Resetear desde `NC_ADMIN_*` en `.env` |
| Caché no funciona | Redis caído o URL incorrecta | `docker compose exec redis redis-cli -a "${REDIS_PASSWORD}" PING` |
