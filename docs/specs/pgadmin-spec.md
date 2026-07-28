# 📘 pgAdmin — Service Specification v1.0

**Capa:** 🟢 Herramienta | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Interfaz gráfica web para administrar PostgreSQL. **Es una ventana a la base de datos, no la base de datos misma.** Si cae, postgres sigue funcionando y se puede administrar por CLI.

---

## 2. Definición del Servicio

> pgAdmin 4 auto-hospedado, conectándose a `postgres:5432` vía la red Docker interna. Expuesto vía Traefik con auth por email/contraseña.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Conveniencia, no necesidad** | Todo lo que hace pgAdmin se puede hacer con `psql` |
| **Auth obligatoria** | Sin login no se accede |
| **Estado en volumen** | Las conexiones guardadas y preferencias persisten |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 128 MB | Sí |
| **Disco** | `./data/pgadmin/` | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://pgadmin.${DOMINIO}` |
| **Puerto interno** | `80` |

### 🔹 Dependencias

```text
pgadmin
  └── postgres (🔴 Fundación) — se conecta pero no depende de él para iniciar
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Contenido | Permisos |
|---|---|---|---|
| `/var/lib/pgadmin` | `./data/pgadmin/` | Conexiones guardadas, preferencias, sesiones | `chown 5050:5050` |

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `PGADMIN_DEFAULT_EMAIL` | **Sí** | Email de login |
| `PGADMIN_DEFAULT_PASSWORD` | **Sí** | Contraseña de login |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps pgadmin` → Up |
| **CA-2** | UI accesible | `curl -s https://pgadmin.${DOMINIO}` → login page |
| **CA-3** | Login funciona | Navegar → ingresar credenciales → dashboard |
| **CA-4** | Conecta a postgres | Agregar server `postgres:5432` → conexión exitosa |
| **CA-5** | Conexiones sobreviven | Agregar server → `restart` → el server sigue guardado |

---

📌 **Nota SDD:** pgAdmin está en Herramienta, no en Core. Su caída no afecta a ningún otro servicio. Es puramente una interfaz de administración.
