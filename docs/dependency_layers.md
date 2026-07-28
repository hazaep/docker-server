# Clasificación por Capas de Dependencia

## 🧠 Cómo clasificar

El criterio es simple: **¿qué pasa si este servicio deja de funcionar?**

Cada servicio pertenece a una capa según el *radio de impacto* de su caída:

| Capa | Definición | Pregunta clave |
|---|---|---|
| 🔴 **Fundación** | Si cae, **todo** el ecosistema colapsa o queda inaccesible. Sin esta capa no hay nada. | ¿Sin esto funciona algo más? |
| 🟠 **Core** | Si cae, la **funcionalidad principal** del ecosistema se degrada gravemente. No todo muere, pero el propósito central se pierde. | ¿Esto define lo que Bildung *hace*? |
| 🟡 **Extensión** | Si cae, algunas **funcionalidades específicas** desaparecen, pero el núcleo sigue operando. Son aceleradores, no cimientos. | ¿Esto potencia pero no define? |
| 🟢 **Herramienta** | Si cae, solo pierdes **conveniencia**. Nada crítico depende de ello. Son interfaces de administración o utilidades. | ¿Es un atajo o una ventana? |
| ⚪ **Observabilidad** | Si cae, **operas ciego** pero el ecosistema funciona. Saber que algo falla es distinto a que algo falle. | ¿Esto produce información o produce valor? |
| 🟣 **Experimento** | Si cae, el ecosistema **ni se entera**. Son apps en exploración, no integradas en cadenas de dependencia. | ¿Si desaparece mañana, alguien lo nota? |
| 💤 **Archivado** | Está comentado. No corre. Pero el diseño existe por si vuelve. | — |

### ⚠️ Reglas adicionales

- Un servicio que depende de otro **nunca** está en una capa superior a su dependencia. Ej: `nocodb` depende de `postgres`, así que `nocodb` (🟠) siempre estará debajo de `postgres` (🔴).
- Si un servicio es dependencia de múltiples servicios en capas distintas, se clasifica en la capa más alta de sus dependientes.
- La capa se asigna por el **peor escenario**, no por el mejor.
- Un servicio puede subir de capa si su valor estratégico lo justifica, aunque técnicamente pertenezca a una inferior por dependencias.
