# 📘 phpMyAdmin — Service Specification v1.0

**Capa:** 🟢 Herramienta | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Interfaz gráfica web para administrar MariaDB. **Es una ventana a la base de datos de WordPress.** Si cae, MariaDB sigue funcionando y se puede administrar por CLI.

---

## 2. Definición del Servicio

> phpMyAdmin conectándose a MariaDB vía la red Docker interna. Expuesto vía Traefik. Sin volumen persistente — es puramente una UI.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Conveniencia, no necesidad** | Todo lo que hace phpMyAdmin se puede hacer con `mysql` CLI |
| **Modo arbitrario** | `PMA_ARBITRARY=1` — permite conectarse a cualquier host MySQL |
| **Efímero** | Sin estado. Si se recrea, no se pierde nada |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 64 MB | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://phpdb.${DOMINIO}` |
| **Puerto interno** | `80` |

### 🔹 Dependencias

```text
phpadmin
  └── mariadb (🟣 Experimento) — se conecta pero no depende de él para iniciar
```

---

## 4. Sin volúmenes persistentes

phpMyAdmin es 100% efímero. No almacena configuraciones, conexiones ni preferencias.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `MARIA_ROOT_PASS` | **Sí** | Contraseña root de MariaDB |
| `MARIA_PASS` | No | Contraseña del usuario `mariadb` |
| `PMA_ARBITRARY` | No | Permitir conexión a cualquier host (default: `1`) |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps phpadmin` → Up |
| **CA-2** | UI accesible | `curl -s https://phpdb.${DOMINIO}` → login page |
| **CA-3** | Login funciona | Ingresar credenciales MariaDB → dashboard |
| **CA-4** | Ve bases de datos | Lista de bases de datos visible |

---

📌 **Nota SDD:** phpMyAdmin existe solo porque WordPress usa MariaDB y `mysql` CLI en una Raspberry Pi headless es incómodo. Si WordPress se archiva, phpMyAdmin se archiva con él.
