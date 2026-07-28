# 🏗️ Bildung Docker — Arquitectura Objetivo v1.0

**Estado:** Documento vivo | **Versión:** 1.0 | **Basado en:** Clasificación por Capas de Dependencia

---

## 1. Filosofía

La arquitectura de Bildung Docker se rige por tres principios:

| Principio | Significado |
|---|---|
| **Una capa, un compose** | Cada capa de dependencia tiene su propio `docker-compose.yml`. Los servicios dentro de una capa comparten el mismo nivel de criticidad. |
| **Extends, no duplicación** | El compose principal (`./docker-compose.yml`) solo declara `extends` y `depends_on`. La definición real vive en la capa. |
| **Datos junto al servicio** | Cada capa tiene su propio directorio `data/`. Si un servicio usa named volumes, se declaran en el compose principal. |

---

## 2. Árbol del proyecto

```
docker/
├── .env                          # Variables de entorno por capa
├── docker-compose.yml            # Punto de entrada: extiende las 6 capas
│
├── docs/
│   ├── architecture.md           # Este documento
│   ├── services.md               # Clasificacion y distribucion de los servicios
│   ├── dependency_layers.md      # Como se clasifica por capas de dependenia
│   ├── specs/                    # Contratos de infraestructura (23 specs)
│   │   ├── postgres-spec.md
│   │   ├── redis-spec.md
│   │   ├── traefik-spec.md
│   │   ├── cloudflared-spec.md
│   │   ├── n8n-spec.md
│   │   ├── open-webui-spec.md
│   │   ├── open-terminal-spec.md
│   │   ├── nocodb-spec.md
│   │   ├── watchtower-spec.md
│   │   ├── minio-spec.md
│   │   ├── flowise-spec.md
│   │   ├── chrome-spec.md
│   │   ├── pgadmin-spec.md
│   │   ├── phpadmin-spec.md
│   │   ├── pgbackup-spec.md
│   │   ├── vaultwarden-spec.md
│   │   ├── dozzle-spec.md
│   │   ├── uptime-kuma-spec.md
│   │   ├── mariadb-spec.md
│   │   ├── wordpress-spec.md
│   │   ├── arcane-spec.md
│   │   └── resume-spec.md
│   └── recipes/                  # Procedimientos operativos (2 recetas)
│       ├── arcane
│       │   └── arcane-recipe.md
│       ├── chrome
│       │   └── chrome-recipe.md
│       ├── cloudflared
│       │   └── cloudflared-recipe.md
│       ├── dozzle
│       │   └── dozzle-recipe.md
│       ├── flowise
│       │   └── flowise-recipe.md
│       ├── mariadb
│       │   └── mariadb-recipe.md
│       ├── minio
│       │   └── minio-recipe.md
│       ├── n8n
│       │   └── n8n-recipe.md
│       ├── nocodb
│       │   └── nocodb-recipe.md
│       ├── open-terminal
│       │   └── open-terminal-recipe.md
│       ├── open-webui
│       │   └── open-webui-recipe.md
│       ├── pgadmin
│       │   └── pgadmin-recipe.md
│       ├── pgbackup
│       │   └── pgbackup-recipe.md
│       ├── phpadmin
│       │   └── phpadmin-recipe.md
│       ├── postgres
│       │   └── postgres-recipe.md
│       ├── redis
│       │   └── redis-recipe.md
│       ├── resume
│       │   └── resume-recipe.md
│       ├── traefik
│       │   └── traefik-recipe.md
│       ├── uptime-kuma
│       │   └── uptime-kuma-recipe.md
│       ├── vaultwarden
│       │   └── vaultwarden-recipe.md
│       ├── watchtower
│       │   └── watchtower-recipe.md
│       └── wordpress
│           └── wordpress-recipe.md
│
└── services/
    ├── foundation/               🔴 Capa 1
    │   ├── docker-compose.yml
    │   └── data/
    │       ├── postgres/
    │       ├── redis/
    │       ├── traefik/
    │       │   └── dynamic/
    │       │       └── cloudflare.yml
    │       └── cloudflared/
    │
    ├── core/                     🟠 Capa 2
    │   ├── docker-compose.yml
    │   └── data/
    │       ├── open-webui/
    │       └── open-terminal/
    │
    ├── extencion/                🟡 Capa 3
    │   ├── docker-compose.yml
    │   └── data/
    │       ├── minio/
    │       └── flowise/
    │
    ├── tool/                     🟢 Capa 4
    │   ├── docker-compose.yml
    │   └── data/
    │       ├── pgadmin/
    │       ├── backups/          (daily, weekly, monthly, last)
    │       └── vaultwarden/
    │
    ├── observability/            ⚪ Capa 5
    │   ├── docker-compose.yml
    │   └── data/
    │       └── uptime-kuma/
    │
    └── experiment/               🟣 Capa 6
        ├── docker-compose.yml
        └── data/
            └── arcane/
                └── projects/
```

