# 📘 Uptime Kuma — Service Specification v1.0

**Capa:** ⚪ Observabilidad | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Monitor de disponibilidad de servicios. **Es el guardián silencioso.** Te avisa cuando algo se cae, pero no evita que nada se caiga. Sin él, un servicio puede estar roto durante horas sin que lo notes.

---

## 2. Definición del Servicio

> Uptime Kuma con base de datos SQLite interna. Monitorea endpoints HTTP, TCP, DNS y ping. Soporta notificaciones por Telegram, Discord, email, y webhook. UI web con dashboard de estado.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Monitoreo externo** | Hace probes desde dentro de la red Docker. Ve lo mismo que un usuario |
| **Notificaciones asíncronas** | Si algo cae, te avisa. Si no configuras notificaciones, solo ves el dashboard |
| **Estado en SQLite** | La configuración de monitores persiste en volumen |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 128 MB | Sí |
| **Disco** | `./data/uptime-kuma/` (SQLite) | Sí |
| **Red** | Acceso a la red `services` (bridge) | **Sí** — necesita alcanzar los servicios que monitorea |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Dashboard** | `https://status.${DOMINIO}` |
| **Puerto interno** | `3001` |

### 🔹 Dependencias

```text
uptime-kuma
  └── (ninguna — monitorea servicios pero no depende de ellos)
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Contenido |
|---|---|---|
| `/app/data` | `./data/uptime-kuma/` | SQLite con monitores, historial, config de notificaciones |

### 🔹 Regla

> Si el volumen se pierde, se pierde la configuración de monitores y el historial. El ecosistema sigue funcionando; solo dejas de recibir alertas.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `TZ` | No | Zona horaria |

> Uptime Kuma se configura completamente desde la UI. No requiere variables de entorno adicionales.

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps uptime-kuma` → Up |
| **CA-2** | UI accesible | `curl -s https://status.${DOMINIO}` → login/setup page |
| **CA-3** | Setup inicial | Crear usuario admin desde la UI |
| **CA-4** | Monitorea servicios | Agregar monitor HTTP `https://n8n.${DOMINIO}` → status verde |
| **CA-5** | Historial sobrevive | Agregar monitor → `restart` → el monitor sigue |

---

## 7. Monitores sugeridos

| Servicio | Tipo | URL/Host | Prioridad |
|---|---|---|---|
| traefik | HTTP | `https://traefik.${DOMINIO}` | 🔴 |
| n8n | HTTP | `https://n8n.${DOMINIO}` | 🟠 |
| open-webui | HTTP | `https://webui.${DOMINIO}` | 🟠 |
| nocodb | HTTP | `https://nocodb.${DOMINIO}` | 🟠 |
| vaultwarden | HTTP | `https://vault.${DOMINIO}` | 🟡 |
| flowise | HTTP | `https://flowise.${DOMINIO}` | 🟡 |
| postgres | TCP | `postgres:5432` | 🔴 |
| redis | TCP | `redis:6379` | 🟠 |

---

## 8. Notificaciones (configuración manual vía UI)

| Canal | Cuándo usarlo |
|---|---|
| **Telegram** | Alertas críticas (postgres, traefik caídos) |
| **Discord** | Alertas de equipo |
| **Webhook** | Disparar workflow en n8n si algo cae |
| **Email** | Reportes diarios de uptime |

---

📌 **Nota SDD:** Uptime Kuma es el servicio más "set and forget" del ecosistema. Una vez configurados los monitores, no necesita mantenimiento. Su valor se revela cuando algo se cae a las 3 AM y recibes una notificación en lugar de descubrirlo a la mañana siguiente.
