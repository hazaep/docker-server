# 📘 Open WebUI — Service Specification v1.0

**Capa:** 🟠 Core | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Definir el contrato de infraestructura del servicio Open WebUI dentro del ecosistema Bildung: qué espera del entorno, qué expone y bajo qué condiciones se considera operativo.

---

## 2. Definición del Servicio

> Interfaz web unificada que expone modelos de lenguaje (LLMs) a usuarios humanos y agentes del ecosistema. Es la **capa de presentación de las capacidades cognitivas** de Bildung.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Un solo punto de acceso** | Toda interacción con LLMs pasa por esta interfaz |
| **Backend agnóstico** | No sabe qué motor de inferencia responde; solo conoce su API |
| **Reemplazabilidad** | Si mañana se migra a otra UI, los clientes no deben notarlo |
| **Estado externo** | Los datos sobreviven al contenedor (volumen persistente) |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada (lo que el servicio exige)

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 512 MB, recomendado 1 GB | Sí |
| **Disco** | Mínimo 2 GB para datos (SQLite + uploads) | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |
| **Reverse Proxy** | Traefik con entrada `web` en puerto 80 | Sí |
| **DNS** | Subdominio `webui.${DOMINIO}` apuntando al host | Sí |
| **Ollama (externo)** | Host accesible con modelos de lenguaje | No — sin él, la UI carga pero no responde |

### 🔹 Salida (lo que el servicio expone)

| Recurso | Valor |
|---|---|
| **Puerto interno** | `8080` (HTTP) |
| **Puerto host** | `3000` |
| **Endpoint web** | `http://webui.${DOMINIO}` |
| **API interna** | `http://open-webui:8080/api/v1/...` (para otros servicios en la misma red) |

### 🔹 Dependencias en el ecosistema

```text
open-webui
  ├── traefik (reverse proxy) — obligatorio
  ├── ollama (inferencia) — externo, obligatorio para funcionar
  └── postgres (opcional) — si se migra desde SQLite
```

---

## 4. Volumen y Persistencia

| Path interno | Path host | Datos | Estrategia de backup |
|---|---|---|---|
| `/app/backend/data` | `./data/open-webui/` | Usuarios, chats, configs, API keys, archivos subidos | `tar.gz` periódico (diario) |

### 🔹 Regla de persistencia

> Si el volumen se pierde, se pierde el historial de chats y las credenciales de usuarios. El servicio es reemplazable pero sus datos no.

---

## 5. Variables de Entorno (Contrato)

| Variable | Tipo | Obligatorio | Default | Descripción |
|---|---|---|---|---|
| `TZ` | string | No | `UTC` | Zona horaria |
| `JINA_API_KEY` | string | No | — | API key para embeddings Jina AI |
| `OLLAMA_BASE_URL` | string | No | — | URL del host de Ollama (si no está en localhost) |
| `DATABASE_URL` | string | No | — | String de conexión Postgres (si se migra de SQLite) |
| `WEBUI_SECRET_KEY` | string | No | — | Clave de sesión (producción multi-instancia) |

### 🔹 Regla

> Las variables sin default deben definirse en `.env` o el servicio usará valores internos. Si una variable requerida por una feature (ej. `JINA_API_KEY` para búsqueda semántica) no está presente, esa feature simplemente no se habilita.

---

## 6. Criterios de Aceptación (Definition of Done)

| # | Criterio | Cómo verificarlo |
|---|---|---|
| **CA-1** | El contenedor inicia sin errores | `docker compose ps open-webui` → State: Up |
| **CA-2** | La UI responde HTTP 200 | `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000` → `200` |
| **CA-3** | Traefik enruta correctamente | Navegar a `http://webui.${DOMINIO}` → carga la UI |
| **CA-4** | Los datos sobreviven a un reinicio | Crear un chat → `docker compose restart open-webui` → el chat sigue ahí |
| **CA-5** | Responde a un modelo | Enviar mensaje en la UI → el modelo responde |
| **CA-6** | Watchtower la monitorea | Label `com.centurylinklabs.watchtower.enable=true` presente |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 (Raspberry Pi 4/5) |
| **Sistema operativo base** | Linux (Debian/Ubuntu ARM) |
| **Motor de contenedor** | Docker 24+ |
| **No soportado** | x86, Kubernetes (por ahora), múltiples instancias |
| **Upload máximo** | 100 MB (configurable en UI) |
| **Usuarios concurrentes** | ~5-10 en Raspberry Pi 4 |

---

## 8. Migración y Evolución

### 🔹 Versionado

| Cambio | Acción |
|---|---|
| Agregar variable de entorno | ✅ Permitido (compatible hacia atrás) |
| Cambiar puerto interno | ⚠️ Nueva versión de spec + actualizar Traefik labels |
| Cambiar imagen base | ⚠️ Test de humo obligatorio |
| Migrar SQLite → Postgres | ⚠️ Requiere migración de datos + actualizar spec |

### 🔹 Plan de migración desde librechat

1. Levantar open-webui en paralelo con librechat
2. Validar que todos los modelos son accesibles
3. Migrar usuarios manualmente (sin automatización por ahora)
4. Archivar librechat tras 30 días sin incidencias

---

## 9. Diagrama de Interacción

```text
[Usuario] ── navegador ──→ webui.hb-system.info
                                │
                           [Traefik :80]
                                │
                           [open-webui :8080]
                                │
                           [Ollama :11434]  ← host externo o remoto
                                │
                           [Modelos de lenguaje]
```

---

📌 **Nota SDD:** Esta spec es el contrato entre open-webui y la infraestructura Bildung. Cualquier cambio en puertos, volúmenes o dependencias debe reflejarse aquí antes de aplicarse. La receta (`open-webui-recipe.md`) contiene los procedimientos operativos; esta spec contiene las reglas que los rigen.
