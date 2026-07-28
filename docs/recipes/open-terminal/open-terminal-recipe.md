# Open Terminal — Recipe v1.0

## 🎯 Propósito

Terminal web con acceso directo al host de la Raspberry Pi. Puerta de administración remota del ecosistema Bildung. Promovida a Core por valor estratégico: sin ella, la intervención manual requiere SSH físico.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `ghcr.io/open-webui/open-terminal` |
| Puerto interno | Gestionado por Open WebUI (sin puerto expuesto directamente) |
| Puerto host | — (se accede vía Open WebUI) |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| open-webui | Interfaz contenedora | Sí — open-terminal se integra como plugin de Open WebUI |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Default | Descripción |
|---|---|---|---|
| `OPEN_TERMINAL_API_KEY` | **Sí** | — | API key que autentica la terminal contra Open WebUI |
| `OPEN_TERMINAL_PACKAGES` | No | — | Paquetes APT a instalar en el contenedor (separados por espacio) |
| `OPEN_TERMINAL_PIP_PACKAGES` | No | — | Paquetes PIP a instalar en el contenedor (separados por espacio) |
| `JINA_API_KEY` | No | — | API key compartida con Open WebUI |
| `TZ` | No | `UTC` | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/home/user` | `./data/open-terminal/` | Home del usuario dentro de la terminal. Scripts, configuraciones, historial de shell. |

---

## 🚀 Deploy

```bash
# Desde la raíz del proyecto docker/
docker compose up -d open-terminal
```

> ⚠️ `open-terminal` depende de `open-webui`. Si `open-webui` no está corriendo, `open-terminal` no se iniciará.  
> Para levantar ambos: `docker compose up -d`

---

## 🩺 Health check

```bash
# Verificar que el contenedor está corriendo
docker compose ps open-terminal
# Esperado: State = Up

# Verificar que la terminal responde (vía API de Open WebUI)
curl -s http://localhost:3000/api/v1/open-terminal/health \
  -H "Authorization: Bearer ${OPEN_TERMINAL_API_KEY}"
```

---

## 🧹 Backup

La terminal en sí no almacena datos críticos — el home del usuario contiene configuraciones y scripts. Si personalizaste herramientas dentro del contenedor, respalda:

```bash
docker compose stop open-terminal
tar -czf open-terminal-backup-$(date +%Y%m%d).tar.gz ./data/open-terminal/
docker compose up -d open-terminal
```

---

## 🔄 Restore

```bash
docker compose stop open-terminal
tar -xzf open-terminal-backup-YYYYMMDD.tar.gz
docker compose up -d open-terminal
```

---

## 📦 Paquetes persistentes

Los paquetes instalados vía `OPEN_TERMINAL_PACKAGES` y `OPEN_TERMINAL_PIP_PACKAGES` se reinstalan en cada arranque del contenedor. Si necesitas herramientas adicionales de forma permanente:

1. Agregarlas al `.env`
2. Recrear el contenedor: `docker compose up -d --force-recreate open-terminal`

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No aparece la terminal en Open WebUI | API key incorrecta | Verificar que `OPEN_TERMINAL_API_KEY` coincide en `.env` y en la config de Open WebUI |
| La terminal carga pero no acepta comandos | Paquete faltante | Revisar `OPEN_TERMINAL_PACKAGES`, recrear contenedor |
| `depends_on` falla | open-webui no está listo aún | Esperar unos segundos tras levantar open-webui. Si persiste: `docker compose restart open-terminal` |
| Error de permisos en `/home/user` | Volumen montado con dueño incorrecto | `chown -R 1000:1000 ./data/open-terminal/` |
| Paquete PIP no se instala | Nombre mal escrito o versión incompatible | Verificar logs: `docker compose logs open-terminal` |
