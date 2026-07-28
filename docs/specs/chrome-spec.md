# 📘 Chrome (Browserless) — Service Specification v1.0

**Capa:** 🟡 Extensión | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Navegador headless para generación de PDFs, screenshots y renderizado server-side. **Es un servicio de soporte: no tiene UI propia, no recibe tráfico de usuario.** Solo lo consumen otras aplicaciones del ecosistema.

---

## 2. Definición del Servicio

> Browserless Chromium v2.18.0 (versión fija por compatibilidad ARM64). Sin Traefik labels, sin exposición externa. Solo accesible internamente vía `ws://chrome:3000`.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Sin exposición externa** | No tiene label de Traefik. Solo red Docker interna |
| **Versión fija** | `v2.18.0` — versiones más recientes tienen problemas en ARM64 |
| **Health check activo** | `PRE_REQUEST_HEALTH_CHECK=true` — verifica que está listo antes de aceptar trabajos |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB (picos de 512 MB al renderizar) | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **WebSocket** | `ws://chrome:3000` |
| **HTTP API** | `http://chrome:3000` |

### 🔹 Dependencias

```text
chrome
  └── (ninguna — es independiente)
```

### 🔹 ¿Quién depende de Chrome?

```text
chrome
  └── resume (🟣 Experimento) — generación de PDFs de CV
```

---

## 4. Sin volúmenes persistentes

Chrome es completamente efímero. Cada trabajo de renderizado crea un contexto nuevo. No almacena nada entre ejecuciones.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Default | Descripción |
|---|---|---|---|
| `CHROME_TIMEOUT` | No | `30000` | Timeout en ms para trabajos de renderizado |
| `CHROME_CONCURRENT` | No | `10` | Máximo de trabajos concurrentes |
| `CHROME_TOKEN` | No | — | Token de autenticación para la API |
| `CHROME_EXIT_FAILURE` | No | `false` | Salir si el health check falla |
| `CHROME_HEALTH_CHECK` | No | `true` | Verificar que el navegador está listo antes de aceptar trabajos |
| `TZ` | No | `UTC` | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps chrome` → Up |
| **CA-2** | API responde | `curl -s http://chrome:3000` → respuesta HTML/JSON |
| **CA-3** | Health check pasa | Logs muestran `Chrome is ready` |
| **CA-4** | Acepta trabajos | `curl -X POST http://chrome:3000/pdf -H "Content-Type: application/json" -d '{"url":"http://example.com"}'` → PDF |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 |
| **Versión congelada** | `v2.18.0`. No actualizar sin pruebas exhaustivas |
| **Sin exposición externa** | No accesible desde internet |
| **Consumo de RAM** | Cada página abierta consume ~50-100 MB adicionales |

---

📌 **Nota SDD:** Chrome es el servicio más "invisible" del ecosistema. Si cae, el único síntoma es que `resume` deja de exportar PDFs. Está en Extensión (no en Herramienta) porque su caída rompe una funcionalidad concreta de una app, no solo la conveniencia.
