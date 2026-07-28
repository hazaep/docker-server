
# 📋 Lista completa

## 🔴 Capa 1 — Fundación
_Si cae, colapsa el ecosistema._

| # | Servicio | ¿Por qué está aquí? |
| --- | --- | --- |
| 1 | **traefik** | Único punto de entrada. Sin él, ninguna app es accesible vía web. |
| 2 | **cloudflared** | Túnel Cloudflare. Sin él, el dominio no resuelve al servidor. El ecosistema queda invisible desde internet. |
| 3 | **postgres** | Base de datos principal. Lo usan: n8n, flowise, nocodb, maybe, memos, grafana, rag-api, arcane… Si cae, la mayoría de apps pierden estado. |
| 4 | **redis** | Caché y colas. Lo usan: n8n, flowise, evolution-api, nocodb, maybe, resume… Sin él, las apps funcionan degradadas o se niegan a arrancar. |

---

## 🟠 Capa 2 — Core
_Si cae, el propósito central de Bildung se degrada._

| # | Servicio | ¿Por qué está aquí? |
| --- | --- | --- |
| 5 | **n8n** | Corazón de la automatización. Sin él, los workflows que conectan servicios dejan de ejecutarse. Es el "sistema operativo" del ecosistema. |
| 6 | **open-webui** | Interfaz unificada de IA. Si cae, pierdes acceso a todos los modelos de lenguaje en un solo lugar. Es la cara visible de las capacidades cognitivas. |
| 7 | **open-terminal** | Terminal web de la Raspberry Pi. Técnicamente sería Extensión, pero se promueve a Core por valor estratégico: es la puerta de administración directa del host. Sin ella, la intervención manual requiere SSH físico. |
| 8 | **watchtower** | Actualizaciones automáticas de contenedores. Sin él las imágenes envejecen, pero el ecosistema sigue. **Está aquí porque es la única defensa automatizada contra vulnerabilidades.** Si cae, nada te avisa. Pero técnicamente no es fundacional — está en el borde entre 🟠 y 🟢. |
| 9 | **nocodb** | Base de datos tipo Airtable. Depende de postgres, algunas apps dependen de nocodb. |

---

## 🟡 Capa 3 — Extensión
_Si cae, funcionalidades específicas desaparecen — pero Bildung sigue siendo Bildung._

| #   | Servicio           | ¿Por qué está aquí?                                                                                            |
| --- | ------------------ | -------------------------------------------------------------------------------------------------------------- |
| 10  | **flowise**        | Constructor visual de agentes IA. Potencia la automatización con IA, pero n8n puede seguir operando sin él.    |
| 11  | **flowise-worker** | Ejecutor de tareas de flowise. Sin el worker, flowise no procesa. Misma capa que flowise.                      |
| 12  | **minio**          | Almacenamiento S3-compatible. Lo usa nocodb para guardar assets. Si cae, algunos workflows y apps se degradan. |
| 13  | **chrome**         | Navegador headless para que resume genere PDFs. Si cae, resume no exporta. Solo afecta a resume                |

---

## 🟢 Capa 4 — Herramienta
_Si cae, pierdes conveniencia. Nada crítico._

| #   | Servicio        | ¿Por qué está aquí?                                                                                                     |
| --- | --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 14  | **phpadmin**    | Administrador de mariadb. Ídem: conveniencia, no necesidad.                                                             |
| 15  | **pgbackup**    | Backups automáticos de postgres. Ídem: seguro de vida, no órgano vital.                                                 |
| 16  | **vaultwarden** | Gestor de contraseñas. App independiente. Crítico para ti como usuario, poco relevante para el ecosistema como sistema. |
| 17  | **pgadmin**     | Administrador gráfico de postgres. Si cae, sigues administrando postgres por CLI. Menos cómodo, pero funcional.         |

---

## ⚪ Capa 5 — Observabilidad
_Si cae, operas ciego. El ecosistema funciona pero no sabes si está sano._

| #   | Servicio        | ¿Por qué está aquí?                                                                        |
| --- | --------------- | ------------------------------------------------------------------------------------------ |
| 18  | **dozzle**      | Visor de logs en tiempo real. Si cae, usas `docker logs`. Menos bonito, misma información. |
| 19  | **uptime-kuma** | Monitor de disponibilidad. Te avisa si algo cae. No evita que nada caiga.                  |

---

## 🟣 Capa 6 — Experimento
_Si cae, el ecosistema ni se entera. Exploración, no infraestructura._

| #   | Servicio      | ¿Por qué está aquí?                                                                                                                        |
| --- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 20  | **mariadb**   | Base de datos exclusiva de wordpress. Si wordpress está en 🟣, mariadb también.                                                            |
| 21  | **arcane**    | Panel de gestión de servidores de juego (Minecraft, etc.). App independiente con su propio ecosistema. Interesante pero no toca al núcleo. |
| 22  | **wordpress** | CMS / blog. Depende de mariadb. Nada depende de él.                                                                                        |
| 23  | **resume**    | Constructor de CV. App independiente con sus propias dependencias (chrome, minio, postgres)                                                |