---

## 3. Modelo de capas

Cada capa responde a una pregunta. El orden es intencional: las capas superiores no pueden iniciar sin que las inferiores estén listas.

| # | Capa | Pregunta | Si cae… |
|---|---|---|---|
| 🔴 | **Fundación** | ¿Sin esto funciona algo más? | Colapsa el ecosistema |
| 🟠 | **Core** | ¿Esto define lo que Bildung *hace*? | El propósito central se degrada |
| 🟡 | **Extensión** | ¿Esto potencia pero no define? | Funcionalidades específicas desaparecen |
| 🟢 | **Herramienta** | ¿Es un atajo o una ventana? | Solo pierdes conveniencia |
| ⚪ | **Observabilidad** | ¿Esto produce información o valor? | Operas ciego pero funcional |
| 🟣 | **Experimento** | ¿Si desaparece, alguien lo nota? | El ecosistema ni se entera |

> 📖 La clasificación completa con justificación por servicio está en `../documentacion/Clasificación por Capas de Dependencia.md`.

---

## 4. Servicios por capa

### 🔴 Fundación (4 servicios)

| # | Servicio | Imagen | Rol |
|---|---|---|---|
| 1 | **traefik** | `traefik:latest` | Reverse proxy. Punto único de entrada HTTP |
| 2 | **cloudflared** | `cloudflare/cloudflared:latest` | Túnel seguro al edge de Cloudflare |
| 3 | **postgres** | `pgvector/pgvector:pg18` | Base de datos relacional + vectorial |
| 4 | **redis** | `redis:7-alpine` | Caché y colas de trabajos |

### 🟠 Core (5 servicios)

| # | Servicio | Imagen | Rol |
|---|---|---|---|
| 5 | **n8n** | `n8nio/n8n:latest` | Motor de automatización. El "SO" del ecosistema |
| 6 | **open-webui** | `open-webui/open-webui:latest` | Interfaz unificada de IA |
| 7 | **open-terminal** | `open-webui/open-terminal` | Terminal web del host. Puerta de administración remota |
| 8 | **nocodb** | `nocodb/nocodb:latest` | Base de datos visual tipo Airtable |
| 9 | **watchtower** | `containrrr/watchtower:latest` | Actualizador automático de imágenes |

### 🟡 Extensión (4 servicios)

| # | Servicio | Imagen | Rol |
|---|---|---|---|
| 10 | **minio** | `minio/minio:latest` | Almacenamiento S3-compatible |
| 11 | **flowise** | `flowiseai/flowise:latest` | Constructor visual de agentes IA |
| 12 | **flowise-worker** | `flowiseai/flowise-worker:latest` | Ejecutor de tareas de flowise |
| 13 | **chrome** | `browserless/chromium:v2.18.0` | Navegador headless para PDFs |

### 🟢 Herramienta (4 servicios)

| # | Servicio | Imagen | Rol |
|---|---|---|---|
| 14 | **pgadmin** | `dpage/pgadmin4:latest` | Administrador gráfico de postgres |
| 15 | **phpadmin** | `phpmyadmin` | Administrador de mariadb |
| 16 | **pgbackup** | `prodrigestivill/postgres-backup-local` | Backups automáticos de postgres |
| 17 | **vaultwarden** | `vaultwarden/server:latest` | Gestor de contraseñas |

