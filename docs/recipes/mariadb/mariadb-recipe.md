# MariaDB — Recipe v1.0

## 🎯 Propósito

Base de datos exclusiva de WordPress. **Si WordPress se va, MariaDB se va con él.**

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `tobi312/rpi-mariadb:10.5-debian` |
| Puerto interno | `3306` |
| Reinicio | `always` |

> ⚠️ Imagen ARM específica no oficial. Si deja de recibir updates, migrar a `mariadb:10.5` oficial.

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| — | — | Ninguna |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `MARIA_ROOT_PASS` | **Sí** | Contraseña root |
| `MARIA_PASS` | **Sí** | Contraseña del usuario `mariadb` |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Tipo | Nombre | Path interno |
|---|---|---|
| Named volume | `mariadb_data` | `/var/lib/mysql` |

---

## 🚀 Deploy

```bash
docker compose up -d mariadb
```

---

## 🩺 Health check

```bash
docker compose exec mariadb mysql -u root -p"${MARIA_ROOT_PASS}" -e "SELECT 1"
# Esperado: 1

# Ver bases de datos
docker compose exec mariadb mysql -u root -p"${MARIA_ROOT_PASS}" -e "SHOW DATABASES"
```

---

## 🧹 Backup

```bash
docker compose exec mariadb mysqldump -u root -p"${MARIA_ROOT_PASS}" --all-databases | gzip > mariadb-backup-$(date +%Y%m%d).sql.gz
```

---

## 🔄 Restore

```bash
docker compose stop wordpress
gunzip < mariadb-backup-YYYYMMDD.sql.gz | docker compose exec -T mariadb mysql -u root -p"${MARIA_ROOT_PASS}"
docker compose up -d wordpress
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | Named volume corrupto | Recrear volumen: `docker compose down -v mariadb && docker compose up -d mariadb` |
| WordPress no conecta | Contraseña incorrecta | Verificar `MARIA_PASS` coincide en mariadb y wordpress |
| `mysql_native_password` error | Plugin de auth | No aplica. La imagen `tobi312/rpi-mariadb` maneja esto |
