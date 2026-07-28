# 📘 Postgres — Service Specification v1.0

**Capa:** 🔴 Fundación | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Base de datos relacional principal del ecosistema Bildung. **Es la memoria persistente de todo lo que importa.** Si cae, n8n, nocodb, open-webui (cuando migre) y cualquier servicio que requiera estado relacional deja de funcionar.

---

## 2. Definición del Servicio

> PostgreSQL con extensión pgvector. Un solo clúster, sin réplicas. Provee almacenamiento relacional + búsqueda vectorial para todos los servicios del ecosistema.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Schema-per-tenant** | Cada servicio usa su propia base de datos lógica |
| **Sin lógica de negocio** | Postgres almacena y consulta; no ejecuta triggers complejos ni stored procedures |
| **Backup es sagrado** | Si se pierde postgres sin backup, se pierde el ecosistema completo |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 512 MB | Sí |
| **Disco** | Mínimo 5 GB (crece con los datos) | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |
| **Puerto** | `5432` (interno, no expuesto al host) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Puerto interno** | `5432` |
| **Hostname en red** | `postgres` |
| **Usuario admin** | `${POSTGRES_USER}` |
| **Base default** | `${POSTGRES_DB}` |

### 🔹 Dependencias

```text
postgres
  └── (ninguna — es la raíz de todas las dependencias)
```

### 🔹 ¿Quién depende de postgres?

```text
postgres
  ├── n8n          (🟠 Core)
  ├── nocodb       (🟠 Core)
  ├── flowise      (🟡 Extensión)
  ├── maybe        (💤 Archivado)
  ├── memos        (💤 Archivado)
  ├── resume       (🟣 Experimento)
  ├── rag-api      (💤 Archivado)
  └── arcane       (🟣 Experimento)
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `postgres_data` | `/var/lib/postgresql` | Datos, índices, WAL, configuraciones |

> Named volume sin configuración personalizada: PostgreSQL usa sus defaults internos. No se requieren archivos `postgresql.conf` ni `pg_hba.conf` externos.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|---|
| `POSTGRES_USER` | **Sí** | Usuario administrador |
| `POSTGRES_PASSWORD` | **Sí** | Contraseña del administrador |
| `POSTGRES_DB` | **Sí** | Base de datos por defecto |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| #        | Criterio                   | Verificación                                                                                                          |     |
| -------- | -------------------------- | --------------------------------------------------------------------------------------------------------------------- | --- |
| **CA-1** | Contenedor inicia          | `docker compose ps postgres` → Up                                                                                     |     |
| **CA-2** | Acepta conexiones          | `docker compose exec postgres pg_isready` → accepting connections                                                     |     |
| **CA-3** | pgvector instalado         | `docker compose exec postgres psql -U postgres -c "SELECT * FROM pg_available_extensions WHERE name='vector'"` → fila |     |
| **CA-4** | Datos sobreviven a restart | Escribir → `restart` → leer                                                                                           |     |
| **CA-5** | Backup funciona            | `docker compose exec pgbackup sh -c "pg_dump..."` → archivo                                                           |     |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Sin réplica** | Instancia única. Si cae, no hay failover automático |
| **Sin pooling nativo** | Las apps usan conexiones directas. Si muchas apps saturan, implementar PgBouncer |
| **No expuesto al host** | Solo accesible desde la red Docker interna |

---

📌 **Nota SDD:** Esta spec define el contrato de infraestructura. La receta (`postgres-recipe.md`) contiene los procedimientos operativos (backup, restore, migración).
