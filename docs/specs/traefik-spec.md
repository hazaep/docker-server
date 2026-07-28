# 📘 Traefik — Service Specification v1.0

**Capa:** 🔴 Fundación | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Reverse proxy y punto único de entrada HTTP al ecosistema Bildung. **Todo el tráfico web pasa por Traefik.** Sin él, ningún servicio es accesible desde un navegador.

---

## 2. Definición del Servicio

> Traefik v3 con configuración dinámica vía Docker labels y archivos YAML. Solo expone HTTP (puerto 80). TLS lo maneja Cloudflare en el edge.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Cero configuración manual por servicio** | Cada servicio se anuncia con labels Docker. Traefik descubre y enruta automáticamente |
| **HTTP solamente** | Cloudflare termina TLS. Entre Cloudflare y Traefik el tráfico es HTTP plano |
| **Dashboard interno** | Accesible solo vía `traefik.${DOMINIO}` con auth |
| **Middleware reutilizable** | Las reglas comunes (headers, rate-limit, auth) se definen una vez en archivos YAML |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 128 MB | Sí |
| **Docker socket** | `/var/run/docker.sock` (read-only) | **Sí** — sin esto no descubre servicios |
| **Directorio dynamic/** | `./data/traefik/dynamic/` con archivos YAML | **Sí** — middlewares y configuraciones file-based |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **Puerto host** | `80` (HTTP) |
| **Endpoint dashboard** | `http://traefik.${DOMINIO}` |
| **API interna** | `:8080` (solo red interna) |
| **Métricas Prometheus** | `:8080/metrics` |

### 🔹 Dependencias

```text
traefik
  ├── cloudflared (recibe tráfico desde el túnel)
  └── docker.sock (descubre servicios vía labels)
```

---

## 4. Archivos de Configuración

| Path | Propósito |
|---|---|
| `./data/traefik/dynamic/cloudflare.yml` | Middleware que inyecta headers `X-Forwarded-Proto: https` |

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `DOMINIO` | **Sí** | Dominio base para las reglas de ruteo |
| `TZ` | No | Zona horaria |
| `TRAEFIK_AUTH` | No | Credenciales para el dashboard (si se habilita auth básica) |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps traefik` → Up |
| **CA-2** | Dashboard accesible | `curl -s http://traefik.${DOMINIO}/dashboard/` → HTML |
| **CA-3** | Descubre servicios vía labels | `docker compose ps` → servicios con labels `traefik.enable=true` aparecen en el dashboard |
| **CA-4** | Métricas expuestas | `curl -s http://traefik:8080/metrics` → métricas en formato Prometheus |
| **CA-5** | Reinicio no rompe ruteo | `restart` → los servicios siguen siendo accesibles |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Solo HTTP** | Sin TLS. Cloudflare es responsable del cifrado |
| **Sin alta disponibilidad** | Una instancia. Si cae, el ecosistema es inaccesible |
| **Docker socket** | Acceso read-only. No puede crear/eliminar contenedores |

---

## 8. Por qué HTTP y no HTTPS

Cloudflare actúa como terminador TLS en el edge:

```text
[Navegador] ── HTTPS ──→ [Cloudflare] ── HTTP ──→ [Traefik :80]
```

Ventajas:
- Traefik no gestiona certificados (simplicidad)
- Cloudflare tiene mejor CDN, DDoS protection y caching
- El tráfico entre Cloudflare y la Raspberry Pi viaja por el túnel cloudflared (encriptado)

---

📌 **Nota SDD:** Traefik es el servicio más difícil de reemplazar en el ecosistema. Cada label `traefik.*` en 20+ servicios depende de su contrato. Cualquier cambio en la API de configuración de Traefik debe probarse contra todos los servicios antes de aplicar.
