# 📘 Cloudflared — Service Specification v1.0

**Capa:** 🔴 Fundación | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Túnel seguro entre Cloudflare y la Raspberry Pi. **Es el puente que conecta el ecosistema Bildung con internet.** Sin él, los dominios no resuelven al servidor.

---

## 2. Definición del Servicio

> Cloudflare Tunnel (cloudflared) conectando mediante token. Sin archivos de configuración locales. Sin certificados. Un solo comando: `tunnel run --token`.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Configuración zero-touch** | Todo se configura en el dashboard de Cloudflare, no en el servidor |
| **Token, no archivos** | Si el token se filtra, se revoca desde Cloudflare sin tocar el servidor |
| **Sin auto-update** | `--no-autoupdate` — las actualizaciones las gestiona watchtower |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 64 MB | Sí |
| **Red** | Acceso a internet + acceso a la red `services` | **Sí** — sin internet no hay túnel |
| **Token** | `${CLOUDFLARE_TUNNEL_TOKEN}` válido | **Sí** — sin token el túnel no se establece |

### 🔹 Salida

| Recurso       | Valor                                 |
| ------------- | ------------------------------------- |
| **Túnel**     | Conecta `*.${DOMINIO}` → `traefik:80` |
| **Dashboard** | Zero Trust Dashboard de Cloudflare    |

### 🔹 Dependencias

```text
cloudflared
  └── traefik (reenvía tráfico HTTP a traefik:80)
```

---

## 4. Sin volúmenes, sin archivos

Cloudflared no almacena estado ni requiere archivos de configuración locales. El túnel se define completamente en el dashboard de Cloudflare Zero Trust. Si el contenedor se recrea, el túnel se restablece automáticamente con el token. No hay `config.yml`, no hay `cert.pem`, no hay volumen.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `CLOUDFLARE_TUNNEL_TOKEN` | **Sí** | Token JWT del túnel. Se obtiene del dashboard de Cloudflare Zero Trust |
| `TUNNEL_TOKEN` | **Sí** | Mismo valor, requerido por la imagen oficial |
| `TZ` | No | Zona horaria |

### 🔹 Regla de seguridad

> Si el token se compromete: 1) revocar en dashboard Cloudflare, 2) rotar en `.env`, 3) recrear contenedor. El tiempo de exposición es mínimo.

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps cloudflared` → Up |
| **CA-2** | Túnel establecido | `docker compose logs cloudflared` → `Registered tunnel connection` |
| **CA-3** | Tráfico enruta | `curl -s http://traefik.${DOMINIO}` → responde Traefik |
| **CA-4** | Reinicio reconecta | `restart` → túnel se restablece en <10 segundos |
| **CA-5** | Sin errores en loop | `docker compose logs cloudflared --tail 50` → sin errores de conexión recurrentes |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Dependencia externa** | Si Cloudflare está caído, el ecosistema es inaccesible desde internet (pero sigue funcionando en local) |
| **Sin failover** | Una instancia. Si se cae, hay que recrearla |
| **Sin暴露 de IP** | La IP real de la Raspberry Pi nunca se expone |

---

## 8. Arquitectura del túnel

```text
[Internet] ──→ cloudflared (túnel) ──→ traefik:80 ──→ servicios
                    │
            [Cloudflare Edge]
         (TLS termination, DDoS, CDN)
```

---

📌 **Nota SDD:** Cloudflared es Fundación porque sin él, el ecosistema es invisible desde internet. Pero técnicamente es el servicio más simple del stack: un binario, un token, una conexión saliente. Si falla, el diagnóstico es casi siempre: ¿internet está arriba? ¿el token es válido?
