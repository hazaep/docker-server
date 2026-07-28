# Watchtower — Recipe v1.0

## 🎯 Propósito

Actualizador automático de imágenes Docker. **El sistema inmunológico del ecosistema.** Solo toca contenedores con label `com.centurylinklabs.watchtower.enable=true`.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `nickfedor/watchtower:latest` (ARM64 maintained fork) |
| Schedule | `0 0 4 * * *` (4 AM diario) |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| docker.sock (host) | Acceso a la API de Docker | **Sí** |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Default | Descripción |
|---|---|---|---|
| `WATCHTOWER_CLEANUP` | No | `false` | Eliminar imágenes viejas |
| `WATCHTOWER_REMOVE_VOLUMES` | No | `false` | ⚠️ NO activar — borra volúmenes huérfanos |
| `WATCHTOWER_SCHEDULE` | No | — | Cron expression |
| `TZ` | No | `UTC` | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/var/run/docker.sock` | Host | API de Docker |
| `/etc/timezone` (ro) | Host | Timezone del sistema |
| `/etc/localtime` (ro) | Host | Hora local |

---

## 🚀 Deploy

```bash
docker compose up -d watchtower
```

---

## 🩺 Health check

```bash
# Verificar que está monitoreando
docker compose logs watchtower | tail -5
# Esperado: "Checking all containers"

# Ver schedule
docker compose logs watchtower | grep "Waiting"
# Esperado: "Waiting for next run"

# Forzar update de un servicio (manual)
docker compose up -d --force-recreate <servicio>
```

---

## 🧹 Backup

No requiere backup. Sin estado.

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No actualiza nada | Labels ausentes | Verificar que los servicios tienen `com.centurylinklabs.watchtower.enable=true` |
| No corre en schedule | Cron expression mal formada | Verificar `WATCHTOWER_SCHEDULE=0 0 4 * * *` |
| Se queda sin disco | `CLEANUP=false` | Activar `WATCHTOWER_CLEANUP=true` |
| Actualiza y rompe algo | Nueva versión incompatible | `docker compose up -d --force-recreate <servicio>` con tag específico |

### ⚠️ Rollback manual tras mala actualización

```bash
# 1. Ver qué actualizó
docker compose logs watchtower | grep "Updating"

# 2. Forzar versión anterior
# Editar el compose de capa: image: servicio:vX.Y.Z (versión específica)
docker compose up -d --force-recreate <servicio>
```
