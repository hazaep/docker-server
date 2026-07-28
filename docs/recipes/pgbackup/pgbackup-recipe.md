# pgBackup — Recipe v1.0

## 🎯 Propósito

Backups automáticos de PostgreSQL. **El seguro de vida del ecosistema.** Sin esto, un fallo de postgres es catastrófico.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `prodrigestivill/postgres-backup-local` |
| Schedule | `@daily` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Origen de los backups | **Sí** |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `BACKUP_HOST` | **Sí** | Hostname de postgres |
| `BACKUP_DATABASE` | **Sí** | Bases de datos separadas por coma |
| `BACKUP_USER` | **Sí** | Usuario con permisos de lectura |
| `BACKUP_PASSWORD` | **Sí** | Contraseña |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Estructura |
|---|---|---|
| `/backups` | `./data/backups/` | `daily/`, `weekly/`, `monthly/`, `last/` |

---

## 🚀 Deploy

```bash
docker compose up -d pgbackup
```

---

## 🩺 Health check

```bash
# Ver que ejecutó backups
ls -la ./data/backups/last/

# Ver último backup de n8n
ls -lh ./data/backups/last/*n8n*

# Logs de la última ejecución
docker compose logs pgbackup | tail -10
```

---

## 📋 Agregar base de datos al backup

Editar `.env`:

```bash
BACKUP_DATABASE=n8n,arcane,vaultwarden,nocodb,flowise
```

Luego recrear:

```bash
docker compose up -d --force-recreate pgbackup
```

---

## 🧹 Backup manual inmediato

```bash
docker compose exec pgbackup sh -c "pg_dump -U ${BACKUP_USER} -h ${BACKUP_HOST} n8n | gzip > /backups/manual-n8n-$(date +%Y%m%d-%H%M).sql.gz"
```

---

## 🔄 Restore desde backup

```bash
# 1. Detener apps
docker compose stop n8n nocodb

# 2. Restaurar
gunzip < ./data/backups/last/n8n_*.sql.gz | docker compose exec -T postgres psql -U postgres -d n8n

# 3. Levantar
docker compose up -d n8n nocodb
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No genera backups | `BACKUP_DATABASE` vacío o nombres incorrectos | Verificar con `docker compose exec postgres psql -U postgres -l` |
| Backups vacíos (0 bytes) | Base de datos no existe | Crear base de datos faltante |
| Error de contraseña | `BACKUP_PASSWORD` no coincide con postgres | Verificar `.env` |
| Disco lleno | Backups acumulados | Ajustar `BACKUP_KEEP_*` o limpiar manualmente |
