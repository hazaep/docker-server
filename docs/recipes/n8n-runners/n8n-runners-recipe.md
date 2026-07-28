# n8n Runners — Recipe v1.0

## 🎯 Propósito

Ejecutor de tareas de Code Node. **Sin esto, los workflows con Python en Code Nodes no se ejecutan.** Imagen personalizada con paquetes Python extra. JS dependencies descartadas por ahora.

---

## 📦 Build

| Campo | Valor |
|---|---|
| Base | `n8nio/runners:latest` |
| Build | `docker compose build n8n-runners` |
| Imagen resultante | `bildung/n8n-runners:custom` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| n8n | Broker de tareas (`:5679`) | **Sí** |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `N8N_RUNNERS_TASK_BROKER_URI` | **Sí** | `http://n8n:5679` |
| `N8N_RUNNERS_AUTH_TOKEN` | **Sí** | Debe coincidir con `N8N_RUNNERS_TOKEN` en n8n |
| `TZ` | No | Zona horaria |

---

## 📂 Archivos de configuración

| Archivo | Path en build context | Path en contenedor |
|---|---|---|
| `Dockerfile` | `./data/n8n-runners/Dockerfile` | — |
| `n8n-task-runners.json` | `./data/n8n-runners/n8n-task-runners.json` | `/etc/n8n-task-runners.json` |

---

## 🚀 Deploy

```bash
# Primera vez: construir la imagen
docker compose build n8n-runners

# Levantar
docker compose up -d n8n-runners
```

---

## 🔄 Reconstruir tras agregar paquetes

```bash
# 1. Editar Dockerfile (agregar RUN con el paquete)
# 2. Editar n8n-task-runners.json (agregar al allowlist)
# 3. Reconstruir sin caché
docker compose build --no-cache n8n-runners
docker compose up -d n8n-runners
```

---

## 🩺 Health check

```bash
# Verificar conexión al broker
docker compose logs n8n-runners | tail -5
# Esperado: sin errores de conexión

# Verificar que n8n lo detecta
# Abrir n8n → Settings → Task Runners → el runner debe aparecer como "connected"
```

---

## 🧹 Backup

No requiere backup. Sin estado. Si se pierde, se reconstruye con `docker compose build n8n-runners`.

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No se conecta al broker | n8n caído o `N8N_RUNNERS_AUTH_TOKEN` no coincide | `docker compose ps n8n` y verificar token en `.env` |
| Error `package not allowed` en Code Node | Paquete no está en `n8n-task-runners.json` | Agregar al allowlist y reconstruir |
| Build falla en ARM64 | Imagen base incompatible | Usar `FROM n8nio/runners:1.121.0` en vez de `latest` |
| `uv pip install` falla | Paquete no disponible para ARM64 | Verificar que el paquete soporta `linux/arm64` |
| Error de permisos en build | `USER root` faltante | El Dockerfile debe tener `USER root` antes de `RUN` |
| `pnpm store mismatch` en build | Versión de pnpm en Corepack cambió | `pnpm config set store-dir` ya lo maneja. Si persiste, usar `npm install` en vez de `pnpm add` |
| `pnpm not in PATH` en build | Corepack descarga pnpm en dir no estándar | Ya manejado con `PATH="$PATH:/root/.local/share/pnpm/bin"` |
