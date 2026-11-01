# Fase 4 — Elección del stack tecnológico

**Objetivo:** elegir la toolchain del clon con criterios explícitos, antes de
escribir una línea de código de juego.

## Preguntas que deciden el stack

1. **¿Qué renderiza el juego?** Sprites 2D/isométrico → canvas basta.
   3D → WebGL (Three.js/Babylon.js/PlayCanvas). Físicas complejas → motor.
2. **¿Tiempo real o por turnos?** El tiempo real en mapa exige loop de
   animación propio; los duelos por turnos son máquinas de estado.
3. **¿Los assets caben en web?** PNG/JPG/WAV/GLB sí; formatos antiguos
   (MIDI, `.smk`, `.wvl`) requieren conversión (ya hecha en fase 1).
4. **¿De qué se compone el código original?** Si era gestionado y el
   decompilado es casi-fuente (C#/Java), la traducción a TypeScript es casi
   literal: priorizar conservar la estructura de clases para facilitar la
   trazabilidad original↔clon.
5. **¿Multijugador?** Si se quiere online: WebSocket + un relay mínimo en
   Node; si el estado debe ser autoritativo, el modelo «un jugador es
   anfitrión» replica lo que hacía el original.
6. **¿Testeable sin navegador?** Exigencia no negociable (abajo).

## Recomendación por defecto (juego 2D con UI propia)

La usada en `browser-game/`:

| Pieza | Elección | Por qué |
|-------|----------|---------|
| Lenguaje | **TypeScript** estricto | Tipos para el dominio; el decompilado se traduce a interfaces limpias |
| Bundler/dev | **Vite** | Dev server instantáneo, `public/` para los assets tal cual |
| Render | **Canvas 2D** | Control total del pixel-art isométrico; sin dependencias de motor |
| Tests | **Vitest** | El núcleo corre en Node sin navegador |
| Online | **`ws`** + relay Node (~100 líneas) | Salas con estado autoritativo en el anfitrión |
| Persistencia | `localStorage` con namespaces versionados (`juego.v1.*`) | Equivalente web de los savegames |

### Alternativas y cuándo elegirlas

| Alternativa | Pros | Contras | Elegir si… |
|-------------|------|---------|-----------|
| **Phaser** | Escenas, input, tweens resueltos | Convenciones propias que pelean con un port fiel; overhead | El juego es acción pura (scroll, físicas) y no importa replicar UI exótica |
| **PixiJS** | Render WebGL rápido | Sólo render: UI, audio y lógica a mano igual | Miles de sprites en pantalla |
| **Three.js / Babylon.js** | 3D real en web | Reconstruir lógica de engine a mano | El original es 3D |
| **Godot (export web)** | Motor completo, editor | El HTML exportado es pesado; tooling fuera del ecosistema JS | Juego 3D o con muchas físicas; el original ya era Godot |
| **React/DOM** | UI declarativa | Malo para render de mapa por píxeles | El «juego» es básicamente UI/cartas estáticas |
| **Porting directo del engine** (p. ej. OpenTK, emulación WASM) | Máxima fidelidad | Coste alto, casi un proyecto aparte | Hay source port comunitario del juego que compila a WASM |

Para un port fiel de un juego 2D con UI propia, **canvas desnudo** suele ganar:
los motores imponen una estructura y el objetivo es replicar la del original.
Para 3D o cuando exista un *source port* abierto del juego, evaluar esa vía
antes de reescribir desde cero.

## Arquitectura no negociable

```
src/
├── core/    # dominio puro: SIN DOM, SIN fetch, SIN Math.random()
├── ui/      # escenas/canvas: sólo presentación
├── net/     # protocolo y cliente online
└── game.ts  # controlador: las escenas no se conocen entre sí
```

- **`core/` testeable en Node**: toda regla del original (costes, fórmulas,
  encuentros) vive aquí y se cubre con Vitest.
- **Determinismo**: toda la aleatoriedad sale de un PRNG con semilla dentro del
  estado (`state.rng`). Partidas reproducibles, tests estables y modo online
  sincronizable.
- **Escenas desacopladas**: cada escena pide cambios al controlador; ninguna
  importa a otra. Esto es lo que permite implementar pantallas **en paralelo**
  (fase 6) sin pisarse.
- **Serialización explícita**: guardado/cargado y red comparten el mismo
  `serialize/deserialize`; versionarlo desde el día 1.

## Scaffold y comandos mínimos

```bash
npm create vite@latest web-game -- --template vanilla-ts
cd web-game && npm i -D vitest && npm i ws
```

`package.json` debe acabar con, como mínimo:

```json
"dev": "vite", "typecheck": "tsc --noEmit",
"test": "vitest run", "build": "tsc --noEmit && vite build"
```

**Puerta de calidad única**: `typecheck && test && build` — se exige verde
antes de dar cualquier HU por terminada.

## Criterio de salida

- [ ] Decisión documentada (tabla comparativa breve en el README del clon).
- [ ] Scaffold arranca: `npm run dev` muestra una pantalla en blanco.
- [ ] Estructura `core/`/`ui/` creada y `AGENTS.md` del clon con las
  convenciones (dominio puro, PRNG, escenas desacopladas).
- [ ] La puerta de calidad pasa en vacío.
