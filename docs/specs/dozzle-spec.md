# 📘 Dozzle — Service Specification v1.0

**Capa:** ⚪ Observabilidad | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Visor de logs en tiempo real para contenedores Docker. **Es la ventana de diagnóstico del ecosistema.** Si cae, `docker compose logs` sigue funcionando. Dozzle solo lo hace más cómodo.

---

## 2. Definición del Servicio

> Dozzle conectado al socket de Docker. UI web con filtros por contenedor, búsqueda full-text, y actualización en tiempo real vía Server-Sent Events. Sin base de datos, sin estado.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Read-only** | Solo lee logs. No puede modificar contenedores |
| **Efímero** | Sin persistencia. Si se recrea, no se pierde nada |
| **Sin auth** | No tiene login. Cualquiera en la red puede ver los logs |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 64 MB | Sí |
| **Docker socket** | `/var/run/docker.sock` (read-only implícito por cómo Dozzle lo usa) | **Sí** |
| **Red** | Acceso a la red `services` (bridge) | Sí |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://logs.${DOMINIO}` |
| **Puerto interno** | `8080` |

### 🔹 Dependencias

```text
dozzle
  └── docker.sock (host) — sin esto no ve nada
```

---

## 4. Sin volúmenes persistentes

Dozzle es completamente efímero. Los logs los lee en tiempo real de Docker; no los almacena.

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `TZ` | No | Zona horaria para timestamps de logs |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps dozzle` → Up |
| **CA-2** | UI accesible | `curl -s https://logs.${DOMINIO}` → HTML |
| **CA-3** | Lista contenedores | Abrir UI → todos los contenedores visibles |
| **CA-4** | Muestra logs | Seleccionar un contenedor → logs en tiempo real |

---

## 7. Riesgos

| Riesgo | Mitigación |
|---|---|
| Acceso sin auth a logs | Los logs pueden contener variables de entorno (con contraseñas). Si esto es inaceptable, poner Dozzle detrás de Traefik basic auth |
| Docker socket expuesto | Dozzle solo lee, pero el socket da acceso root al host. No exponer Dozzle sin Traefik |

---

📌 **Nota SDD:** Dozzle es el servicio más simple del ecosistema. Un binario, un socket, una UI. Si quieres debuggear algo rápido, es la primera herramienta que abres. Si no está, `docker compose logs <servicio>` hace lo mismo con más fricción.
