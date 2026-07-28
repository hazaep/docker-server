# 📘 Open Terminal — Service Specification v1.0

**Capa:** 🟠 Core | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Definir el contrato de infraestructura del servicio Open Terminal dentro del ecosistema Bildung: qué espera del entorno, cómo se integra, y bajo qué condiciones se considera operativo.

---

## 2. Definición del Servicio

> Terminal web integrada como plugin de Open WebUI. Proporciona acceso shell directo al filesystem del contenedor, que a su vez puede montar volúmenes del host. **Es la puerta de administración remota del ecosistema Bildung.**

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Integración, no independencia** | No tiene su propia UI. Se accede exclusivamente desde Open WebUI |
| **Puerta, no casa** | Permite administrar, no reemplaza SSH ni docker exec |
| **Efímero con persistencia selectiva** | El contenedor se recrea, el `/home/user` persiste |
| **Promovido por estrategia** | Técnicamente es Extensión; subió a Core porque sin terminal remota, un Raspberry Pi headless es inaccesible |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada (lo que el servicio exige)

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB | Sí |
| **Disco** | Mínimo 500 MB para home del usuario | Sí |
| **Red** | Acceso a la red `services` (bridge) | Sí |
| **Open WebUI** | Instancia corriendo y accesible | **Sí** — es su única interfaz |
| **API Key** | `OPEN_TERMINAL_API_KEY` configurada en ambos servicios | **Sí** — sin esto no autentica |

### 🔹 Salida (lo que el servicio expone)

| Recurso | Valor |
|---|---|
| **Puerto interno** | Ninguno expuesto directamente |
| **Acceso** | Exclusivamente vía Open WebUI → plugin de terminal |
| **Shell disponible** | `/bin/bash` dentro del contenedor |
| **Home persistente** | `/home/user` → `./data/open-terminal/` |

### 🔹 Dependencias en el ecosistema

```text
open-terminal
  └── open-webui (interfaz contenedora) — obligatorio, sin él no existe
```

> ⚠️ Es la única dependencia directa. Pero indirectamente, si Open WebUI depende de Traefik y Ollama, Open Terminal también depende de ellos para ser accesible y útil.

---

## 4. Volumen y Persistencia

| Path interno | Path host | Datos | Estrategia de backup |
|---|---|---|---|
| `/home/user` | `./data/open-terminal/` | Scripts, configs de shell, historial, herramientas instaladas manualmente | `tar.gz` semanal |

### 🔹 Regla de persistencia

> Los paquetes instalados vía `OPEN_TERMINAL_PACKAGES` y `OPEN_TERMINAL_PIP_PACKAGES` **no** son persistentes: se reinstalan en cada arranque del contenedor. Para herramientas que deban sobrevivir, instalarlas manualmente dentro del volumen persistente.

---

## 5. Variables de Entorno (Contrato)

| Variable | Tipo | Obligatorio | Default | Descripción |
|---|---|---|---|---|
| `OPEN_TERMINAL_API_KEY` | string | **Sí** | — | API key compartida con Open WebUI para autenticación |
| `OPEN_TERMINAL_PACKAGES` | string | No | — | Paquetes APT separados por espacio (ej. `syncthing htop`) |
| `OPEN_TERMINAL_PIP_PACKAGES` | string | No | — | Paquetes PIP separados por espacio (ej. `requests numpy`) |
| `JINA_API_KEY` | string | No | — | Compartida con Open WebUI |
| `TZ` | string | No | `UTC` | Zona horaria |

### 🔹 Regla de paquetes

> Si un paquete falla al instalarse, el contenedor **aún inicia**. El error aparece en logs pero no bloquea el servicio. Esto evita que una dependencia rota deje sin terminal al ecosistema.

---

## 6. Criterios de Aceptación (Definition of Done)

| # | Criterio | Cómo verificarlo |
|---|---|---|
| **CA-1** | El contenedor inicia sin errores | `docker compose ps open-terminal` → State: Up |
| **CA-2** | Open WebUI detecta el plugin | En Open WebUI, la terminal aparece como opción disponible |
| **CA-3** | La terminal acepta comandos | Abrir terminal → ejecutar `whoami` → responde `user` |
| **CA-4** | Los paquetes APT se instalan | `which syncthing` (si está en `OPEN_TERMINAL_PACKAGES`) → retorna path |
| **CA-5** | Los paquetes PIP se instalan | `python3 -c "import requests"` → sin error |
| **CA-6** | El home sobrevive a un reinicio | Crear archivo en `/home/user/test` → `restart` → el archivo sigue |
| **CA-7** | Sin API key, no conecta | Si `OPEN_TERMINAL_API_KEY` es inválida, la terminal no se muestra en Open WebUI |

---

## 7. Límites y Restricciones

| Restricción | Valor |
|---|---|
| **Arquitectura** | ARM64 (Raspberry Pi 4/5) |
| **Sistema operativo base** | Linux (Debian/Ubuntu ARM) |
| **Motor de contenedor** | Docker 24+ |
| **Acceso** | Solo por Open WebUI. No tiene puerto expuesto, no tiene Traefik label |
| **Seguridad** | La API key es la única barrera. Si se filtra, cualquiera con acceso a Open WebUI tiene shell |
| **Aislamiento** | El contenedor **no** tiene acceso al socket de Docker del host. No puede manipular otros contenedores |

---

## 8. Riesgos y Mitigaciones

| Riesgo | Impacto | Mitigación |
|---|---|---|
| API key filtrada | 🔴 Acceso shell no autorizado | Rotar key, usar valores largos (64+ caracteres) |
| Paquete APT no disponible | 🟡 El paquete no se instala, el resto sigue | Revisar logs, corregir nombre, recrear contenedor |
| Open WebUI caído | 🔴 Terminal inaccesible | La terminal no puede hacer nada al respecto. Restaurar Open WebUI primero |
| `depends_on` no espera ready | 🟡 Open WebUI arrancó pero aún no carga el plugin | `docker compose restart open-terminal` si falla la primera vez |

---

## 9. Diagrama de Interacción

```text
[Usuario] ── navegador ──→ webui.hb-system.info
                                │
                           [Traefik :80]
                                │
                           [open-webui :8080]
                                │
                           [Plugin: Open Terminal]
                                │
                           [open-terminal :shell]
                                │
                           [./data/open-terminal/]  ← home persistente
```

---

## 10. Justificación de Capa (🟠 Core)

Por dependencias técnicas, Open Terminal pertenece a 🟡 Extensión: si cae, el ecosistema funciona. Pero se promueve a 🟠 Core por:

1. **Raspberry Pi es headless.** Sin terminal web, no hay acceso al host sin monitor+teclado físico o SSH desde otra máquina en la misma red local
2. **Es el plan B.** Si Traefik tiene un problema de configuración que rompe el enrutamiento, la terminal (por estar dentro de Open WebUI) puede ser la única forma de diagnosticar
3. **Bildung depende de ella para mantenerse.** No es un lujo; es la llave de administración

> Esta promoción es intencional y documentada. Si en el futuro se implementa SSH remoto robusto, Open Terminal puede degradarse a 🟡 Extensión.

---

📌 **Nota SDD:** Esta spec define el contrato entre Open Terminal y la infraestructura Bildung. No es una spec de API (el servicio no expone endpoints), sino una spec de **servicio infraestructural**: qué garantiza, qué exige, y cómo saber si está bien. La receta (`open-terminal-recipe.md`) contiene los procedimientos operativos.
