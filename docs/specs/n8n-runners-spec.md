# 📘 n8n Runners — Service Specification v1.0

**Capa:** 🟠 Core | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Ejecutor de tareas del Code Node de n8n. **Sin runners, los workflows que usan JavaScript o Python en Code Nodes no se ejecutan.** Es el motor de ejecución aislada que permite correr código arbitrario sin comprometer el proceso principal de n8n.

---

## 2. Definición del Servicio

> Imagen personalizada basada en `n8nio/runners:latest` extendida con paquetes extra: `moment`, `uuid` (JavaScript) y `numpy`, `pandas`, `sklearn` (Python). La configuración de allowlists está en `n8n-task-runners.json`.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Imagen extendida** | No usa la imagen vanilla. Se construye con `build` desde un Dockerfile local |
| **Dependencias declaradas** | Los paquetes permitidos están explícitamente en `n8n-task-runners.json` |
| **Aislado de n8n** | n8n envía tareas al broker (`n8n:5679`). El runner solo ejecuta, no tiene acceso al editor ni a la base de datos |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB (depende de las tareas) | Sí |
| **Red** | Acceso a la red `services` (bridge) | **Sí** |
| **n8n broker** | `n8n:5679` accesible y con el mismo `N8N_RUNNERS_AUTH_TOKEN` | **Sí** |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Conexión** | Se conecta al broker en `http://n8n:5679` |
| **Sin puertos expuestos** | El runner no expone endpoints HTTP |

### 🔹 Dependencias

```text
n8n-runners
  └── n8n (🟠 Core) — broker de tareas en :5679
```

---

## 4. Build

La imagen se construye desde un Dockerfile local:

```dockerfile
FROM n8nio/runners:latest
USER root
# JS dependencies descartadas por ahora — pnpm/Corepack incompatible con la imagen base.
# RUN cd /opt/runners/task-runner-javascript && npm install moment uuid
RUN cd /opt/runners/task-runner-python && uv pip install numpy pandas PyUp-LifeUp-API
COPY n8n-task-runners.json /etc/n8n-task-runners.json
USER runner
```

> JS dependencies están comentadas. `pnpm`/`Corepack` en la imagen base de n8n tiene problemas de store/PATH en ARM64. Si se necesitan paquetes JS, usar `npm install` directamente (no `pnpm`).

El contexto de build es `./data/n8n-runners/`, que contiene el Dockerfile y `n8n-task-runners.json`.

```bash
docker compose build n8n-runners
docker compose up -d n8n-runners
```

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `N8N_RUNNERS_TASK_BROKER_URI` | **Sí** | URI del broker de n8n (`http://n8n:5679`) |
| `N8N_RUNNERS_AUTH_TOKEN` | **Sí** | Token compartido con n8n |
| `TZ` | No | Zona horaria |

---

## 6. Paquetes disponibles

### JavaScript

| Paquete | Categoría | Allowlist |
|---|---|---|
| — | — | Sin paquetes JS externos por ahora |

### Python

| Paquete | Categoría | Allowlist |
|---|---|---|
| `json` | Stdlib | `N8N_RUNNERS_STDLIB_ALLOW` |
| `numpy` | Third-party | `N8N_RUNNERS_EXTERNAL_ALLOW` |
| `pandas` | Third-party | `N8N_RUNNERS_EXTERNAL_ALLOW` |
| `PyUp-LifeUp-API` | Third-party | `N8N_RUNNERS_EXTERNAL_ALLOW` |

### 🔹 Agregar un paquete nuevo

1. Agregar al `RUN` correspondiente en el Dockerfile
2. Agregar al allowlist en `n8n-task-runners.json`
3. Reconstruir: `docker compose build --no-cache n8n-runners && docker compose up -d n8n-runners`

---

## 7. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Imagen se construye sin errores | `docker compose build n8n-runners` → exit 0 |
| **CA-2** | Contenedor inicia y se conecta al broker | `docker compose logs n8n-runners` → sin errores de conexión |
| **CA-3** | n8n detecta el runner | En n8n UI → Settings → Task Runners → el runner aparece conectado |
| **CA-4** | Code Node JS funciona | Workflow con `moment()` o `uuid()` en Code Node → ejecuta sin error |
| **CA-5** | Code Node Python funciona | Workflow con `import numpy` en Code Node → ejecuta sin error |
| **CA-6** | Paquete no allowlisteados son rechazados | Workflow con `import requests` → error de seguridad |

---

📌 **Nota SDD:** Los runners son la pieza más nueva del ecosistema. Si el build falla en ARM64, probar con `n8nio/runners:1.121.0` (versión fija) en lugar de `latest`.
