# Postgres — Recipe v1.0

## 🎯 Propósito

Base de datos relacional principal del ecosistema Bildung. **Si se pierde sin backup, se pierde todo.**

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `pgvector/pgvector:pg18` |
| Puerto interno | `5432` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| — | — | Ninguna. Es la raíz de todas las dependencias. |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `POSTGRES_USER` | **Sí** | Usuario administrador |
| `POSTGRES_PASSWORD` | **Sí** | Contraseña del admin |
| `POSTGRES_DB` | **Sí** | Base de datos por defecto |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| Named volume | `postgres_data` | `/var/lib/postgresql` | Datos, índices, WAL |

---

## 🚀 Deploy

```bash
docker compose up -d postgres
```

---

## 🩺 Health check

```bash
docker compose exec postgres pg_isready
# Esperado: /var/run/postgresql:5432 - accepting connections

# Verificar pgvector
docker compose exec postgres psql -U postgres -c "SELECT * FROM pg_available_extensions WHERE name='vector'"
# Esperado: una fila con name='vector'
```

---

## 🧹 Backup

```bash
# Automático: pgbackup corre @daily
ls services/tool/data/backups/last/

# Manual de una base específica:
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres n8n | gzip > /backups/manual-n8n.sql.gz"

# Manual de todas:
docker compose exec postgres pg_dumpall -U postgres | gzip > postgres-full-$(date +%Y%m%d).sql.gz
```

---

## 🔄 Restore

```bash
# 1. Detener apps que escriben en postgres
docker compose stop n8n nocodb flowise arcane vaultwarden

# 2. Restaurar una base
gunzip < manual-n8n.sql.gz | docker compose exec -T postgres psql -U postgres -d n8n

# 3. Restaurar todas
gunzip < postgres-full.sql.gz | docker compose exec -T postgres psql -U postgres

# 4. Levantar
docker compose up -d n8n nocodb flowise arcane vaultwarden
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | Puerto ocupado o datos corruptos | `docker compose logs postgres` |
| `too many clients` | Demasiadas apps conectadas | `docker compose exec postgres psql -U postgres -c "SHOW max_connections"` |
| Lentitud | Disco lleno o índices faltantes | `df -h /var/lib/docker/volumes/postgres_data/` y revisar `pg_stat_user_tables` |
| No acepta conexiones | `pg_hba.conf` restrictivo | Revisar que las apps están en la red `services` |
| pgvector no instalado | Imagen incorrecta | Debe usar `pgvector/pgvector:pg18`, no `postgres:15` |
