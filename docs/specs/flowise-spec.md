# 📘 Flowise — Service Specification v1.0

**Capa:** 🟡 Extensión | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Constructor visual de agentes de IA mediante interfaz drag-and-drop. **Potencia la automatización con flujos de IA sin escribir código.** n8n puede seguir operando sin él, pero los workflows que requieren agentes conversacionales o cadenas LLM complejas dependen de Flowise.

---

## 2. Definición del Servicio

> Flowise en modo `queue` con PostgreSQL como backend de estado, Redis como broker de trabajos, y worker dedicado para ejecución. UI accesible vía Traefik.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Modo queue** | Las ejecuciones se encolan en Redis y las procesa el worker |
| **Estado en Postgres** | Flujos, credenciales y configuraciones persisten en base de datos |
| **Worker separado** | La UI y la ejecución están desacopladas |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 512 MB | Sí |
| **Disco** | `./data/flowise/` | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://flowise.${DOMINIO}` |
| **API interna** | `flowise-main:3000` |
| **Worker** | `flowise-worker:5566` |

### 🔹 Dependencias

```text
flowise
  ├── postgres    (🔴 Fundación) — estado persistente
  └── redis       (🔴 Fundación) — colas queue mode

flowise-worker
  ├── postgres    (🔴 Fundación)
  ├── redis       (🔴 Fundación)
  └── flowise     (🟡 Extensión) — worker sin UI no sirve
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Contenido |
|---|---|---|
| `/root/.flowise` | `./data/flowise/` | Logs, API keys, storage de archivos, secretos encriptados |

> ⚠️ El volumen original usaba `~/.flowise` (home del usuario host). Se migró a `./data/flowise/` para portabilidad.

---

## 5. Variables de Entorno (principales)

| Variable | Obligatorio | Descripción |
|---|---|---|
| `FLOWISE_USERNAME` | **Sí** | Usuario de login |
| `FLOWISE_PASSWORD` | **Sí** | Contraseña de login |
| `FLOWISE_MODE` | **Sí** | `queue` para worker dedicado |
| `FLOWISE_DATABASE_TYPE` | **Sí** | `postgres` |
| `FLOWISE_DATABASE_HOST` | **Sí** | Hostname de postgres |
| `FLOWISE_DATABASE_PORT` | **Sí** | Puerto de postgres |
| `FLOWISE_DATABASE_NAME` | **Sí** | Base de datos |
| `FLOWISE_DATABASE_USER` | **Sí** | Usuario |
| `FLOWISE_DATABASE_PASSWORD` | **Sí** | Contraseña |
| `FLOWISE_REDIS_URL` | **Sí** | URL de conexión a Redis |
| `FLOWISE_JWT_AUTH_TOKEN_SECRET` | **Sí** | Secreto JWT |
| `FLOWISE_JWT_REFRESH_TOKEN_SECRET` | **Sí** | Secreto refresh JWT |
| `FLOWISE_PORT` | No | Puerto de la UI (default: 3000) |
| `FLOWISE_WORKER_PORT` | No | Puerto del worker (default: 5566) |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | UI inicia | `docker compose ps flowise` → Up |
| **CA-2** | Worker inicia | `docker compose ps flowise-worker` → Up |
| **CA-3** | UI accesible | `curl -s https://flowise.${DOMINIO}` → login page |
| **CA-4** | Conexión a postgres | Crear un flow → `restart flowise` → el flow sigue |
| **CA-5** | Worker procesa | Ejecutar un flow → worker muestra logs de ejecución |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Worker único** | Una sola instancia de worker. Si se satura, hay que escalar horizontalmente |
| **UI + worker = mismo código base** | Ambos comparten la misma imagen pero con roles distintos |

---

📌 **Nota SDD:** Flowise está en Extensión porque n8n puede ejecutar llamadas LLM directamente. Flowise añade una capa visual que acelera el desarrollo de agentes, pero si cae, los workflows críticos en n8n siguen funcionando.
