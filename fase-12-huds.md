# Fase 12 — Fidelidad recursiva de HUDs

**Objetivo:** que **cada HUD y panel** del clon sea igual al diseño — la
ficha `docs/ui/` de la fase 5 y la referencia del original — no sólo el
HUD principal. «De manera recursiva» significa dos cosas: la cola recorre
todos los HUDs y, al revisar cada uno, los **estados y variantes** que se
descubren (vida baja, inventario lleno, grupo de 1 vs. 3, versión con
objetivo activo…) se encolan como entradas propias hasta que no queda
nada sin comparar.

## La cola

`docs/cobertura/huds.md` — una fila por HUD/panel/variante:

| HUD | Variante/estado | Ficha | Estado |
|-----|-----------------|-------|--------|
| HUD combate | base | `docs/ui/hud.md` | `pendiente / conforme / corregido(BUG-xx)` |
| HUD combate | HP < 25 % | idem | `pendiente` |

Estados: `pendiente`, `conforme` (comparado y aprobado, con evidencia),
`corregido(BUG-xx)` (tenía diferencias, ya arregladas y re-comparado),
`descartado(motivo)` (imposible de reproducir fielmente — documentar).

Semillas de la cola: todas las fichas `docs/ui/`, los `.SPR`/assets de
interfaz de la fase 1 (cada marco implica un panel), los menús y
submenús, los diálogos, el mapa, la ficha del ratón (cursores por
contexto), las pantallas de estado (pausa, game over, créditos) y los
mensajes/notificaciones flotantes.

## El bucle

```
┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
│ 1. Siguien-│──▶│ 2. Repro   │──▶│ 3. Comparar│──▶│ 4. ¿Confo- │
│ te HUD de  │   │ el estado  │   │ lado a lado│   │ rme?       │
│ la cola    │   │ en el clon │   │            │   │            │
└─────┬──────┘   └────────────┘   └────────────┘   └─────┬──────┘
      │                                                 │ no → BUG-xx:
      │ variantes/estados          sí → conforme        │ fix → re-comparar
      │ descubiertos se            (marcar + shots)      │ o spec → corregir
      │ encolan                                          │ ficha
      └──────────────────────────────────────────────────┘

Cola sin pendientes → SALIR
```

1. **Siguiente HUD** pendiente de la cola.
2. **Reproducir el estado exacto** en el clon (hook de dev: vida al 10 %,
   inventario lleno, grupo completo) — sin hooks, comparar el estado raro
   cuesta el doble.
3. **Comparar lado a lado**: captura del clon contra la ficha `docs/ui/`
   y la referencia del original (captura propia del juego, MobyGames o
   frame de gameplay con minuto citado), con la checklist de abajo.
4. **Juzgar**: si no hay diferencias → `conforme` + evidencia
   (`docs/verificacion/huds/<hud>-*.png`, par original/clon en el
   informe). Si las hay → bug de fidelidad (`docs/bugs/BUG-xx.md`,
   severidad `deforma`/`menor`), fix y **re-comparación** — no se cierra
   sin la segunda captura. Si la diferencia es que **la ficha mintió**,
   el bug es de spec: se corrige ficha/HU y se re-verifica (misma regla
   que la fase 9).

## Checklist por HUD

| Frente | Qué se mira |
|--------|-------------|
| Composición | Posición y anclaje de cada pieza (esquinas, centrado, orden) |
| Medidas | Tamaños sobre los sprites extraídos; escala sin deformar |
| Tipografía | Fuente correcta, tamaño, color, sombreado, alineación |
| Color/paleta | Colores exactos de la paleta original, no aproximados |
| Iconos/retratos | Sprite correcto por contexto, recortes y marcos |
| Dinámica | Barras que se llenan/vacían, parpadeos, contadores, iconos de estado |
| Estados | Vacío, lleno, límite, deshabilitado, resaltado, hover/press |
| Escalado | El HUD mantiene su disposición al redimensionar la ventana |

## Reglas

- **Recursivo de verdad**: cada variante de estado es una fila. Descubrir
  «el inventario cambia con 0/1/N objetos» encola tres filas, no una nota
  mental.
- **Evidencia obligatoria**: `conforme` sin par de capturas no vale — la
  evidencia es lo que permite auditar la fidelidad después.
- **La diferencia se clasifica**: bug de implementación (arreglar el
  clon) o bug de spec (arreglar la ficha y re-verificar); nunca «se parece
  bastante».
- **Interacción incluida**: un HUD también es su comportamiento — qué
  abre cada clic, qué muestra el tooltip, qué hace la rueda del pie menu.
- Cada fix de fidelidad que descubra una convención (rejilla, offset,
  regla de anclaje) la anota en la ficha `docs/ui/` y en `AGENTS.md` si
  es una trampa del formato.

## Criterio de salida

- [ ] `docs/cobertura/huds.md` sin `pendiente`: todo `conforme`,
      `corregido(BUG-xx)` o `descartado(motivo)`.
- [ ] Par de capturas original/clon por cada fila conforme en
      `docs/verificacion/huds/`.
- [ ] Cero diferencias sin clasificar; cada `corregido` tiene su
      re-comparación.
- [ ] Fichas `docs/ui/` actualizadas con lo aprendido en la barrida.
- [ ] Puerta de calidad verde → **cierre del pipeline**.
