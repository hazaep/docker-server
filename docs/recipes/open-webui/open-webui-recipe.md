# Open WebUI — Recipe v1.0

## 🎯 Propósito

Interfaz web unificada para interactuar con modelos de lenguaje (LLMs). Es la cara visible de las capacidades cognitivas de Bildung.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `ghcr.io/open-webui/open-webui:latest` |
| Puerto interno | `8080` |
| Puerto host | `3000` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| traefik | Reverse proxy | Sí |
| ollama (externo) | Backend de modelos | No — se conecta a instancia remota |
| postgres | Base de datos | No — usa SQLite interno por defecto |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Default | Descripción |
|---|---|---|---|
| `TZ` | No | `UTC` | Zona horaria |
| `JINA_API_KEY` | No | — | API key para embeddings Jina AI (búsqueda semántica) |

### Variables comunes (no definidas aún, útiles para escalar)

| Variable | Cuándo usarla |
|---|---|
| `OLLAMA_BASE_URL` | Si Ollama corre en otro host |
| `DATABASE_URL` | Para migrar de SQLite a Postgres |
| `WEBUI_SECRET_KEY` | Entorno productivo con múltiples instancias |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/app/backend/data` | `./data/open-webui/` | Chats, usuarios, configuraciones, API keys |

---

## 🌐 Traefik

| Label | Valor |
|---|---|
| Router | `webui.${DOMINIO}` |
| Entrypoint | `web` |
| Puerto backend | `8080` |

---

## 🚀 Deploy

```bash
# Desde la raíz del proyecto docker/
docker compose up -d open-webui
```

---

## 🩺 Health check

```bash
# Verificar que responde
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
# Esperado: 200

# Verificar que sirve HTML (no error)
curl -s http://localhost:3000 | head -3
# Esperado: <!DOCTYPE html>...
```

---

## 🧹 Backup

```bash
# Detener para evitar corrupción de SQLite
docker compose stop open-webui

# Respaldar
tar -czf open-webui-backup-$(date +%Y%m%d).tar.gz ./data/open-webui/

# Levantar
docker compose up -d open-webui
```

---

## 🔄 Restore

```bash
docker compose stop open-webui
tar -xzf open-webui-backup-YYYYMMDD.tar.gz
docker compose up -d open-webui
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No carga la UI (502 Bad Gateway) | open-webui no está corriendo | `docker compose ps open-webui` |
| Carga pero no responde el chat | Ollama no accesible | Verificar `OLLAMA_BASE_URL` o conectividad con el host de Ollama |
| Error 500 al hacer login | SQLite corrupto | Restaurar backup del volumen |
| No se ven modelos | Ollama sin modelos descargados | `ollama pull <modelo>` en el host de Ollama |
| Traefik no enruta | Label mal configurada | Revisar `docker compose logs traefik` |
