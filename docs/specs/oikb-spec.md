# 📘 oikb — Service Specification v1.0

**Capa:** 🟡 Extensión | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Sincronizador incremental de Knowledge Bases de Open WebUI. **Mantiene las KBs actualizadas desde fuentes locales sincronizadas vía Syncthing.** Si cae, las KBs dejan de actualizarse automáticamente, pero Open WebUI sigue funcionando.

---

## 2. Definición del Servicio

> Daemon que lee `.oikb.yaml`, escanea fuentes, calcula SHA-256, envía un manifiesto a la API de Open WebUI, y sube solo archivos nuevos o modificados. Soporta 46 conectores (GitHub, S3, Confluence, Zotero, etc.) y directorios locales vía bind mount.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Solo lectura** | Nunca escribe en los directorios fuente. Bind mount `:ro` |
| **Incremental** | Solo transfiere archivos con hash diferente |
| **Configuración externa** | Todo se define en `.oikb.yaml`. El compose no cambia al agregar fuentes |
| **Stateless** | Sin base de datos propia. El estado está en Open WebUI |

---

## 3. Contrato de Infraestructura

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **API** | `https://oikb.${DOMINIO}` |
| **Health** | `GET /health/ready` |
| **Metrics** | `GET /metrics` (Prometheus) |
| **Sync trigger** | `POST /sync/{name-or-kb-id}` |
| **Puerto interno** | `8080` |

### 🔹 Dependencias

```text
oikb
  └── open-webui (🟠 Core) — API de KBs
```

---

## 4. Volumen y Configuración

| Tipo | Path host | Path contenedor | Contenido |
|---|---|---|---|
| Bind mount (ro) | `./data/oikb/.oikb.yaml` | `/app/.oikb.yaml` | Fuentes, schedules, filtros |
| Bind mount (ro) | `${SYNC_PATH:-/home/user/sync}` | `/sync` | Directorios a sincronizar |

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `OIKB_OPEN_WEBUI_API_KEY` | **Sí** | API key de Open WebUI |
| `OIKB_API_KEY` | **Sí** | API key para proteger el daemon |
| `KB_CODEX_ID` | Condicional | UUID de la KB del Codex |
| `KB_N8N_DOCS_ID` | Condicional | UUID de la KB de n8n docs |
| `KB_DOCKER_SERVER_ID` | Condicional | UUID de la KB del docker-server |
| `SYNC_PATH` | No | Path host para bind mount `/sync` |
| `TZ` | No | Zona horaria |

---

## 6. Fuentes activas

| Name | Source | KB | Formato |
|---|---|---|---|
| `codex` | `/sync/Obsidian/Mi Mente/Bildung/Lab/codex` | `${KB_CODEX_ID}` | `.md`, `.html` |
| `n8n-docs` | `/sync/repos/n8n-docs` | `${KB_N8N_DOCS_ID}` | `.md`, `.html` |
| `docker-server` | `/sync/Obsidian/Mi Mente/Bildung/Lab/5-infraestructura/docker-server` | `${KB_DOCKER_SERVER_ID}` | `.md`, `.html` |

---

## 7. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps oikb` → Up (healthy) |
| **CA-2** | Health check responde | `curl -s http://oikb:8080/health/ready` → 200 |
| **CA-3** | Lee la config | `docker compose logs oikb` → fuentes listadas |
| **CA-4** | Conecta a Open WebUI | `docker compose logs oikb` → sin errores 401 |
| **CA-5** | Sync se ejecuta | Esperar el intervalo → logs muestran "Synced" o "nothing to do" |
| **CA-6** | API protegida | `curl -s http://oikb:8080/sync/codex` → 401 |

---

📌 **Nota SDD:** oikb está en Extensión porque su caída solo detiene la sincronización automática de KBs. Open WebUI y sus KBs existentes siguen funcionando. Las fuentes se sincronizan primero a la Raspberry Pi vía Syncthing, y oikb las lee desde el bind mount `/sync`.
