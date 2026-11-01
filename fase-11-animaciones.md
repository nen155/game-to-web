# Fase 11 — Animaciones y acciones por personaje

**Objetivo:** que **cada personaje** del roster de la fase 10 camine y
ejecute **todas las acciones que el original define para él**. Cargar el
modelo estático no basta: un personaje que se desliza sin animar o cuyo
ataque no reproduce su clip es un pendiente. Esta fase cierra el frente
con dos garantías automáticas: un spec que recorre la matriz
personaje × animación por el loader real, y una **prueba de movimiento
por personaje** hecha por la interfaz.

## La matriz: personaje × animación

`docs/cobertura/animaciones.md` — una fila por pareja personaje/clip:

| Personaje | Animación | Fuente | Estado |
|-----------|-----------|--------|--------|
| david | walk | `ANIMS.TXT` / `david.CHR` | `pendiente / anima / N-A(motivo)` |

La columna de animaciones no se inventa: sale del catálogo de animaciones
del juego (fichero índice, enum del decompilado, nombres referenciados en
los scripts) más lo observado jugando. Cada clip nuevo descubierto se
**encola** — la matriz puede crecer.

Clips mínimos a considerar por personaje jugable (adaptar al juego):
`idle`, `walk`, `run`, `attack` (cada variante), `cast`, `hit`, `die`,
más las acciones especiales del diseño (saltar, nadar, sentarse,
hablar…). Un `N-A` exige motivo («los enemigos tipo X nunca corren en el
original — fuente: script Y»).

## Pruebas a dos niveles

### Nivel core (Vitest)

- El spec de carga de la fase 10 se extiende: por cada fila de la matriz,
  el clip **decodifica** (frames > 0, duración > 0, keyframes válidos).
- La máquina de estados de animación: `idle→walk` al mover, `walk→idle`
  al parar, acción interrumpe locomoción y **devuelve** al estado que
  toca, `die` no vuelve. Determinista sobre el PRNG del juego.

### Nivel interfaz (Playwright) — la prueba de movimiento

Por **cada personaje**, sobre `npm run dev`:

1. **Spawn** del personaje (hook de dev / escena sandbox) en un mapa
   transitable.
2. **Movimiento real**: orden de moverse por el input verdadero (clic en
   el suelo / teclas) y comprobar que la posición cambia **y** que la
   pose avanza — dos capturas a distintos ticks deben diferir en la
   silueta (no deslizamiento congelado).
3. **Cada acción del personaje** disparada por su input real (ataque,
   magia, interacción…): el clip se reproduce **y** ocurre el efecto de
   juego (daño aplicado, proyectil, sonido pedido).
4. Evidencia: par de capturas por movimiento y una por acción a
   `docs/verificacion/anim/<personaje>-*.png`.

## El banco de animaciones (muy recomendado)

Una escena sandbox — p. ej. `?debug=anim&char=<id>&clip=<nombre>` — que
instancia cualquier personaje y reproduce cualquier clip con el renderer
real. Acelera la barrida (no hay que llevar el juego al estado donde vive
ese NPC), produce la evidencia de cada fila y queda como herramienta de
depuración permanente del clon.

## El bucle

```
┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
│ 1. Siguien-│──▶│ 2. Spec de │──▶│ 3. Prueba  │──▶│ 4. Marcar  │
│ te fila de │   │ carga del  │   │ de movim. /│   │ fila en la │
│ la matriz  │   │ clip       │   │ acción     │   │ matriz     │
└────────────┘   └────────────┘   └─────┬──────┘   └────────────┘
       ▲                                │ falla → BUG-xx: repro,
       │ clip/personaje nuevo           │ test rojo, fix, puerta
       └────────────────────────────────┘

Matriz sin pendientes → SALIR
```

- Un clip que no carga o no se ve: bug del decoder/renderer — mismo
  protocolo que la fase 9 (repro, test de regresión antes del fix).
- Un clip que carga pero **no se usa** (el personaje ataca sin animar) es
  bug del controlador/máquina de estados, no del asset: se repro por la
  interfaz y se arregla el enlace, no el clip.

## Reglas

- **Andar siempre.** Todo personaje que se mueve en el original tiene su
  clip de locomoción cargado al cargar el modelo: la animación es parte
  del asset, no un extra posterior.
- **Por la interfaz, no por fe.** Que el clip exista en disco o decodifique
  no prueba que el personaje lo use: la prueba de movimiento es por
  Playwright con input real.
- **Las acciones cuentan lo visual y lo funcional**: animación sin efecto
  (o efecto sin animación) es fila roja.
- Personajes que comparten esqueleto/rig (variantes de paleta) heredan la
  verificación del rig, pero cada variante con acciones propias se prueba.
- Si el formato de animación del juego no está resuelto aún, esa RE es el
  primer pendiente de la matriz — y se documenta en `AGENTS.md`.

## Criterio de salida

- [ ] `docs/cobertura/animaciones.md` sin `pendiente`: cada fila `anima`
      o `N-A(motivo)`.
- [ ] Spec de carga de clips verde sobre toda la matriz.
- [ ] Prueba de movimiento por personaje en PASS con evidencia en
      `docs/verificacion/anim/`.
- [ ] Banco `?debug=anim` funcional listado en el README del clon.
- [ ] Puerta de calidad verde y continuar a la **fase 12 — HUDs**.
