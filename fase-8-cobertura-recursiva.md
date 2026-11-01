# Fase 8 — Cobertura recursiva: historias e implementación por elemento

**Objetivo:** garantizar que **todo elemento jugable** del juego termina
cubierto por una historia de usuario implementada y verificada. A
diferencia del tramo lineal (fases 3-7: todas las HUs primero, todos los
paquetes después), esta fase **intercala especificación e implementación**:
recorre el inventario completo de elementos del juego de forma recursiva y,
por cada elemento sin cubrir, **crea la HU → la implementa → la verifica →
marca la cobertura → pasa al siguiente**. Cuando la cola queda vacía, se
entra en la fase 9 (caza de bugs).

## Dos usos de la misma fase

- **Modo recursivo (motor principal)**: la fase 8 *sustituye* la secuencia
  3-6 como forma de construir el clon — la ficha de UI, la HU y la
  verificación se hacen por elemento dentro del bucle. Adecuado para juegos
  enormes, extracciones incrementales o cuando el inventario madura mejor
  explorando que listando todo por adelantado.
- **Modo barrido de cierre**: tras el pipeline lineal (3-7 ya ejecutadas),
  la fase 8 actúa como red de seguridad: lo que el inventario tiene y la
  matriz de cobertura no cubrió se detecta, especifica e implementa aquí.

La fase 0 decide el modo y lo documenta; en duda, modo barrido de cierre.

## El elemento: unidad de iteración

Un **elemento** es cualquier cosa jugable detectable en el original:

- pantalla, menú, diálogo, opción de menú o modo de juego;
- mecánica, regla o acción del jugador (incluidos comandos y atajos);
- entidad de datos con comportamiento (carta, unidad, objeto, enemigo);
- estado/transición del juego (inicio, pausa, game over, créditos);
- subsistema (guardado, sonido, online, configuración, ayuda);
- asset con comportamiento propio (animación reactiva, escena, FMV).

Fuentes del inventario: clasificación de la fase 0, `asset_manifest.json`,
strings/menús, decompilado, fichas `docs/ui/` y la matriz de cobertura.

## El bucle

```
┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│ 1. Cola de     │──▶│ 2. Investigar  │──▶│ 3. Escribir la │
│    elementos   │   │    el elemento │   │    HU-xx       │
│    pendientes  │   │                │   │                │
└────────────────┘   └───────┬────────┘   └───────┬────────┘
       ▲                     │ descubre           │ criterios
       │                     │ sub-elementos      │ comprobables
       │                     ▼                    ▼
       │              ┌────────────────┐   ┌────────────────┐
       │              │ encolar nuevos │   │ 4. Implementar │
       │              │ elementos      │   │    la HU       │
       │              └────────────────┘   └───────┬────────┘
       │                                          │
┌──────┴────────┐   ┌────────────────┐   ┌────────▼───────┐
│ 7. Siguiente  │◀──│ 6. Marcar      │◀──│ 5. Verificar   │
│    elemento   │   │    cobertura   │   │    por interfaz│
└───────────────┘   └────────────────┘   └────────────────┘

Cola vacía → cobertura total → FASE 9 (bugs)
```

1. **Cola viva** en `docs/cobertura/elementos.md`: inventario de elementos
   con estado `pendiente / cubierto(HU-xx) / descartado(motivo)`. Es la
   fuente de verdad del avance, no la memoria del agente.
2. **Investigar** el elemento: strings, decompilado, assets, ficha `docs/ui/`
   y referencia externa si aplica. Si la investigación revela
   **sub-elementos** no inventariados, se encolan — la cola puede crecer;
   eso es la recursión.
3. **Escribir la HU** con formato y fuente citada, y actualizar
   `docs/matriz-cobertura.md` en ambas direcciones.
4. **Implementar sólo esa HU** (core + escena), con su test unitario si es
   dominio, y pasar la puerta de calidad antes de seguir.
5. **Verificar por la interfaz** con `mcp-playwright` e informe
   `docs/verificacion/HU-xx.md`, igual que en la fase 7.
6. **Marcar** elemento como cubierto con su HU-xx y commit.
7. Repetir hasta que la cola quede vacía.

## Reglas del bucle

- **Un elemento → una HU → una implementación** por iteración: nada de
  lotes «ya implementaré luego». La gracia del modo es que lo creado se
  construye al momento.
- **Puerta de calidad tras cada HU**
  (`npx tsc --noEmit && npx vitest run && npx vite build`): el clon debe
  estar siempre en estado compilable entre iteraciones.
- **Descartar también cierra**: un elemento no implementable o no
  verificable queda cerrado con motivo documentado en la cola y la matriz
  — no se deja en el aire.
- **La cola es exhaustiva, no la exhaustividad**: si al explorar una
  pantalla aparecen tres opciones de menú no listadas, se añaden a la cola
  en vez de inflar la HU en curso.
- Ficheros compartidos (controlador, estilos, datos) se tocan con
  cambios mínimos por iteración y con la puerta verde inmediata.
- Cada iteración termina con commit; la cola y el backlog se actualizan
  en el mismo commit para no perder estado si se interrumpe.

## Relación con la implementación paralela

El bucle recursivo es **secuencial por naturaleza** (un elemento detrás de
otro). Si a mitad de camino se detecta un dominio grande paralelizable
(p. ej. «todas las fichas de combate»), se puede abrir el modo de la fase 6
sólo para ese dominio: los elementos se extraen de la cola a un paquete,
se implementan en paralelo y vuelven marcados a la cola al integrar.

## Criterio de salida

- [ ] `docs/cobertura/elementos.md` sin ningún `pendiente`: todo elemento
      cubierto por HU implementada y verificada, o descartado con motivo.
- [ ] Matriz de cobertura coherente con la cola (cobertura total).
- [ ] Puerta de calidad verde en el último commit.
- [ ] Entrar a la **fase 9 — caza recursiva de bugs** con el clon ya
      jugable de punta a punta.
