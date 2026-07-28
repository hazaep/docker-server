# MinIO — Recipe v1.0

## 🎯 Propósito

Almacenamiento de objetos S3-compatible. **El sistema de archivos del ecosistema.** NocoDB y Resume dependen de él para assets.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `minio/minio:latest` |
| Puerto API S3 | `9000` (interno) |
| Puerto Consola | `39001` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| traefik | Acceso a la consola web | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `MINIO_R_USER` | **Sí** | Usuario root (admin) |
| `MINIO_R_PASS` | **Sí** | Contraseña root |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/data` | `./data/minio/` | Buckets, objetos, metadata |

---

## 🚀 Deploy

```bash
docker compose up -d minio
```

---

## 🩺 Health check

```bash
# API S3
curl -s http://minio:9000/minio/health/live
# Esperado: 200

# Consola web
curl -s -o /dev/null -w "%{http_code}" https://minio.${DOMINIO}
# Esperado: 200
```

---

## 🪣 Crear un bucket

```bash
# Instalar cliente mc (una vez)
docker compose exec minio sh -c "wget -q https://dl.min.io/client/mc/release/linux-arm64/mc -O /usr/bin/mc && chmod +x /usr/bin/mc"

# Configurar alias
docker compose exec minio mc alias set local http://localhost:9000 ${MINIO_R_USER} ${MINIO_R_PASS}

# Crear bucket
docker compose exec minio mc mb local/nuevo-bucket

# Listar buckets
docker compose exec minio mc ls local
```

---

## 🧹 Backup

```bash
# Backup completo de todos los buckets
docker compose stop nocodb resume
tar -czf minio-backup-$(date +%Y%m%d).tar.gz ./data/minio/
docker compose up -d nocodb resume
```

---

## 🔄 Restore

```bash
docker compose stop nocodb resume
tar -xzf minio-backup-YYYYMMDD.tar.gz
docker compose up -d nocodb resume
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| Consola no carga | `MINIO_CONSOLE_ADDRESS` mal configurado | Debe ser `minio:39001` |
| Bucket no accesible | Credenciales incorrectas | Verificar `MINIO_R_USER` y `MINIO_R_PASS` |
| NocoDB no sube archivos | Bucket no existe | Crear bucket manualmente |
| Disco lleno | Archivos acumulados sin limpiar | Revisar `du -sh ./data/minio/` |
