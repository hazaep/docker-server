# 📘 pgBackup — Service Specification v1.0

**Capa:** 🟢 Herramienta | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Backups automatizados de PostgreSQL. **Es el seguro de vida del ecosistema.** Si cae, no pasa nada hoy. Si no existe cuando postgres falla, el ecosistema pierde todo su estado.

---

## 2. Definición del Servicio

> Contenedor especializado que ejecuta `pg_dump` contra postgres en un schedule configurable. Almacena los backups comprimidos en un volumen persistente.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Schedule, no continuo** | Corre una vez al día, no es un servicio always-on |
| **Rotación automática** | Elimina backups viejos según política de retención |
| **Backup es sagrado** | Si pgbackup no existe, no hay red de seguridad |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 64 MB | Sí |
| **Disco** | `./data/backups/` — crece con el tiempo | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Backups** | Archivos `.gz` en `./data/backups/` |
| **Schedule** | `@daily` (una vez al día) |

### 🔹 Dependencias

```text
pgbackup
  └── postgres (🔴 Fundación) — obligatorio: sin postgres no hay nada que respaldar
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Contenido |
|---|---|---|
| `/backups` | `./data/backups/` | Archivos `.sql.gz` nombrados por fecha |

### 🔹 Política de retención

| Regla | Valor |
|---|---|
| `BACKUP_KEEP_DAYS` | 2 (backups diarios de los últimos 2 días) |
| `BACKUP_KEEP_WEEKS` | 2 (backups semanales de las últimas 2 semanas) |
| `BACKUP_KEEP_MONTHS` | 1 (backups mensuales del último mes) |

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `BACKUP_HOST` | **Sí** | Hostname de postgres |
| `BACKUP_DATABASE` | **Sí** | Lista de bases de datos separadas por coma |
| `BACKUP_USER` | **Sí** | Usuario de postgres con permisos de lectura |
| `BACKUP_PASSWORD` | **Sí** | Contraseña del usuario |
| `TZ` | No | Zona horaria |

### 🔹 Regla

> Si `BACKUP_DATABASE` cambia (se agrega o quita una base de datos), pgbackup debe recrearse para aplicar el cambio.

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps pgbackup` → Up |
| **CA-2** | Primer backup se ejecuta | `docker compose logs pgbackup` → `Dumping` |
| **CA-3** | Archivos generados | `ls ./data/backups/` → archivos `.sql.gz` |
| **CA-4** | Rotación funciona | Después de 3 días, solo hay backups dentro de la política de retención |
| **CA-5** | Backup es restaurable | `gunzip < backup.sql.gz \| docker compose exec -T postgres psql -U postgres` → sin errores |

---

## 7. Restauración

```bash
# 1. Detener apps que escriben en postgres
docker compose stop n8n nocodb flowise

# 2. Restaurar
gunzip < ./data/backups/n8n_20250101.sql.gz | \
  docker compose exec -T postgres psql -U postgres -d n8n

# 3. Levantar apps
docker compose up -d n8n nocodb flowise
```

---

📌 **Nota SDD:** pgBackup es Herramienta, no Fundación, porque no es parte de la cadena de ejecución. Pero es la Herramienta más importante del ecosistema. Si postgres muere y pgbackup no existe, el costo es total.
