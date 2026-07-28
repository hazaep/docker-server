# openWebui_compose-recipe-v1.0

## tree

```txt
./
├── docker-compose.yml
├── .env
└── services/
    └── core/
        ├── data/
        |   ├── open-webui/
        |   └── open-terminal/
        └── docker-compose.yml
```

---

## ./docker-compose.yml

```yaml
networks:
  hbs-net:
    name: hbs-net
    driver: bridge

services:
  open-webui:
    extends:
      file: ./services/core/openwebui/docker-compose.yml
      service: open-webui
    networks:
      - hbs-net

  open-terminal:
    extends:
      file: ./services/core/openwebui/docker-compose.yml
      service: open-terminal
    networks:
      - hbs-net
    depends_on:
      - open-webui
```

---

## ./services/core/openwebui/docker-compose.yml

```yml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:latest
    container_name: open-webui
    ports:
      - ":8080"
    volumes:
      - ./data/open-webui:/app/backend/data
    networks:
      - services
    environment:
      - TZ=${TZ}
      - JINA_API_KEY=${JINA_API_KEY}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.webui.rule=Host(`webui.${DOMINIO}`)"
      - "traefik.http.routers.webui.entrypoints=web"
      - "traefik.http.services.webui.loadbalancer.server.port=8080"
      - "com.centurylinklabs.watchtower.enable=true"

  open-terminal:
    image: ghcr.io/open-webui/open-terminal
    container_name: open-terminal
    volumes:
      - ./data/open-terminal:/home/user
    environment:
      - OPEN_TERMINAL_API_KEY=${OPEN_TERMINAL_API_KEY}
      - JINA_API_KEY=${JINA_API_KEY}
      - OPEN_TERMINAL_PACKAGES=${OPEN_TERMINAL_PACKAGES}
      - OPEN_TERMINAL_PIP_PACKAGES=${OPEN_TERMINAL_PIP_PACKAGES}
      - TZ=${TZ}
    networks:
      - services
```

---

## ./.env

```txt
# ================================================
# ======            ENVIRONMENT             ======
# ================================================
# --- CONFIG ---
TZ=America/Ciudad_Juarez
DOMINIO=hb-system.info
ROOT_PATH=/home/path

# ================================================
# ======               core                 ======
# ================================================

# --- OPEN TERMINAL ---
OPEN_TERMINAL_API_KEY=apikey
OPEN_TERMINAL_PACKAGES=syncthing
OPEN_TERMINAL_PIP_PACKAGES=requests numpy pandas
JINA_API_KEY=jina-apikey
```

---
