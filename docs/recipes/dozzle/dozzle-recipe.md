# Dozzle — Recipe v1.0

## 🎯 Propósito

Visor de logs en tiempo real. **Si cae, `docker compose logs` hace lo mismo.** Dozzle solo lo hace más rápido y bonito.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `amir20/dozzle:latest` |
| Puerto interno | `8080` |
| Reinicio | `unless-stopped` |

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| docker.sock (host) | Acceso a logs de contenedores | **Sí** |
| traefik | Reverse proxy | Sí |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `TZ` | No | Zona horaria para timestamps |

---

## 📂 Volúmenes

| Path contenedor | Path host |
|---|---|
| `/var/run/docker.sock` | Host |

---

## 🚀 Deploy

```bash
docker compose up -d dozzle
```

---

## 🩺 Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://logs.${DOMINIO}
# Esperado: 200
```

---

## 🧹 Backup

No requiere backup. Sin estado.

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No muestra contenedores | docker.sock sin permisos | `ls -la /var/run/docker.sock` |
| Logs vacíos | Contenedor sin output | Verificar con `docker compose logs <servicio>` |
| UI lenta con muchos containers | Filtro muy amplio | Usar el filtro por nombre en la UI |

### ⚠️ Seguridad

Dozzle no tiene autenticación. Los logs pueden contener variables de entorno con contraseñas. No exponer sin Traefik. Si necesitas auth, agregar middleware `basic-auth` en las labels de Traefik.
