# 📘 Reactive Resume — Service Specification v1.0

**Capa:** 🟣 Experimento | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Constructor de currículums vitae. **Es una herramienta personal, no infraestructura.** Si cae, Bildung sigue. Si no lo usas en 6 meses, se archiva.

---

## 2. Definición del Servicio

> Reactive Resume v4 con PostgreSQL para datos, Redis para sesiones, MinIO para almacenamiento de assets, y Chrome para exportación de PDFs. Expuesto vía Traefik con OAuth de Google.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Depende de medio ecosistema** | Sin postgres, redis, minio y chrome, no funciona |
| **Pero nada depende de él** | Es un sumidero de dependencias, no una fuente |
| **Google OAuth** | Login con cuenta de Google, no email/password |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://resume.${DOMINIO}` |
| **Puerto interno** | `3000` |

### 🔹 Dependencias

```text
resume
  ├── postgres (🔴 Fundación) — datos de usuario y CVs
  ├── redis    (🔴 Fundación) — sesiones
  ├── minio    (🟡 Extensión) — almacenamiento de fotos y assets
  └── chrome   (🟡 Extensión) — exportación de PDFs
```

---

## 4. Sin volumen persistente

Los datos se almacenan en:
- **Postgres**: estructura del CV, datos de usuario
- **MinIO**: fotos, imágenes, archivos subidos

El contenedor en sí es efímero. Si se recrea, no se pierde nada.

---

## 5. Variables de Entorno (principales)

| Variable | Obligatorio | Descripción |
|---|---|---|
| `RR_PORT` | **Sí** | Puerto interno (3000) |
| `RR_PUBLIC_URL` | **Sí** | URL pública |
| `RR_DB_URL` | **Sí** | String de conexión PostgreSQL |
| `RR_REDIS_URL` | **Sí** | String de conexión Redis |
| `RR_STORAGE_ENDPOINT` | **Sí** | Host de MinIO |
| `RR_STORAGE_BUCKET` | **Sí** | Bucket en MinIO |
| `RR_STORAGE_ACCESS_KEY` | **Sí** | Usuario MinIO |
| `RR_STORAGE_SECRET_KEY` | **Sí** | Contraseña MinIO |
| `RR_CHROME_TOKEN` | **Sí** | Token de autenticación para Chrome |
| `RR_CHROME_URL` | **Sí** | WebSocket de Chrome |
| `RR_ACCESS_TOKEN` | **Sí** | Secreto JWT access |
| `RR_REFRESH_TOKEN` | **Sí** | Secreto JWT refresh |
| `RR_GOOGLE_CLIENT_ID` | No | OAuth Google |
| `RR_GOOGLE_CLIENT_SECRET` | No | Secreto OAuth Google |
| `RR_GITHUB_CLIENT_ID` | No | OAuth GitHub |
| `RR_GITHUB_CLIENT_SECRET` | No | Secreto OAuth GitHub |
| `RR_GITHUB_CALLBACK_URL` | Condicional | Callback OAuth GitHub |
| `RR_APP_URL` | **Sí** | URL base de la app (sin esto no inicia) |
| `RR_AUTH_SECRET` | **Sí** | Secreto de autenticación (sin esto no inicia) |
| `RR_ENCRYPTION_SECRET` | **Sí** | Clave de encriptación para API keys de AI providers |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps resume` → Up |
| **CA-2** | UI accesible | `curl -s https://resume.${DOMINIO}` → HTML |
| **CA-3** | Login con Google funciona | Navegar → login with Google → dashboard |
| **CA-4** | Exporta PDF | Crear CV → Export → se descarga PDF |
| **CA-5** | CV sobrevive a restart | Crear CV → `restart resume` → el CV sigue |

---

## 7. Riesgos

| Riesgo | Impacto | Nota |
|---|---|---|
| Depende de 4 servicios | Si chrome o minio caen, no exporta PDFs | Los datos del CV sobreviven en postgres |
| Sin SMTP | No se pueden enviar CVs por email desde la app | Descargar PDF y enviar manualmente |
| Google OAuth como único login | Si Google OAuth falla, no se puede acceder | Tener exportados los CVs en PDF como backup |

---

📌 **Nota SDD:** Reactive Resume es el servicio con más dependencias de todo el ecosistema (4 servicios de 3 capas distintas). Si alguna vez hay que debuggear una cadena de fallos, probablemente empieza aquí.
