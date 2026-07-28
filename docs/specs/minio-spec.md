# 📘 MinIO — Service Specification v1.0

**Capa:** 🟡 Extensión | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Almacenamiento de objetos compatible con S3. **Es el sistema de archivos del ecosistema.** NocoDB lo usa para guardar assets adjuntos a las tablas. Sin él, las apps que dependen de almacenamiento de archivos se degradan.

---

## 2. Definición del Servicio

> MinIO self-hosted, single-node, single-drive. Sin réplicas. Sin erasure coding. Un bucket por aplicación. Consola web accesible vía Traefik.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Un bucket por app** | Cada servicio que necesita almacenamiento crea su bucket |
| **Consola con auth** | La UI de MinIO requiere usuario y contraseña |
| **Sin alta disponibilidad** | Instancia única. Si se pierde el disco, se pierden los archivos |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB | Sí |
| **Disco** | `./data/minio/` — crece con los archivos | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **API S3** | `minio:9000` (interno) |
| **Consola web** | `https://minio.${DOMINIO}` → `minio:39001` |
| **URL interna** | `http://minio:9000` |

### 🔹 Dependencias

```text
minio
  └── (ninguna — es independiente)
```

### 🔹 ¿Quién depende de MinIO?

```text
minio
  ├── nocodb   (🟠 Core)     — assets de tablas
  └── resume   (🟣 Experimento) — plantillas y PDFs generados
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Contenido |
|---|---|---|
| `/data` | `./data/minio/` | Buckets, objetos, metadata |

### 🔹 Regla

> Si el volumen se pierde, se pierden todos los archivos subidos por las aplicaciones. Los datos de las tablas (nocodb) están en postgres — solo los adjuntos se pierden.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `MINIO_R_USER` | **Sí** | Usuario root (admin) |
| `MINIO_R_PASS` | **Sí** | Contraseña root |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps minio` → Up |
| **CA-2** | Consola accesible | `curl -s https://minio.${DOMINIO}` → login page |
| **CA-3** | API S3 responde | `curl -s http://minio:9000/minio/health/live` → 200 |
| **CA-4** | Datos sobreviven a restart | Subir archivo → `restart` → archivo sigue |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Sin réplica** | Single-node, single-drive |
| **Sin versionado** | No configurado por defecto |
| **Sin cifrado en reposo** | Los archivos están en texto plano en disco |

---

📌 **Nota SDD:** MinIO está en Extensión y no en Core porque su caída solo afecta assets adjuntos, no datos estructurales. Si nocodb depende críticamente de archivos en MinIO, reevaluar promoción a Core.