---

## 💤 Capa 7 — Archivado
_Comentado en los docker-compose. No corre. El diseño existe._

| #   | Servicio                | ¿Qué era?                                                                                                     |
| --- | ----------------------- | ------------------------------------------------------------------------------------------------------------- |
| 24  | **appsmith**            | Builder de apps internas tipo Retool                                                                          |
| 25  | **airflow-webserver**   | Orquestador de pipelines de datos                                                                             |
| 26  | **airflow-scheduler**   | Planificador de tareas de airflow                                                                             |
| 27  | **airflow-worker**      | Ejecutor de tareas de airflow                                                                                 |
| 28  | **airflow-init**        | Inicializador de base de datos de airflow                                                                     |
| 29  | **wallos**              | Gestión de suscripciones                                                                                      |
| 30  | **maybe**               | Finanzas personales                                                                                           |
| 31  | **wopi_server**         | Servidor Collabora para editar documentos en filestash                                                        |
| 32  | **memos**               | Notas tipo micro-blog                                                                                         |
| 33  | **prometheus**          | Recolector de métricas. Sin él, grafana no tiene datos. Pero ninguna app depende de prometheus para funcionar |
| 34  | **grafana**             | Dashboards de visualización. Depende de prometheus. Sin él, no ves métricas. Las apps siguen funcionando      |
| 35  | **node-exporter**       | Métricas del sistema operativo host. Solo alimenta a prometheus                                               |
| 36  | **cadvisor**            | Métricas de contenedores Docker. Solo alimenta a prometheus                                                   |
| 37  | **blackbox-exporter**   | Probes HTTP/TCP externos. Solo alimenta a prometheus                                                          |
| 38  | **mongo-express**       | Administrador de mongodb                                                                                      |
| 39  | **flame**               | Dashboard de inicio con marcadores                                                                            |
| 40  | **duplicati**           | Backups a servicios cloud. Grave a largo plazo, pero el ecosistema no se cae hoy                              |
| 41  | **filestash**           | Gestor de archivos web tipo Dropbox self-hosted                                                               |
| 42  | **evolution-api**       | Puente con WhatsApp                                                                                           |
| 43  | **rag-api**             | Motor de embeddings y búsqueda semántica de librechat                                                         |
| 44  | **meilisearch**         | Búsqueda full-text                                                                                            |
| 45  | **qdrant**              | Base de datos vectorial                                                                                       |
| 46  | **anythingllm**         | Interfaz de IA con RAG independiente para documentos                                                          |
| 47  | **librechat**           | Interfaz unificada de IA (reemplazada por open-webui en el compose activo)                                    |
| 48  | **rabbitmq**            | Sistema de mensajería entre servicios                                                                         |
| 49  | **mongodb**             | Base de datos NoSQL                                                                                           |
| 50  | **homeassistant**       | Domótica. Opera en `network_mode: host`, aislado del resto                                                    |
| 51  | **portainer**           | Interfaz de administración Docker. Comentado en el compose activo                                             |
| 52  | **dockge**              | Interfaz gráfica de Docker Compose. Comentado en el compose activo                                            |
| 53  | **tailscale** (`ts-vw`) | VPN para vaultwarden. En desarrollo, comentado                                                                |

---

## 📊 Resumen

```text
🔴 FUNDACIÓN (4)
traefik ── cloudflared ── postgres ── redis
───────────────────────────────────────────────
🟠 CORE (5)
n8n ── open-webui ── open-terminal ── watchtower ── nocodb
───────────────────────────────────────────────
🟡 EXTENSIÓN (3)
flowise ── flowise-worker ── minio ── chrome
───────────────────────────────────────────────
🟢 HERRAMIENTA (4)
pgadmin ── phpadmin ── pgbackup ── vaultwarden
───────────────────────────────────────────────
⚪ OBSERVABILIDAD (2)
uptime-kuma ── dozzle
───────────────────────────────────────────────
🟣 EXPERIMENTO (3)
wordpress ── mariadb ── arcane ── resume
───────────────────────────────────────────────
💤 ARCHIVADO (32)
filestash, wopi_server, appsmith, airflow (×4),
wallos, maybe, resume, memos, prometheus, grafana,
node-exporter, cadvisor, blackbox-exporter,
mongo-express, flame, duplicati, chrome,
evolution-api, rag-api, meilisearch, qdrant,
anythingllm, librechat, rabbitmq, mongodb,
homeassistant, portainer, dockge, tailscale
```

### 📈 Conteo por capa

| Capa             | Cantidad | %   |
| ---------------- | -------- | --- |
| 🔴 Fundación     | 4        | 8%  |
| 🟠 Core          | 5        | 9%  |
| 🟡 Extensión     | 4        | 8%  |
| 🟢 Herramienta   | 4        | 8%  |
| ⚪ Observabilidad | 2        | 4%  |
| 🟣 Experimento   | 4        | 8%  |
| 💤 Archivado     | 30       | 56% |
