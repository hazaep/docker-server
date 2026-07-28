# pgAdmin — Recipe v1.0

## 🎯 Propósito

Administrador gráfico de PostgreSQL. **Si cae, usas `psql` por terminal.** Conveniencia, no necesidad.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `dpage/pgadmin4:latest` |
| Puerto interno | `80` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| postgres | Base de datos a administrar | No (pgadmin inicia sin él) |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `PGADMIN_DEFAULT_EMAIL` | **Sí** | Email de login |
| `PGADMIN_DEFAULT_PASSWORD` | **Sí** | Contraseña |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Path contenedor | Path host | Contenido | Permisos |
|---|---|---|---|
| `/var/lib/pgadmin` | `./data/pgadmin/` | Conexiones guardadas, preferencias | ⚠️ `chown 5050:5050` |

---

## 🚀 Deploy

```bash
docker compose up -d pgadmin
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://pgadmin.${DOMINIO}
# Esperado: 200
```

---

## 🔌 Conectar a postgres

1. Abrir `https://pgadmin.${DOMINIO}`
2. Login con `PGADMIN_DEFAULT_EMAIL` / `PGADMIN_DEFAULT_PASSWORD`
3. Add New Server:
   - Name: `Bildung`
   - Host: `postgres`
   - Port: `5432`
   - Username: `${POSTGRES_USER}`
   - Password: `${POSTGRES_PASSWORD}`

---

## 🧹 Backup

```bash
tar -czf pgadmin-backup-$(date +%Y%m%d).tar.gz ./data/pgadmin/
```

---

## 🔄 Restore

```bash
docker compose stop pgadmin
tar -xzf pgadmin-backup-YYYYMMDD.tar.gz
docker compose up -d pgadmin
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| Error de permisos al iniciar | Dueño del volumen incorrecto | `sudo chown -R 5050:5050 ./data/pgadmin/` |
| No conecta a postgres | Hostname incorrecto | Usar `postgres` (nombre del contenedor en la red Docker) |
| Login no funciona | Credenciales cambiadas | Verificar `PGADMIN_DEFAULT_*` en `.env` |
| Sesión expira muy rápido | Configuración interna de pgadmin | Ajustar `SESSION_EXPIRATION_TIME` en config_local.py |
