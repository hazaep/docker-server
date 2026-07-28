# 📘 n8n — Service Specification v1.0

**Capa:** 🟠 Core | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Motor de automatización del ecosistema Bildung. **Es el sistema operativo que conecta servicios entre sí.** Sin n8n, los workflows que orquestan IA, bases de datos, notificaciones y APIs externas dejan de ejecutarse.

---

## 2. Definición del Servicio

> n8n self-hosted con PostgreSQL como backend de estado y Redis como broker de colas. Expone tres puntos de entrada vía Traefik: editor, webhooks y MCP.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Un workflow, una responsabilidad** | Cada workflow hace una cosa y la hace bien |
| **Estado en Postgres** | Las credenciales y ejecuciones sobreviven al contenedor |
| **Webhooks como interfaz** | Los servicios externos se integran vía webhooks, no conexiones directas |
| **MCP para agentes** | Los agentes IA usan el endpoint MCP para descubrir herramientas |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 512 MB, recomendado 1 GB | Sí |
| **Disco** | Mínimo 2 GB (workflows + credenciales) | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |
| **Docker socket** | `/var/run/docker.sock` (read-only) | Sí — para ejecutar runners |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Editor** | `https://n8n.${DOMINIO}` |
| **Webhooks** | `https://hooks.${DOMINIO}/webhook/*` |
| **MCP** | `https://hooks.${DOMINIO}/mcp/*` |
| **Puerto interno** | `5678` |

### 🔹 Dependencias

```text
n8n
  ├── postgres    (🔴 Fundación) — estado persistente
  └── redis       (🔴 Fundación) — colas Bull
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `n8n_data` | `/home/node/.n8n` | Workflows, credenciales, configuraciones, ejecuciones |
| Bind mount (host) | — | `/var/run/docker.sock` | Acceso a Docker (ro) |

### 🔹 Regla de backup

> El backup de n8n es el backup de Postgres. Las configuraciones están en `n8n_data`, pero los workflows y credenciales se almacenan en la base de datos `n8n` de Postgres. Respaldar ambos.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `N8N_ENCRYPTION_KEY` | **Sí** | Clave para encriptar credenciales |
| `N8N_POSTGRES_DB` | **Sí** | Nombre de la base de datos |
| `N8N_POSTGRES_USER` | **Sí** | Usuario de postgres |
| `N8N_POSTGRES_PASSWORD` | **Sí** | Contraseña de postgres |
| `N8N_POSTGRES_HOST` | **Sí** | Hostname de postgres |
| `N8N_POSTGRES_PORT` | **Sí** | Puerto de postgres |
| `N8N_REDIS_HOST` | **Sí** | Hostname de redis |
| `N8N_REDIS_PORT` | **Sí** | Puerto de redis |
| `N8N_REDIS_PASSWORD` | **Sí** | Contraseña de redis |
| `N8N_BASIC_AUTH_PASSWORD` | Condicional | Hash de contraseña (si auth activo) |
| `N8N_RUNNERS_TOKEN` | **Sí** | Token compartido con n8n-runners para autenticación del broker |
| `N8N_RUNNERS_MODE` | **Sí** | `external` — delega ejecución a runners dedicados |
| `N8N_RUNNERS_TASK_TIMEOUT` | No | Timeout en segundos para tareas en runners (default: 300) |
| `N8N_NATIVE_PYTHON_RUNNER` | No | Habilitar soporte nativo de Python en Code nodes |
| `N8N_RUNNERS_BROKER_LISTEN_ADDRESS` | **Sí** | `0.0.0.0` — expone el broker a la red Docker |
| `N8N_COMPRESSION_NODE_MAX_ZIP_ENTRIES` | No | Límite de entradas en archivos ZIP (default aumentado a 5000) |
| `DOMINIO` | **Sí** | Dominio base para URLs |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps n8n` → Up |
| **CA-2** | Editor accesible | `curl -s https://n8n.${DOMINIO}` → login page |
| **CA-3** | Webhooks responden | `POST https://hooks.${DOMINIO}/webhook/test` → 200 |
| **CA-4** | Conexión a postgres | Workflows y credenciales sobreviven a recrear el contenedor |
| **CA-5** | Colas funcionan | Workflow con múltiples ejecuciones concurrentes no pierde jobs |
| **CA-6** | Runners operativos | Workflow con `Execute Command` node funciona |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Sin workers externos** | Por ahora no hay n8n workers dedicados |
| **Rate limit** | Sin límite configurado (la Raspberry Pi lo impone) |
| **Docker socket** | Acceso read-only |

---

📌 **Nota SDD:** n8n es el servicio más difícil de migrar en el ecosistema. Sus credenciales están encriptadas con `N8N_ENCRYPTION_KEY`. Si esta clave se pierde, todas las credenciales guardadas se vuelven irrecuperables.
