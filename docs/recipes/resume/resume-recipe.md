# Reactive Resume — Recipe v1.0

## 🎯 Propósito

Constructor de CV. **Depende de medio ecosistema, pero nadie depende de él.** Si no lo usas en 6 meses, archivar.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `amruthpillai/reactive-resume:latest` |
| Puerto interno | `3000` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Datos de usuario y CVs | **Sí** |
| redis | Sesiones | **Sí** |
| minio | Almacenamiento de fotos y assets | **Sí** |
| chrome | Exportación de PDFs | **Sí** |
| traefik | Reverse proxy | Sí |

> ⚠️ Es el servicio con más dependencias de todo el ecosistema.

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `RR_DB_URL` | **Sí** | String de conexión PostgreSQL |
| `RR_REDIS_URL` | **Sí** | String de conexión Redis |
| `RR_STORAGE_ENDPOINT` | **Sí** | Host de MinIO |
| `RR_STORAGE_BUCKET` | **Sí** | Bucket en MinIO |
| `RR_STORAGE_ACCESS_KEY` | **Sí** | Usuario MinIO |
| `RR_STORAGE_SECRET_KEY` | **Sí** | Contraseña MinIO |
| `RR_CHROME_URL` | **Sí** | WebSocket de Chrome |
| `RR_CHROME_TOKEN` | **Sí** | Token de Chrome |
| `RR_ACCESS_TOKEN` | **Sí** | Secreto JWT access |
| `RR_REFRESH_TOKEN` | **Sí** | Secreto JWT refresh |
| `RR_PUBLIC_URL` | **Sí** | URL pública |
| `RR_GOOGLE_CLIENT_ID` | No | OAuth Google |
| `RR_GOOGLE_CLIENT_SECRET` | No | Secreto OAuth Google |
| `RR_GITHUB_CLIENT_ID` | No | OAuth GitHub |
| `RR_GITHUB_CLIENT_SECRET` | No | Secreto OAuth GitHub |
| `RR_GITHUB_CALLBACK_URL` | Condicional | Callback OAuth GitHub |
| `RR_APP_URL` | **Sí** | URL base de la app (sin esto no inicia) |
| `RR_AUTH_SECRET` | **Sí** | Secreto de autenticación (sin esto no inicia) |
| `RR_ENCRYPTION_SECRET` | **Sí** | Clave de encriptación para API keys de AI providers |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

Sin volúmenes. Los datos están en Postgres (CVs, usuarios) y MinIO (fotos, assets).

---

## 🚀 Deploy

```bash
docker compose up -d resume
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://resume.${DOMINIO}
# Esperado: 200
```

---

## 🧹 Backup

```bash
# Base de datos (Postgres)
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres resume | gzip > /backups/resume-$(date +%Y%m%d).sql.gz"

# Assets en MinIO (fotos, imágenes)
# Respaldar con el backup completo de MinIO (ver minio-recipe.md)
```

---

## 🔄 Restore

```bash
docker compose stop resume
gunzip < resume-YYYYMMDD.sql.gz | docker compose exec -T postgres psql -U postgres -d resume
docker compose up -d resume
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | Una de las 4 dependencias caída | Verificar: `docker compose ps postgres redis minio chrome` |
| Login no funciona | Google OAuth mal configurado | Verificar `RR_GOOGLE_CLIENT_ID` y callback URL |
| No exporta PDF | Chrome caído o inaccesible | `curl http://chrome:3000` desde la red Docker |
| No sube fotos | MinIO no creó el bucket | Crear bucket `reactive-resume` manualmente (ver minio-recipe) |
| Error 500 | `RR_DB_URL` mal formada | Verificar formato: `postgresql://user:pass@postgres:5432/resume` |
