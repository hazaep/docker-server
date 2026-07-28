# 📘 MariaDB — Service Specification v1.0

**Capa:** 🟣 Experimento | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Base de datos exclusiva para WordPress. **No la usa ningún otro servicio del ecosistema.** Si WordPress se archiva, MariaDB se archiva con él.

---

## 2. Definición del Servicio

> MariaDB 10.5 en imagen ARM específica (`tobi312/rpi-mariadb`). Named volume para persistencia. Sin exposición externa — solo WordPress lo consume.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Exclusividad WordPress** | Ningún otro servicio debe usar esta base de datos |
| **Imagen ARM nativa** | Versión específica para Raspberry Pi. No usar imágenes genéricas |
| **Sin réplica** | Instancia única. Si se pierde, WordPress se queda sin backend |

---

## 3. Contrato de Infraestructura

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Puerto interno** | `3306` |
| **Hostname** | `mariadb` |
| **Base default** | `mariadb` |

### 🔹 Dependencias

```text
mariadb
  └── (ninguna)
```

### 🔹 ¿Quién depende de MariaDB?

```text
mariadb
  └── wordpress (🟣 Experimento)
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre | Path interno |
|---|---|---|
| Named volume | `mariadb_data` | `/var/lib/mysql` |

---

## 5. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps mariadb` → Up |
| **CA-2** | Acepta conexiones | `docker compose exec mariadb mysql -u mariadb -p"${MARIA_PASS}" -e "SELECT 1"` → `1` |
| **CA-3** | WordPress la ve | WordPress admin panel carga sin errores de base de datos |
| **CA-4** | Datos sobreviven a restart | Crear post en WP → `restart mariadb` → el post sigue |

---

📌 **Nota SDD:** MariaDB es el único servicio que depende de una imagen ARM específica no oficial. Si `tobi312/rpi-mariadb` deja de recibir updates, migrar a `mariadb:10.5` oficial (que ya soporta ARM64).
