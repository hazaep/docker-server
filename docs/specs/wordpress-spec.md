# 📘 WordPress — Service Specification v1.0

**Capa:** 🟣 Experimento | **Enfoque:** Spec-First / Infrastructure Contract | **Estado:** Living Spec

---

## 1. Propósito

CMS / blog personal. **Es un lienzo para escribir, no infraestructura.** Si cae, Bildung sigue funcionando. Si WordPress deja de ser útil, se archiva sin consecuencias.

---

## 2. Definición del Servicio

> WordPress con MariaDB como backend. Named volume para temas, plugins y uploads. Expuesto vía Traefik.

### 🔑 Principios

| Principio | Regla |
|---|---|
| **Aislado** | Su base de datos es exclusiva. No comparte Postgres |
| **Sin plugins críticos** | Si un plugin rompe, se desactiva. No afecta al ecosistema |
| **Contenido, no sistema** | WordPress almacena posts, no datos operativos de Bildung |

---

## 3. Contrato de Infraestructura

### 🔹 Salida

| Recurso | Valor |
|---|---|
| **UI** | `https://wordp.${DOMINIO}` |
| **Puerto interno** | `80` |

### 🔹 Dependencias

```text
wordpress
  └── mariadb (🟣 Experimento)
```

---

## 4. Volumen y Persistencia

| Tipo | Nombre | Path interno | Contenido |
|---|---|---|---|
| Named volume | `wordp_data` | `/var/www/html` | Temas, plugins, uploads, wp-config |

---

## 5. Criterios de Aceptación

| # | Criterio | Verificación |
|---|---|---|
| **CA-1** | Contenedor inicia | `docker compose ps wordpress` → Up |
| **CA-2** | UI accesible | `curl -s https://wordp.${DOMINIO}` → HTML de WordPress |
| **CA-3** | Admin panel carga | `https://wordp.${DOMINIO}/wp-admin` → login |
| **CA-4** | Conexión a MariaDB | Publicar un post → se guarda y se ve en el front |
| **CA-5** | Uploads funcionan | Subir una imagen → se muestra en el post |

---

📌 **Nota SDD:** WordPress está en Experimento, no en Herramienta, porque su propósito es expresión personal, no operación del ecosistema. Si mañana decides migrar a un static site generator, nada más se rompe.
