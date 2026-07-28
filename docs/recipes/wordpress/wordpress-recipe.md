# WordPress — Recipe v1.0

## 🎯 Propósito

CMS / blog personal. **Lienzo para escribir, no infraestructura.** Si mañana migras a un static site generator, nada se rompe.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `wordpress:latest` |
| Puerto interno | `80` |
| Reinicio | `always` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| mariadb | Base de datos | **Sí** |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `MARIA_PASS` | **Sí** | Contraseña del usuario `mariadb` |
| `DOMINIO` | **Sí** | Dominio base |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `wordp_data` | `/var/www/html` | Temas, plugins, uploads, wp-config |

---

## 🚀 Deploy

```bash
docker compose up -d wordpress
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://wordp.${DOMINIO}
# Esperado: 200

# Admin panel
curl -s -o /dev/null -w "%{http_code}" https://wordp.${DOMINIO}/wp-admin
# Esperado: 200
```

---

## ⚙️ Setup inicial

1. Abrir `https://wordp.${DOMINIO}`
2. Completar instalación (idioma, nombre del sitio, usuario admin, contraseña)
3. La base de datos ya está configurada vía variables de entorno

---

## 🧹 Backup

```bash
# Archivos (temas, plugins, uploads)
docker run --rm -v wordp_data:/data -v $(pwd):/backup alpine tar czf /backup/wordpress-files-$(date +%Y%m%d).tar.gz -C /data .

# Base de datos
docker compose exec mariadb mysqldump -u root -p"${MARIA_ROOT_PASS}" mariadb | gzip > wordpress-db-$(date +%Y%m%d).sql.gz
```

---

## 🔄 Restore

```bash
docker compose stop wordpress

# Restaurar archivos
docker run --rm -v wordp_data:/data -v $(pwd):/backup alpine tar xzf /backup/wordpress-files-YYYYMMDD.tar.gz -C /data

# Restaurar base de datos
gunzip < wordpress-db-YYYYMMDD.sql.gz | docker compose exec -T mariadb mysql -u root -p"${MARIA_ROOT_PASS}" mariadb

docker compose up -d wordpress
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| "Error establishing database connection" | MariaDB caído o credenciales mal | `docker compose ps mariadb` |
| Pantalla blanca | Plugin incompatible | Desactivar plugins vía `wp option update` |
| No puedo subir imágenes | Permisos de `wp-content/uploads` | `docker compose exec wordpress chown -R www-data:www-data /var/www/html/wp-content` |
| Demasiado lento | Muchos plugins | Desactivar los no esenciales |
