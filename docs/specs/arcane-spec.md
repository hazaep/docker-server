# 📘 Arcane — Service Specification v1.0

**Capa:** 🟣 Experimento | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

Panel de gestión de servidores de juego (Minecraft, etc.). **Es un hobby con esteroides.** Si cae, Bildung ni se entera. Si crece, podría convertirse en un producto independiente.

---

## 2. Definición del Servicio

> Arcane con PostgreSQL como backend y Docker socket para gestionar contenedores de juegos. Expuesto vía Traefik. Los proyectos (mundos, configs) se almacenan en bind mount.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Aislado del ecosistema** | Su razón de existir no tiene nada que ver con Bildung |
| **Docker socket compartido** | Arcane crea/maneja sus propios contenedores. No toca los de Bildung |
| **Postgres compartido** | Usa la misma instancia de Postgres pero su propia base de datos |

---

## 3. Contrato de Infraestructura

### 🔹 Entrada

| Recurso | Especificación | Obligatorio |
|---|---|---|
| **CPU** | ARM64 compatible | Sí |
| **RAM** | Mínimo 256 MB | Sí |
| **Disco** | `arcane-data` (named volume) + `./data/arcane/projects` (bind mount) | Sí |
| **Docker socket** | `/var/run/docker.sock` | **Sí** — para crear/manejar contenedores de juegos |

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://arcane.${DOMINIO}` |
| **Puerto interno** | `3552` |

### 🔹 Dependencias

```text
arcane
  └── postgres (🔴 Fundación) — backend de base de datos
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre/Path | Path interno | Contenido |
|---|---|---|---|
| Named volume | `arcane-data` | `/app/data` | Datos de la aplicación |
| Bind mount | `./data/arcane/projects` | `/app/data/projects` | Mundos, configs de servidores |

---

## 5. Variables de Entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `ARCANE_PG_USER` | **Sí** | Usuario de postgres |
| `ARCANE_PG_PASSWORD` | **Sí** | Contraseña de postgres |
| `ARCANE_PG_DB` | **Sí** | Base de datos |
| `ARCANE_ENCRYPTION_KEY` | **Sí** | Clave de encriptación interna |
| `ARCANE_JWT_SECRET` | **Sí** | Secreto JWT para autenticación |
| `DOMINIO` | **Sí** | Dominio base |
| `TZ` | No | Zona horaria |

---

## 6. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps arcane` → Up |
| **CA-2** | UI accesible | `curl -s https://arcane.${DOMINIO}` → login/setup |
| **CA-3** | Conexión a postgres | Login funciona → panel carga sin errores |
| **CA-4** | Ve proyectos existentes | Proyectos en `./data/arcane/projects` son visibles en la UI |

---

## 7. Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Docker socket compartido | 🔴 Arcane puede ver/modificar contenedores de Bildung | Idealmente usar Docker contexts separados. Por ahora, confianza en el scope de Arcane |
| Juegos consumen RAM | 🟡 Competencia con servicios de Bildung | Limitar recursos desde la UI de Arcane o con constraints Docker |

---

📌 **Nota SDD:** Arcane es el servicio más "independiente" del ecosistema. Si mañana migra a su propia Raspberry Pi, el único cambio es el `DATABASE_URL`. Está en Experimento porque es exploración pura — si se vuelve parte del core de Bildung, reevaluar.
