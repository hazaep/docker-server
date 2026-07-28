# n8n — Recipe v1.0

## 🎯 Propósito

Motor de automatización. **El sistema operativo del ecosistema.** Conecta servicios entre sí mediante workflows.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `docker.n8n.io/n8nio/n8n:latest` |
| Puerto interno | `5678` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Base de datos de estado | **Sí** |
| redis | Colas de ejecución | **Sí** |
| traefik | Reverse proxy | Sí |
| docker.sock | Runners (ro) | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `N8N_ENCRYPTION_KEY` | **Sí** | ⚠️ Si se pierde, todas las credenciales son irrecuperables |
| `N8N_POSTGRES_HOST` | **Sí** | Hostname de postgres |
| `N8N_POSTGRES_PORT` | **Sí** | Puerto de postgres |
| `N8N_POSTGRES_DB` | **Sí** | Base de datos |
| `N8N_POSTGRES_USER` | **Sí** | Usuario |
| `N8N_POSTGRES_PASSWORD` | **Sí** | Contraseña |
| `N8N_REDIS_HOST` | **Sí** | Hostname de redis |
| `N8N_REDIS_PORT` | **Sí** | Puerto de redis |
| `N8N_REDIS_PASSWORD` | **Sí** | Contraseña de redis |
| `DOMINIO` | **Sí** | Dominio base para URLs |
| `N8N_RUNNERS_TOKEN` | **Sí** | Token compartido con n8n-runners |
| TZ | No | Zona horaria |

---

## 📂 Volúmenes

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `n8n_data` | `/home/node/.n8n` | Workflows, credenciales cifradas, config |
| Bind mount (host) | — | `/var/run/docker.sock` (ro) | Acceso a Docker |

---

## 🚀 Deploy

```bash
docker compose up -d n8n n8n-runners
```

---

## 🩺 Health check

```bash
# Verificar que responde
curl -s -o /dev/null -w "%{http_code}" https://n8n.${DOMINIO}
# Esperado: 200

# Verificar webhooks
curl -s https://hooks.${DOMINIO}/webhook/test
# Esperado: 404 (no hay webhook, pero el endpoint existe)

# Ver base de datos
docker compose exec n8n n8n db:status
```

---

## 🧹 Backup

```bash
# La base de datos (workflows, credenciales, ejecuciones) está en Postgres.
# El named volume tiene configuraciones complementarias.

# Backup de la base (vía pgbackup)
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres n8n | gzip > /backups/n8n-$(date +%Y%m%d).sql.gz"

# Backup del named volume
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine tar czf /backup/n8n_data-$(date +%Y%m%d).tar.gz -C /data .
```

---

## 🔄 Restore

```bash
# 1. Restaurar base de datos
gunzip < n8n-YYYYMMDD.sql.gz | docker compose exec -T postgres psql -U postgres -d n8n

# 2. Restaurar volumen
docker compose stop n8n
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine tar xzf /backup/n8n_data-YYYYMMDD.tar.gz -C /data
docker compose up -d n8n n8n-runners
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | Postgres no accesible | `docker compose ps postgres` |
| Workflows no se ejecutan | Redis caído | `docker compose ps redis` |
| "Cannot decrypt credentials" | `N8N_ENCRYPTION_KEY` cambió | ⚠️ Restaurar `.env` desde backup |
| Webhooks no llegan | URL mal configurada | Verificar `WEBHOOK_URL` y `WEBHOOK_TUNNEL_URL` |
| Runners no aparecen conectados | Token no coincide o n8n-runners caído | `ls -la /var/run/docker.sock` |