### ⚪ Observabilidad (2 servicios)

| # | Servicio | Imagen | Rol |
|---|---|---|---|
| 18 | **dozzle** | `amir20/dozzle:latest` | Visor de logs en tiempo real |
| 19 | **uptime-kuma** | `louislam/uptime-kuma:1` | Monitor de disponibilidad |

### 🟣 Experimento (4 servicios)

| # | Servicio | Imagen | Rol |
|---|---|---|---|
| 20 | **mariadb** | `tobi312/rpi-mariadb:10.5-debian` | Base de datos para WordPress |
| 21 | **wordpress** | `wordpress:latest` | CMS / blog personal |
| 22 | **arcane** | `getarcaneapp/arcane:latest` | Panel de servidores de juego |
| 23 | **resume** | `amruthpillai/reactive-resume:latest` | Constructor de CV |

---

## 5. Grafo de dependencias

```text
                        🔴 FUNDACIÓN
        ┌───────────────┼───────────────┐
    traefik         cloudflared      postgres ──────── redis
        │                │               │                │
        └────────────────┴───────────────┴────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    🟠 CORE           🟡 EXTENSIÓN     🟣 EXPERIMENTO
    ─────────         ────────────     ─────────────
    n8n               minio            mariadb ─── wordpress
    ├─postgres        │                │
    └─redis           flowise          arcane ─── postgres
                      ├─postgres       │
    open-webui        ├─redis          resume
    │                 └─flowise        ├─postgres
    open-terminal     │                ├─redis
                      flowise-worker   ├─minio
    nocodb            ├─postgres       └─chrome
    ├─postgres        ├─redis
    └─redis           └─flowise       ─────────────
                      │
    watchtower        chrome
    (docker.sock)     (sin deps)

    ─────────         ────────────     ─────────────
    🟢 HERRAMIENTA    ⚪ OBSERVAB.     (nombrados)
    ─────────         ────────────     n8n_data
    pgadmin           dozzle           nocodb-data
    (sin deps)        (docker.sock)    mariadb_data
                                       wordp_data
    phpadmin          uptime-kuma      arcane-data
    (sin deps)        (sin deps)
    │
    pgbackup
    └─postgres
    │
    vaultwarden
    └─postgres
```

---

## 6. Red

Una sola red Docker bridge para todo el ecosistema:

```yaml
networks:
  services:
    name: services
    driver: bridge
```

| Propiedad | Valor |
|---|---|
| Nombre | `services` |
| Driver | `bridge` |
| Resolución DNS | Automática por container_name |
| Aislamiento | Solo los servicios en esta red se ven entre sí |

---

## 7. Volúmenes

### Bind mounts (por capa)

| Capa | Path | Servicios |
|---|---|---|
| foundation | Named volume `postgres_data` | postgres |
| foundation | Named volume `redis_data` | redis |
| foundation | `./data/traefik/dynamic/` | traefik |
| core | `./data/open-webui/` | open-webui |
| core | `./data/open-terminal/` | open-terminal |
| extencion | `./data/minio/` | minio |
| extencion | `./data/flowise/` | flowise, flowise-worker |
| tool | `./data/pgadmin/` | pgadmin |
| tool | `./data/backups/` | pgbackup |
| tool | `./data/vaultwarden/` | vaultwarden |
| observability | `./data/uptime-kuma/` | uptime-kuma |
| experiment | `./data/arcane/projects/` | arcane |

### Named volumes (declarados en compose principal)

| Volumen | Usado por | Razón |
|---|---|---|
| `n8n_data` | n8n | Heredado del compose anterior. Sin migrar a bind mount |
| `nocodb-data` | nocodb | Heredado. Sin migrar |
| `mariadb_data` | mariadb | Named volume estándar para bases de datos |
| `wordp_data` | wordpress | Plugins, temas, uploads |
| `arcane-data` | arcane | Datos de aplicación |

---

## 8. Variables de entorno

Un solo archivo `.env` organizado en secciones por capa:

