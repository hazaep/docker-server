# Arcane — Recipe v1.0

## 🎯 Propósito

Panel de gestión de servidores de juego. **Hobby con esteroides.** Si crece, podría independizarse de Bildung.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `ghcr.io/getarcaneapp/arcane:latest` |
| Puerto interno | `3552` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Base de datos | **Sí** |
| docker.sock | Gestión de contenedores de juegos | **Sí** |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `ARCANE_PG_USER` | **Sí** | Usuario postgres |
| `ARCANE_PG_PASSWORD` | **Sí** | Contraseña postgres |
| `ARCANE_PG_DB` | **Sí** | Base de datos |
| `ARCANE_ENCRYPTION_KEY` | **Sí** | Clave de encriptación interna |
| `ARCANE_JWT_SECRET` | **Sí** | Secreto JWT |
| `DOMINIO` | **Sí** | Dominio base |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Tipo | Nombre/Path | Path interno | Contenido |
|---|---|---|---|
| Named volume | `arcane-data` | `/app/data` | Datos de aplicación |
| Bind mount | `./data/arcane/projects` | `/app/data/projects` | Mundos, configs de servidores |

---

## 🚀 Deploy

```bash
docker compose up -d arcane
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://arcane.${DOMINIO}
# Esperado: 200
```

---

## 🧹 Backup

```bash
# Base de datos
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres arcane | gzip > /backups/arcane-$(date +%Y%m%d).sql.gz"

# Proyectos (mundos)
tar -czf arcane-projects-$(date +%Y%m%d).tar.gz ./data/arcane/projects/
```

---

## 🔄 Restore

```bash
docker compose stop arcane
gunzip < arcane-YYYYMMDD.sql.gz | docker compose exec -T postgres psql -U postgres -d arcane
tar -xzf arcane-projects-YYYYMMDD.tar.gz
docker compose up -d arcane
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | `ARCANE_ENCRYPTION_KEY` o `JWT_SECRET` incorrectos | No cambiar después del primer deploy |
| No ve proyectos | `PROJECTS_DIRECTORY` mal configurado | Debe apuntar a `/app/data/projects` |
| No crea servidores | docker.sock sin permisos | Verificar: `ls -la /var/run/docker.sock` |
| Consume mucha RAM | Servidores de juegos activos | Limitar desde la UI de Arcane o con constraints Docker |
