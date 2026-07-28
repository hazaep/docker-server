# Traefik — Recipe v1.0

## 🎯 Propósito

Reverse proxy. **Todo el tráfico HTTP del ecosistema pasa por aquí.** Sin él, nada es accesible.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `traefik:latest` |
| Puerto host | `80` (HTTP) |
| Dashboard | `8080` (interno) |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| cloudflared | Fuente de tráfico | Sí (desde internet) |
| docker.sock | Descubrimiento de servicios | **Sí** |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `DOMINIO` | **Sí** | Dominio base para reglas Host() |
| `TRAEFIK_AUTH` | No | Credenciales dashboard (si se activa auth) |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/var/run/docker.sock` (ro) | Host | Descubrimiento de servicios |
| `/dynamic` (ro) | `./data/traefik/dynamic/` | Middlewares YAML (cloudflare.yml) |

---

## 🚀 Deploy

```bash
docker compose up -d traefik
```

---

## 🩺 Health check

```bash
# Dashboard
curl -s http://traefik.${DOMINIO}/dashboard/ | head -3

# API (solo red interna)
curl -s http://traefik:8080/api/rawdata | jq '.routers | keys'

# Verificar que un servicio está registrado
curl -s http://traefik:8080/api/http/routers | jq '.[] | select(.service=="n8n-service@docker")'
```

---

## 🧹 Backup

```bash
# Solo el archivo de middlewares
cp ./data/traefik/dynamic/cloudflare.yml ./data/traefik/dynamic/cloudflare.yml.backup
```

---

## 🔄 Restore

```bash
cp cloudflare.yml.backup ./data/traefik/dynamic/cloudflare.yml
docker compose restart traefik
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| 502 Bad Gateway | El servicio backend no está corriendo | `docker compose ps <servicio>` |
| 404 en ruta | Label `traefik.http.routers.*.rule` mal configurada | Revisar compose del servicio |
| Dashboard no carga | `--api.dashboard=true` ausente | Verificar command en compose |
| Servicio no aparece | `traefik.enable=true` ausente en labels | Agregar label al servicio |
| Error de middlewares | `cloudflare.yml` no encontrado | Verificar `--providers.file.directory=/dynamic` y el bind mount |
