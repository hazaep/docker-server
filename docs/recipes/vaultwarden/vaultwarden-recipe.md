# Vaultwarden — Recipe v1.0

## 🎯 Propósito

Gestor de contraseñas self-hosted. **Crítico para ti, irrelevante para Bildung.** Si cae, pierdes acceso a tus contraseñas; el ecosistema sigue.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `vaultwarden/server:latest` |
| Puerto interno | `80` |
| WebSocket | `3012` (para sync en tiempo real) |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Backend de base de datos | **Sí** |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `VAULTWARDEN_DATABASE_URL` | **Sí** | String de conexión PostgreSQL |
| `VAULTWARDEN_WEBSOCKET_ENABLED` | **Sí** | `true` para sync en tiempo real |
| `VAULTWARDEN_SIGNUPS_ALLOWED` | No | `true` / `false` |
| `VAULTWARDEN_ADMIN_TOKEN` | No | Token Argon2 para panel admin |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido |
|---|---|---|
| `/data` | `./data/vaultwarden/` | Attachments, icon cache |

> Los datos principales (usuarios, contraseñas, organizaciones) están en Postgres.

---

## 🚀 Deploy

```bash
docker compose up -d vaultwarden
```

---

## 🩺 Health check

```bash
# UI
curl -s -o /dev/null -w "%{http_code}" https://vault.${DOMINIO}
# Esperado: 200

# WebSocket (requiere wscat: npm install -g wscat)
wscat -c wss://vault.${DOMINIO}/notifications/hub
# Esperado: Connected

# Ver usuarios en postgres
docker compose exec postgres psql -U postgres -d vaultwarden -c "SELECT email FROM users"
```

---

## 🔑 Panel de administración

```bash
# Acceder
open https://vault.${DOMINIO}/admin

# Ingresar ADMIN_TOKEN
```

Funciones:
- Ver todos los usuarios
- Borrar usuarios
- Forzar rotación de claves
- Ver logs de acceso

---

## 🧹 Backup

```bash
# Base de datos (Postgres)
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres vaultwarden | gzip > /backups/vaultwarden-$(date +%Y%m%d).sql.gz"

# Attachments
tar -czf vaultwarden-data-$(date +%Y%m%d).tar.gz ./data/vaultwarden/
```

---

## 🔄 Restore

```bash
docker compose stop vaultwarden
gunzip < vaultwarden-YYYYMMDD.sql.gz | docker compose exec -T postgres psql -U postgres -d vaultwarden
tar -xzf vaultwarden-data-YYYYMMDD.tar.gz
docker compose up -d vaultwarden
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | `DATABASE_URL` incorrecta | Verificar formato: `postgresql://user:pass@postgres:5432/vaultwarden` |
| Sync no funciona | WebSocket no habilitado | `VAULTWARDEN_WEBSOCKET_ENABLED=true` |
| No se puede registrar | `SIGNUPS_ALLOWED=false` | Cambiar a `true`, registrar, volver a `false` |
| Admin panel no accesible | `ADMIN_TOKEN` no configurado | Generar: `docker compose exec vaultwarden ./vaultwarden hash` |
| Contraseñas no se guardan | Base de datos no inicializada | La DB se crea en el primer arranque. Verificar logs |
