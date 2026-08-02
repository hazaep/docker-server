# oikb — Recipe v1.0

## 🎯 Propósito

Sincronizador incremental de Knowledge Bases. **Mantiene las KBs de Open WebUI actualizadas desde directorios locales sincronizados vía Syncthing.** Modo daemon con schedule, API REST y webhooks.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `ghcr.io/open-webui/oikb:latest` |
| Puerto interno | `8080` |
| Comando | `daemon` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| open-webui | API de KBs (`:8080`) | **Sí** |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

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

## 📂 Volúmenes

| Path host | Path contenedor | Propósito |
|---|---|---|
| `./data/oikb/.oikb.yaml` (ro) | `/app/.oikb.yaml` | Fuentes, schedules, filtros |
| `${SYNC_PATH}` (ro) | `/sync` | Directorios a sincronizar |

---

## 🚀 Deploy

```bash
docker compose up -d oikb
```

---

## 🩺 Health check

```bash
curl -s http://oikb:8080/health/ready
# Esperado: {"status": "ok"}
```

---

## ⚙️ Setup inicial

### 1. Crear KBs en Open WebUI

Abrir Open WebUI → Knowledge → Create Knowledge Base. Copiar los UUIDs de las URLs.

### 2. Configurar `.oikb.yaml`

Las fuentes ya están definidas en `./data/oikb/.oikb.yaml`:

- `codex` → Codex de Bildung (7 Escuelas, ontología)
- `n8n-docs` → documentación de n8n
- `docker-server` → documentación del docker-server

### 3. Asegurar que Syncthing sincroniza las carpetas

```bash
ls /home/user/sync/Obsidian/Mi\ Mente/Bildung/Lab/codex/
ls /home/user/sync/repos/n8n-docs/
ls /home/user/sync/Obsidian/Mi\ Mente/Bildung/Lab/docker-server/
```

### 4. Configurar `.env`

```bash
OIKB_OPEN_WEBUI_API_KEY=sk-...
OIKB_API_KEY=<generar>
KB_CODEX_ID=<uuid>
KB_N8N_DOCS_ID=<uuid>
KB_DOCKER_SERVER_ID=<uuid>
SYNC_PATH=/home/user/sync
```

### 5. Levantar

```bash
docker compose up -d oikb
docker compose logs -f oikb
```

---

## 🔄 Forzar sync manual

```bash
curl -X POST https://oikb.${DOMINIO}/sync/codex \
  -H "Authorization: Bearer ${OIKB_API_KEY}"
```

---

## 🧹 Backup

```bash
cp ./data/oikb/.oikb.yaml ./data/oikb/.oikb.yaml.backup
```

Los datos están en Open WebUI (no en oikb).

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| `401 Unauthorized` | API key de Open WebUI incorrecta | Verificar `OIKB_OPEN_WEBUI_API_KEY` en `.env` |
| `source not found` | El directorio no existe en `/sync` | Verificar bind mount y Syncthing |
| No sincroniza | Schedule no se ha cumplido aún | Forzar: `POST /sync/{name}` |
| `KB not found` | KB ID incorrecto | Verificar UUID en Open WebUI |
