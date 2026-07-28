# Redis — Recipe v1.0

## 🎯 Propósito

Caché y colas de trabajos en memoria. **Efímero por diseño.** Si se pierden los datos, el sistema se reconstruye desde Postgres.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `redis:7-alpine` |
| Puerto interno | `6379` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| — | — | Ninguna. Solo requiere la red `services`. |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `REDIS_PASSWORD` | **Sí** | Contraseña para `requirepass` |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| Named volume | `redis_data` | `/data` | dump.rdb si se habilita persistencia |

---

## 🚀 Deploy

```bash
docker compose up -d redis
```

---

## 🩺 Health check

```bash
# Sin auth debe rechazar
docker compose exec redis redis-cli PING
# Esperado: NOAUTH Authentication required

# Con auth debe responder
docker compose exec redis redis-cli -a "${REDIS_PASSWORD}" PING
# Esperado: PONG

# Ver memoria usada
docker compose exec redis redis-cli -a "${REDIS_PASSWORD}" INFO memory | grep used_memory_human
```

---

## 🧹 Backup

```bash
# Redis es efímero. Si necesitas persistencia, habilitar SAVE:
docker compose exec redis redis-cli -a "${REDIS_PASSWORD}" BGSAVE
# El dump.rdb se guarda en ./data/redis/

# Backup manual del dump
cp /var/lib/docker/volumes/redis_data/_data/dump.rdb /var/lib/docker/volumes/redis_data/_data/dump.rdb.backup-$(date +%Y%m%d)
```

---

## 🔄 Restore

```bash
docker compose stop redis
cp dump.rdb.backup-YYYYMMDD /var/lib/docker/volumes/redis_data/_data/dump.rdb
docker compose up -d redis
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | `REDIS_PASSWORD` no definida | Verificar `.env` |
| `NOAUTH` en apps | Contraseña incorrecta | Revisar `REDIS_URL` en las apps |
| Memoria llena | Sin `maxmemory` configurado | Agregar `--maxmemory 256mb` al command |
| Jobs de n8n se pierden | Redis sin persistencia | Habilitar `SAVE` en config o aceptar que es efímero |
