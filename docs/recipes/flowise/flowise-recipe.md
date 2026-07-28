# Flowise — Recipe v1.0

## 🎯 Propósito

Constructor visual de agentes de IA. **n8n puede hacer lo mismo con más código.** Flowise lo acelera con drag-and-drop.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `flowiseai/flowise:latest` |
| Puerto UI | `3000` |
| Reinicio | `always` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Backend de estado | **Sí** |
| redis | Colas (modo queue) | **Sí** |
| flowise-worker | Ejecución de flujos | **Sí** (en modo queue) |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `FLOWISE_USERNAME` | **Sí** | Usuario de login |
| `FLOWISE_PASSWORD` | **Sí** | Contraseña |
| `FLOWISE_MODE` | **Sí** | `queue` para worker dedicado |
| `FLOWISE_DATABASE_TYPE` | **Sí** | `postgres` |
| `FLOWISE_DATABASE_HOST` | **Sí** | Hostname de postgres |
| `FLOWISE_DATABASE_PORT` | **Sí** | Puerto |
| `FLOWISE_DATABASE_NAME` | **Sí** | Base de datos |
| `FLOWISE_DATABASE_USER` | **Sí** | Usuario |
| `FLOWISE_DATABASE_PASSWORD` | **Sí** | Contraseña |
| `FLOWISE_REDIS_URL` | **Sí** | URL de conexión a Redis |
| `FLOWISE_JWT_AUTH_TOKEN_SECRET` | **Sí** | Secreto JWT |
| `FLOWISE_JWT_REFRESH_TOKEN_SECRET` | **Sí** | Secreto refresh JWT |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/root/.flowise` | `./data/flowise/` | Logs, API keys, storage, secretos |

---

## 🚀 Deploy

```bash
docker compose up -d flowise flowise-worker
```

---

## 🩺 Health check

```bash
# UI
curl -s -o /dev/null -w "%{http_code}" https://flowise.${DOMINIO}
# Esperado: 200

# Worker
docker compose logs flowise-worker | tail -3
# Esperado: "Flowise Worker is running"

# Ver flujos creados (API)
curl -s https://flowise.${DOMINIO}/api/v1/chatflows \
  -H "Authorization: Bearer $(echo -n ${FLOWISE_USERNAME}:${FLOWISE_PASSWORD} | base64)"
```

---

## 🧹 Backup

```bash
# Base de datos (Postgres)
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres flowise | gzip > /backups/flowise-$(date +%Y%m%d).sql.gz"

# Volumen (logs, storage)
tar -czf flowise-data-$(date +%Y%m%d).tar.gz ./data/flowise/
```

---

## 🔄 Restore

```bash
docker compose stop flowise flowise-worker
gunzip < flowise-YYYYMMDD.sql.gz | docker compose exec -T postgres psql -U postgres -d flowise
tar -xzf flowise-data-YYYYMMDD.tar.gz
docker compose up -d flowise flowise-worker
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| UI carga pero flujos no ejecutan | Worker caído | `docker compose ps flowise-worker` |
| Error de conexión a postgres | Base de datos `flowise` no existe | Crearla: `docker compose exec postgres psql -U postgres -c "CREATE DATABASE flowise"` |
| Login no funciona | JWT secrets inconsistentes | No cambiar `FLOWISE_JWT_*` después del primer deploy |
| Flujos desaparecen tras restart | Postgres inaccesible al iniciar | Verificar `depends_on` y orden de arranque |
