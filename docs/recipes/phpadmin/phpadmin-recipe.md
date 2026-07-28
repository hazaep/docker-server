# phpMyAdmin — Recipe v1.0

## 🎯 Propósito

Administrador gráfico de MariaDB. **Si cae, usas `mysql` CLI.** Existe solo porque WordPress usa MariaDB y una terminal en Raspberry Pi es incómoda.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `phpmyadmin` |
| Puerto interno | `80` |
| Reinicio | `always` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| mariadb | Base de datos a administrar | No (phpadmin inicia sin él) |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `MARIA_ROOT_PASS` | No | Contraseña root (para login directo) |
| `MARIA_PASS` | No | Contraseña del usuario `mariadb` |
| `PMA_ARBITRARY` | No | `1` = permitir cualquier host |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

Sin volúmenes. Completamente efímero.

---

## 🚀 Deploy

```bash
docker compose up -d phpadmin
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://phpdb.${DOMINIO}
# Esperado: 200
```

---

## 🔌 Conectar a MariaDB

1. Abrir `https://phpdb.${DOMINIO}`
2. Server: `mariadb`
3. Username: `root` (o `mariadb`)
4. Password: `${MARIA_ROOT_PASS}` (o `${MARIA_PASS}`)

---

## 🧹 Backup

No requiere backup. Sin estado. La base de datos se respalda con:

```bash
docker compose exec mariadb mysqldump -u root -p"${MARIA_ROOT_PASS}" --all-databases | gzip > mariadb-backup-$(date +%Y%m%d).sql.gz
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No conecta | MariaDB caído | `docker compose ps mariadb` |
| `Cannot log in to MySQL` | Credenciales incorrectas | Verificar `MARIA_ROOT_PASS` en `.env` |
| #2002 - Connection refused | Hostname `mariadb` no resuelve | Verificar que ambos están en la red `services` |
