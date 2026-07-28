# 📘 NocoDB — Service Specification v1.0

**Capa:** 🟠 Core | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Base de datos visual tipo Airtable auto-hospedada. **Es la capa de datos accesible para usuarios no técnicos del ecosistema.** Algunos workflows de n8n dependen de datos almacenados en NocoDB.

---

## 2. Definición del Servicio

> NocoDB conectado a Postgres como backend de almacenamiento y Redis como caché. Expone una UI web tipo spreadsheet con API REST y webhooks integrados.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Postgres es la fuente de verdad** | NocoDB es una interfaz; los datos viven en Postgres |
| **API first** | Toda tabla es automáticamente un endpoint REST |
| **Webhooks para integración** | Los cambios en tablas disparan webhooks a n8n |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB | Sí |
| **Disco** | Named volume `nocodb-data` | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://nocodb.${DOMINIO}` |
| **API REST** | `https://nocodb.${DOMINIO}/api/v1/...` |
| **Puerto interno** | `8080` |

### 🔹 Dependencias

```text
nocodb
  ├── postgres    (🔴 Fundación) — almacenamiento de tablas
  └── redis       (🔴 Fundación) — caché
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `nocodb-data` | `/usr/app/data` | Archivos adjuntos, configuraciones de UI |

> Los datos de las tablas viven en Postgres, no en este volumen. Este volumen guarda assets y configuraciones de la interfaz.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `NOCO_PUBLIC_URL` | **Sí** | URL pública de la instancia |
| `NOCO_ADMIN_EMAIL` | **Sí** | Email del admin inicial |
| `NOCO_ADMIN_PASSWORD` | **Sí** | Contraseña del admin inicial |
| `NOCO_DB_NAME` | **Sí** | Base de datos en Postgres |
| `NOCO_DB_USER` | **Sí** | Usuario de Postgres |
| `NOCO_DB_PW` | **Sí** | Contraseña de Postgres |
| `NOCO_REDIS_URL` | **Sí** | URL de conexión a Redis |
| `NOCO_GOOGLE_CLIENT` | No | OAuth Google |
| `NOCO_GOOGLE_SECRET` | No | Secreto OAuth Google |
| `DOMINIO` | **Sí** | Dominio base |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps nocodb` → Up |
| **CA-2** | UI accesible | `curl -s https://nocodb.${DOMINIO}` → login page |
| **CA-3** | Conexión a postgres | Crear una tabla → `restart` → la tabla sigue |
| **CA-4** | API REST funciona | `GET https://nocodb.${DOMINIO}/api/v1/...` → JSON |
| **CA-5** | Watchtower la monitorea | Label `com.centurylinklabs.watchtower.enable=true` |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Sin autenticación SSO** | Solo email/password o Google OAuth |
| **Rate limit de API** | Sin límite (la Raspberry Pi lo impone) |

---

📌 **Nota SDD:** NocoDB está en Core porque otras apps dependen de sus datos. Si se promoviera a Extensión, habría que verificar que ningún workflow crítico de n8n depende de sus tablas.
