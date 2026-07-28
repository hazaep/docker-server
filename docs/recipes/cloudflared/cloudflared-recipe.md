# Cloudflared — Recipe v1.0

## 🎯 Propósito

Túnel seguro entre Cloudflare y la Raspberry Pi. **El puente que conecta Bildung con internet.**

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `cloudflare/cloudflared:latest` |
| Reinicio | `unless-stopped` |
| Sin puertos | Conexión saliente al edge de Cloudflare |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| traefik | Destino del tráfico | Sí |
| Internet | Conexión saliente | **Sí** |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `CLOUDFLARE_TUNNEL_TOKEN` | **Sí** | Token JWT del túnel |
| `TZ` | No | Zona horaria |

---

## 📂 Volúmenes

Sin volúmenes. Totalmente efímero. El túnel se reconstruye con el token.

---

## 🚀 Deploy

```bash
docker compose up -d cloudflared
```

---

## 🩺 Health check

```bash
# Verificar que el túnel está conectado
docker compose logs cloudflared | tail -5
# Esperado: "Registered tunnel connection"

# Verificar que el tráfico llega a traefik
curl -s -o /dev/null -w "%{http_code}" https://traefik.${DOMINIO}
# Esperado: 200
```

---

## 🧹 Backup

No requiere backup. Si se pierde el contenedor, se recrea con el token.

---

## 🔄 Rotar token

```bash
# 1. Crear nuevo token en Cloudflare Zero Trust dashboard
# 2. Actualizar .env
# 3. Recrear
docker compose up -d --force-recreate cloudflared
docker compose logs cloudflared | tail -3
# Esperado: Registered tunnel connection
```

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| Túnel no conecta | Token inválido o expirado | Rotar token en Cloudflare dashboard |
| `connection refused` | Sin internet | Verificar conectividad: `ping 1.1.1.1` |
| Tráfico no llega a apps | Traefik caído | `docker compose ps traefik` |
| DNS no resuelve | Registro CNAME en Cloudflare incorrecto | Verificar DNS → Túnel en Zero Trust dashboard |
| Reinicio en loop | Token mal copiado | Revisar que no tenga espacios ni saltos de línea |