```text
# ====== CONFIG ======
TZ, DOMINIO, PROJECT_PATH

# ====== 🔴 FUNDACIÓN ======
POSTGRES_*, REDIS_*, CLOUDFLARE_TUNNEL_TOKEN, TRAEFIK_AUTH

# ====== 🟠 CORE ======
N8N_*, JINA_API_KEY, OPEN_TERMINAL_*, NOCO_*, WATCHTOWER_*

# ====== 🟡 EXTENSIÓN ======
MINIO_*, FLOWISE_*, CHROME_*

# ====== 🟢 HERRAMIENTA ======
PGADMIN_*, MARIA_*, BACKUP_*, VAULTWARDEN_*

# ====== ⚪ OBSERVABILIDAD ======
(usa TZ únicamente)

# ====== 🟣 EXPERIMENTO ======
ARCANE_*, RR_*
```

> ⚠️ Las variables marcadas con `*` son todas las que pertenecen a ese prefijo. Ver `.env` para la lista completa.

---

## 9. Spec vs Receta

| Documento | Responde | Cuándo se lee | Cuándo se actualiza |
|---|---|---|---|
| **Spec** | ¿Qué contrato tiene este servicio con la infraestructura? | Al diseñar, al diagnosticar dependencias | Cuando cambia la arquitectura |
| **Receta** | ¿Cómo opero este servicio día a día? | Al desplegar, al hacer backup, al debuggear | Cuando cambia el procedimiento |

Ambos documentos existen para cada servicio. Las specs están completas (23/23). Las recetas están en progreso (2 completas).

---

## 10. Cómo usar esta arquitectura

### Levantar todo

```bash
cd docker/
docker compose up -d
```

### Levantar solo una capa

```bash
docker compose up -d traefik cloudflared postgres redis   # 🔴 Fundación
docker compose up -d n8n open-webui nocodb watchtower     # 🟠 Core
```

### Ver el estado de una capa

```bash
docker compose ps --filter "status=running" | grep -E "traefik|cloudflared|postgres|redis"
```

### Recrear un servicio específico

```bash
docker compose up -d --force-recreate n8n
```

### Ver logs de un servicio

```bash
# Opción A: Dozzle (UI web)
open https://logs.hb-system.info

# Opción B: CLI
docker compose logs -f --tail 100 n8n
```

### Backup de postgres

```bash
# Automático: pgbackup corre @daily
ls -la services/tool/data/backups/last/

# Manual:
docker compose exec pgbackup sh -c "pg_dump -U postgres -h postgres n8n | gzip > /backups/manual-n8n.sql.gz"
```

---

## 11. Principios de evolución

| Principio | Regla |
|---|---|
| **Agregar servicio** | Crear spec → agregar a la capa correspondiente → `extends` en compose principal → test |
| **Mover de capa** | Actualizar spec (nueva capa) → cambiar el `extends` → actualizar clasificación |
| **Archivar servicio** | Comentar en el compose de capa → mover spec a `docs/specs/archived/` |
| **Cambiar imagen** | Actualizar spec → test → `docker compose up -d --force-recreate <servicio>` |
| **Nueva variable** | Agregar en `.env` en su sección de capa → referenciar en el compose → documentar en spec |

---

## 12. Diferencias con la arquitectura anterior

| Aspecto | Antes (semilla) | Ahora (Migracion) |
|---|---|---|
| Composes | 1 monolito de ~1100 líneas + servicios sueltos en `./services/` | 1 principal + 6 de capa, cada uno <80 líneas |
| Red | `hbs-net` | `services` |
| Volúmenes | Mezcla de named/bind inconsistente | Bind mounts por defecto, named volumes solo donde se heredaron |
| Variables | `.env` plano sin organización | `.env` seccionado por capa |
| Documentación | Ninguna | 23 specs + 2 recetas + este documento |
| Dependencias visibles | `depends_on` disperso en un solo archivo | `depends_on` en compose principal, árbol completo documentado |
| Servicios activos | ~40 declarados, ~19 realmente usados | 23 activos, el resto archivados en `semilla/` |

---

📌 **Este documento es la fuente de verdad de la arquitectura Docker de Bildung.** Cualquier cambio en la estructura de composes, capas o dependencias debe reflejarse aquí.
