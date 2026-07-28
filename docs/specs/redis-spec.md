# 📘 Redis — Service Specification v1.0

**Capa:** 🔴 Fundación | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Almacén clave-valor en memoria. **Es la memoria volátil del ecosistema.** Provee caché, colas de trabajos y almacenamiento de sesiones. Sin él, los servicios no mueren, pero se degradan significativamente.

---

## 2. Definición del Servicio

> Redis 7 Alpine. Instancia única con autenticación por contraseña. Sin persistencia RDB/AOF habilitada por defecto (los datos críticos están en postgres).

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Efímero por diseño** | Redis es caché, no fuente de verdad. Si se pierden los datos, el sistema se reconstruye |
| **Contraseña obligatoria** | Sin `requirepass`, Redis rechaza conexiones |
| **Una instancia, un propósito** | Sin clustering. La simplicidad es la prioridad |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 128 MB | Sí |
| **Disco** | Mínimo 500 MB (dump.rdb si se habilita) | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Puerto interno** | `6379` |
| **Hostname en red** | `redis` |
| **Auth** | Contraseña (`${REDIS_PASSWORD}`) |

### 🔹 Dependencias

```text
redis
  └── (ninguna)
```

### 🔹 ¿Quién depende de redis?

```text
redis
  ├── n8n          (🟠 Core) — colas de ejecución
  ├── flowise      (🟡 Extensión) — colas + caché
  ├── nocodb       (🟠 Core) — caché
  ├── maybe        (💤 Archivado) — colas background jobs
  └── resume       (🟣 Experimento) — sesiones
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `redis_data` | `/data` | dump.rdb (si se habilita persistencia) |

> Named volume sin configuración personalizada: Redis usa sus defaults. Toda la configuración va por `command` (requirepass) y variables de entorno.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|---|
| `REDIS_PASSWORD` | **Sí** | Contraseña para `requirepass` |
| `REDIS_HOST` | No (default: `redis`) | Hostname para apps que se conectan |
| `REDIS_PORT` | No (default: `6379`) | Puerto |
| `REDIS_URL` | No | URL de conexión completa para apps que la requieren |
| `TZ` | No | Zona horaria |

### 🔹 Regla

> Si `REDIS_PASSWORD` no está definida o cambia, **todas** las apps que dependen de redis deben actualizar su configuración y recrearse.

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps redis` → Up |
| **CA-2** | Requiere autenticación | `docker compose exec redis redis-cli PING` → `NOAUTH` |
| **CA-3** | Acepta conexiones autenticadas | `docker compose exec redis redis-cli -a "${REDIS_PASSWORD}" PING` → `PONG` |
| **CA-4** | Datos en memoria funcionan | `SET test 1` → `GET test` → `1` |
| **CA-5** | Reinicio no corrompe | `restart` → PING responde |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Persistencia** | No habilitada por defecto |
| **Maxmemory** | Sin límite explícito (la Raspberry Pi lo limita) |
| **No expuesto al host** | Solo red Docker interna |

---

📌 **Nota SDD:** Redis es Fundación por las colas de n8n. Si se cae redis, los workflows de n8n que usan colas Bull dejan de ejecutarse. El sistema no colapsa, pero la automatización se detiene.
