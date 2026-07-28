# 📘 Watchtower — Service Specification v1.0

**Capa:** 🟠 Core | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Actualizador automático de imágenes Docker. **Es el sistema inmunológico del ecosistema.** Sin él, las imágenes envejecen, las vulnerabilidades se acumulan y la actualización manual consume tiempo y atención.

---

## 2. Definición del Servicio

> Watchtower monitorea los contenedores en ejecución, detecta nuevas versiones de sus imágenes, las descarga y recrea los contenedores automáticamente. Solo actualiza contenedores con la label `com.centurylinklabs.watchtower.enable=true`.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Opt-in por label** | Solo actualiza lo que explícitamente lo pide |
| **Horario controlado** | Corre una vez al día a las 4 AM |
| **Limpieza automática** | Elimina imágenes viejas después de actualizar |
| **Sin notificaciones** | Por ahora no notifica. Si algo se rompe, el monitoring lo detecta |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 64 MB | Sí |
| **Red** | Acceso a internet + acceso a la red `services` | **Sí** |
| **Docker socket** | `/var/run/docker.sock` (read-write) | **Sí** — necesita crear/recrear contenedores |
| **Timezone** | `/etc/timezone` y `/etc/localtime` del host | Sí — para ejecutar en el horario correcto |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Actualizaciones** | Contenedores recreados con nuevas imágenes |
| **Schedule** | `0 0 4 * * *` (todos los días a las 4 AM) |

### 🔹 Dependencias

```text
watchtower
  └── docker.sock (host) — no depende de ningún otro servicio
```

---

## 4. Sin volúmenes persistentes

Watchtower no almacena estado. Todo está en la memoria del contenedor y el socket de Docker.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Default | Descripción |
|---|---|---|---|
| `WATCHTOWER_CLEANUP` | No | `false` | Eliminar imágenes viejas tras actualizar |
| `WATCHTOWER_REMOVE_VOLUMES` | No | `false` | Eliminar volúmenes huérfanos (⚠️ peligroso) |
| `WATCHTOWER_SCHEDULE` | No | — | Cron expression (vacío = watch inmediato) |
| `TZ` | No | `UTC` | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps watchtower` → Up |
| **CA-2** | Detecta imágenes | `docker compose logs watchtower` → `Checking all containers` |
| **CA-3** | Respeta labels | Solo revisa contenedores con `watchtower.enable=true` |
| **CA-4** | Schedule activo | `docker compose logs watchtower` → `Waiting for next run` |
| **CA-5** | No rompe nada | Después de una actualización programada, los servicios críticos siguen respondiendo |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Escritura en Docker socket** | Puede crear/recrear contenedores. Es un riesgo si se compromete |
| **Sin rollback automático** | Si una actualización rompe algo, hay que restaurar manualmente |
| **Sin notificaciones** | No sabes que actualizó hasta que ves los logs |
| **Una vez al día** | Si sale un CVE crítico, hay que esperar al ciclo o ejecutar manualmente |

---

## 8. Estrategia de seguridad

Los contenedores con watchtower habilitado deben usar tags `:latest` o versiones flotantes. Si un servicio usa una versión fija (`:v1.5.1`), watchtower no lo actualizará aunque exista `:v1.5.2`.

| Tipo de tag | ¿Watchtower actualiza? |
|---|---|
| `:latest` | ✅ Sí |
| `:stable` | ✅ Sí |
| `:v1.5.1` | ❌ No |
| `:pg18` | ❌ No |

---

📌 **Nota SDD:** Watchtower está en Core y no en Extensión por una razón: en un ecosistema con 20+ servicios, las actualizaciones manuales no escalan. Un servicio sin actualizar por 6 meses es una vulnerabilidad andante. Watchtower convierte eso en riesgo cero operativo.
