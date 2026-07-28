# oikb — Recipe v1.0

## 🎯 Propósito

Sincronizador incremental de Knowledge Bases. **Mantiene las KBs de Open WebUI actualizadas desde directorios locales u otras fuentes.** Modo daemon con schedule, API REST y webhooks.

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
| `KB_BILDUNG_ID` | Condicional | UUID de la KB de Bildung |
| `KB_OBSIDIAN_ID` | Condicional | UUID de la KB de Obsidian |
| `SYNC_PATH` | No | Path host montado en `/sync` (default: `/home/user/sync`) |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path host | Path contenedor | Propósito |
|---|---|---|
| `./data/oikb/.oikb.yaml` (ro) | `/app/.oikb.yaml` | Configuración de fuentes |
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

# Ver fuentes configuradas
docker compose logs oikb | grep -i "source"
```

---

## ⚙️ Setup inicial

### 1. Crear KB en Open WebUI

Abrir Open WebUI → Knowledge → Create Knowledge Base → copiar el UUID de la URL.

### 2. Configurar `.oikb.yaml`

```yaml
# ./data/oikb/.oikb.yaml
defaults:
  interval: 30m
  concurrency: 2

sources:
  - name: bildung-docs
    source: /sync/bildung/docs
    kb-id: ${KB_BILDUNG_ID}
    filter:
      include: ["**.md"]
```

### 3. Configurar `.env`

```bash
OIKB_OPEN_WEBUI_API_KEY=sk-...
OIKB_API_KEY=<generar>
KB_BILDUNG_ID=<uuid>
SYNC_PATH=/home/user/sync
```

### 4. Asegurar que los directorios existen bajo `SYNC_PATH`

```bash
ls /home/user/sync/bildung/docs/
```

### 5. Levantar

```bash
docker compose up -d oikb
docker compose logs -f oikb
```

---

## 🔄 Forzar sync manual

```bash
curl -X POST https://oikb.${DOMINIO}/sync/bildung-docs \
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
| `source not found` | El directorio no existe en `/sync` | Verificar bind mount con `docker compose exec oikb ls /sync/` |
| No sincroniza | Schedule no se ha cumplido aún | Forzar: `POST /sync/{name}` |
| `KB not found` | KB ID incorrecto | Verificar UUID en Open WebUI |
| API responde 401 | `OIKB_API_KEY` no configurada o incorrecta | Verificar `.env` y header `Authorization: Bearer` |
