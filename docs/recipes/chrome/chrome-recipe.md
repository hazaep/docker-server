# Chrome (Browserless) — Recipe v1.0

## 🎯 Propósito

Navegador headless para PDFs y screenshots. **Invisible para el usuario final.** Solo lo consume Reactive Resume.

---

## 📦 Imagen

| Campo | Valor |
|---|---|
| Image | `ghcr.io/browserless/chromium:v2.18.0` |
| API HTTP | `3000` (interno) |
| WebSocket | `ws://chrome:3000` |
| Reinicio | `unless-stopped` |

> ⚠️ Versión congelada en v2.18.0. Versiones más nuevas rompen en ARM64.

---

## 🔗 Dependencias

| Servicio | Tipo | Obligatorio |
|---|---|---|
| resume | Único consumidor | No (chrome inicia sin él) |

---

## 🔐 Variables de entorno

| Variable | Obligatorio | Default | Descripción |
|---|---|---|---|
| `CHROME_TIMEOUT` | No | `30000` | Timeout por trabajo (ms) |
| `CHROME_CONCURRENT` | No | `10` | Trabajos concurrentes máximos |
| `CHROME_TOKEN` | No | — | Token de autenticación |
| `CHROME_EXIT_FAILURE` | No | `false` | Salir si health check falla |
| `CHROME_HEALTH_CHECK` | No | `true` | Verificar readiness antes de aceptar |
| `TZ` | No | `UTC` | Zona horaria |

---

## 📂 Volúmenes

Sin volúmenes. Completamente efímero.

---

## 🚀 Deploy

```bash
docker compose up -d chrome
```

---

## 🩺 Health check

```bash
# Verificar readiness
docker compose logs chrome | grep "ready"
# Esperado: "Chrome is ready"

# API responde
curl -s http://chrome:3000
# Esperado: HTML de browserless

# Generar PDF de prueba
curl -X POST http://chrome:3000/pdf \
  -H "Content-Type: application/json" \
  -d '{"url":"http://example.com"}' \
  -o test.pdf
# Esperado: archivo PDF generado
```

---

## 🧹 Backup

No requiere backup. Sin estado.

---

## 🐛 Troubleshooting

| Síntoma | Causa probable | Solución |
|---|---|---|
| No inicia | Imagen incompatible con ARM64 | ⚠️ Solo usar `v2.18.0`. No actualizar |
| Resume no exporta PDFs | Chrome no accesible | `curl http://chrome:3000` desde dentro de la red |
| Timeout en PDFs grandes | `CHROME_TIMEOUT` muy bajo | Subir a `30000` o más |
| Consume mucha RAM | Múltiples trabajos concurrentes | Reducir `CHROME_CONCURRENT` a 5 |
| Se congela | Bug conocido en ARM64 | Reiniciar: `docker compose restart chrome` |
