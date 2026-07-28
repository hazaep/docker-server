# 📘 Vaultwarden — Service Specification v1.0

**Capa:** 🟢 Herramienta | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Gestor de contraseñas self-hosted compatible con Bitwarden. **Es crítico para ti como usuario, irrelevante para el ecosistema como sistema.** Si cae, Bildung sigue funcionando; tú pierdes acceso a tus contraseñas.

---

## 2. Definición del Servicio

> Vaultwarden (implementación ligera de Bitwarden) con PostgreSQL como backend. Expuesto vía Traefik con middleware `cloudflare-https`. WebSocket habilitado para sincronización en tiempo real.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Soberanía de datos** | Tus contraseñas no salen de tu Raspberry Pi |
| **Postgres, no SQLite** | Migrado de SQLite a Postgres para consistencia con el ecosistema |
| **WebSocket para sync** | Los clientes (móvil, navegador, extensión) se sincronizan en tiempo real |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 128 MB | Sí |
| **Disco** | `./data/vaultwarden/` | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://vault.${DOMINIO}` |
| **API** | `https://vault.${DOMINIO}/api` |
| **WebSocket** | `wss://vault.${DOMINIO}/notifications/hub` |
| **Puerto interno** | `80` |

### 🔹 Dependencias

```text
vaultwarden
  └── postgres (🔴 Fundación) — backend de base de datos
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Contenido |
|---|---|---|
| `/data` | `./data/vaultwarden/` | Attachments, icon cache, configuraciones residuales (datos principales en Postgres) |

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `VAULTWARDEN_DATABASE_URL` | **Sí** | String de conexión PostgreSQL |
| `VAULTWARDEN_WEBSOCKET_ENABLED` | **Sí** | Habilitar notificaciones en tiempo real |
| `VAULTWARDEN_SIGNUPS_ALLOWED` | No | Permitir registro de nuevos usuarios |
| `VAULTWARDEN_ADMIN_TOKEN` | No | Token para el panel de administración |
| `TZ` | No | Zona horaria |

### 🔹 Regla de seguridad

> `VAULTWARDEN_ADMIN_TOKEN` usa Argon2. Si se pierde, hay que generar uno nuevo con `vaultwarden hash`. El panel de admin permite gestionar usuarios, ver logs y forzar rotación de claves.

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps vaultwarden` → Up |
| **CA-2** | UI accesible | `curl -s https://vault.${DOMINIO}` → login page |
| **CA-3** | Login funciona | Ingresar credenciales → vault se abre |
| **CA-4** | Sync funciona | App móvil/extensión → sincroniza sin errores |
| **CA-5** | WebSocket activo | `wscat -c wss://vault.${DOMINIO}/notifications/hub` → connected |
| **CA-6** | Datos en postgres | `docker compose exec postgres psql -U postgres -d vaultwarden -c "SELECT count(*) FROM users"` → ≥ 1 |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Sin SMTP** | No configurado. Invitaciones y recuperación de cuenta no funcionan |
| **Un solo usuario** | Por ahora solo tu cuenta. `SIGNUPS_ALLOWED` determina si otros pueden registrarse |
| **Middleware cloudflare-https** | Requiere que el archivo `cloudflare.yml` exista en Traefik dynamic |

---

## 8. Migración SQLite → Postgres

Vaultwarden se migró de SQLite (default) a PostgreSQL. El string de conexión:

```
postgresql://vaultwarden:<password>@postgres:5432/vaultwarden
```

Si se pierde la base de datos `vaultwarden` en Postgres, Vaultwarden no arranca. Restaurar desde backup de postgres.

---

📌 **Nota SDD:** Vaultwarden es el servicio más personal del ecosistema. No le importa a Bildung, pero le importa a Hazael. Está en Herramienta porque su caída no afecta a ningún otro servicio. Sin embargo, si las contraseñas de administración del ecosistema viven aquí, su importancia subjetiva es Fundación.
