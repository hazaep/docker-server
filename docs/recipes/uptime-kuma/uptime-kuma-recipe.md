# Uptime Kuma — Recipe v1.0

## 🎯 Propósito

Monitor de disponibilidad. **El guardián silencioso.** Te avisa cuando algo se cae. No evita que nada se caiga.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `louislam/uptime-kuma:1` |
| Puerto interno | `3001` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| traefik | Reverse proxy | Sí |
| (servicios monitoreados) | Targets de los probes | No (Kuma inicia sin ellos) |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `TZ` | No | Zona horaria |

> Toda la configuración se hace desde la UI web. No requiere variables.

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/app/data` | `./data/uptime-kuma/` | SQLite con monitores, historial, notificaciones |

---

## 🚀 Deploy

```bash
docker compose up -d uptime-kuma
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://status.${DOMINIO}
# Esperado: 200
```

---

## ⚙️ Setup inicial

1. Abrir `https://status.${DOMINIO}`
2. Crear usuario admin
3. Configurar notificaciones (Telegram, Discord, etc.)
4. Agregar monitores (ver lista recomendada abajo)

---

## 📡 Monitores recomendados

| Servicio | Tipo | Target |
|---|---|---|
| traefik | HTTP | `https://traefik.${DOMINIO}` |
| n8n | HTTP | `https://n8n.${DOMINIO}` |
| open-webui | HTTP | `https://webui.${DOMINIO}` |
| nocodb | HTTP | `https://nocodb.${DOMINIO}` |
| vaultwarden | HTTP | `https://vault.${DOMINIO}` |
| flowise | HTTP | `https://flowise.${DOMINIO}` |
| postgres | TCP | `postgres:5432` |
| redis | TCP | `redis:6379` |

---

## 🧹 Backup

```bash
tar -czf uptime-kuma-backup-$(date +%Y%m%d).tar.gz ./data/uptime-kuma/
```

---

## 🔄 Restore

```bash
docker compose stop uptime-kuma
tar -xzf uptime-kuma-backup-YYYYMMDD.tar.gz
docker compose up -d uptime-kuma
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | Permisos del volumen | `chown -R 1000:1000 ./data/uptime-kuma/` |
| Monitores en rojo pero el servicio funciona | Problema de red o DNS | Verificar conectividad desde dentro del contenedor: `docker compose exec uptime-kuma ping <host>` |
| No envía notificaciones | Webhook/Telegram mal configurado | Revisar configuración en Settings → Notifications |
| SQLite corrupto | Apagado brusco | Restaurar desde backup |
