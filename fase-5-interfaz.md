# Fase 5 — Perfilado de la interfaz

**Objetivo:** antes de implementar cada pantalla, saber **exactamente** cómo se
veía y se comportaba en el original: qué piezas la componen, dónde va cada
elemento y qué hace. Produce una ficha por pantalla en `docs/ui/`.

## 5.1 Catálogo de pantallas

De la fase 0/1 se sabe qué assets de UI existen. Cruzarlos con los strings y
con el decompilado para listar **todas** las pantallas:

| Pantalla | Assets que la componen | Trigger |
|----------|------------------------|---------|
| Aventura (HUD) | `Advinter800`, `Advfac64`, `Clocknew`, `Sunmoon` | siempre en el mapa |
| Duelo | `Prd<color>`, `TERR_<color>PATT`, `HAND_`, `LIFE_`, `GRAVE_`, `WINBK_Phase` | encuentro → luchar |
| Asentamiento | `Village`/`City`, `Smbuybttn`, `Buycards`, `Wiseman3`, `Questnew` | entrar en aldea/ciudad |
| Guardar/cargar | `Menopt`, `Optbox` | Esc → lista |

Un asset de fondo/marco sin pantalla asignada (o al revés) indica una feature
olvidada → volver a la fase 3.

## 5.2 Medición con los assets

Para cada pantalla, medir sobre las piezas extraídas (los PNG ya están):

- **Tamaño y posición** de cada ventana/caja embutida. Ejemplo real: el marco
  de carta `CARDBK_*` (228×325) tiene ventana de arte en `(18,21,192×156)` y de
  texto en `(14,200,197×98)` — esas cifras son la especificación del render.
- **Rejillas implícitas**: `WINBK_Phase.bmp` (82×760) resultó ser 2 bloques ×
  8 fases × 2 columnas de 41×47. Medir primero, cortar después.
- **Variantes por resolución**: el original traía `*640`, `*800`, `*1024`.
  Elegir la referencia (960×600 en el clon) y escalar **sin deformar**
  (escala al alto + alas en espejo, `drawScreenFill`).

## 5.3 Referencias externas — cuando el código no basta

El decompilado dice *qué* se dibuja, pero no siempre *cómo se siente*: ritmos,
transiciones, orden de animaciones, sonido asociado. Para eso:

| Recurso | Qué buscar | Cómo usarlo |
|---------|-----------|-------------|
| **Manual / wiki oficial** (archive.org, fandom…) | Capturas oficiales con texto explicativo | Confirma layout y terminología |
| **MobyGames / Wikipedia / Steam** | Capturas de pantalla por versión | Referencia de cada pantalla |
| **YouTube: longplay / gameplay / speedrun** | El juego *en movimiento* | Pausar en cada pantalla; anotar timings, transiciones, qué responde a cada clic |
| **Comunidad del juego** (foros, discords de modding, datamining) | Comportamientos documentados, archivos ya analizados | Confirmar reglas dudosas y features ocultas |
| **El propio juego ejecutado** | Si aún corre (emulador, VM, compat) | La mejor referencia: jugar y capturar |

Registro de cada pantalla (`docs/ui/<pantalla>.md`):

```
# Pantalla: <nombre>
## Fuentes
- assets: <piezas>
- decompilado: <funciones>
- externo: <vídeo, minuto exacto / url de captura>
## Layout
<medidas, posiciones, rejillas>
## Comportamiento
<qué pasa al hacer clic en cada zona, estados, animaciones>
## Criterios visibles
<lo que la verificación de fase 7 podrá comprobar>
```

## 5.4 Reglas de fidelidad aprendidas (generalizables)

- **El arte se respeta, la tecnología no**: recomponer pantallas a la
  resolución objetivo sin estirar; rellenar los márgenes con alas en espejo.
- **Las regiones dan identidad, no el color del suelo**: en el mapa, la
  decoración por región + cuñas de transición es lo que lo hacía parecer el
  original; dar suelo propio a cada edificio creaba «burbujas».
- **Anclas, no cajas**: los sprites ocupan una fracción de su fotograma
  (personaje de 22×37 en celda de 154×108); dibujar y hacer clic sobre la
  caja del **contenido**.
- **Los vídeos cortos → hojas**: secuencias de pocos segundos se reempaquetan
  como sprite sheets (`ffmpeg tile`) y se reproducen en canvas.
- **Anotar cada trampa en `AGENTS.md`** del proyecto en cuanto se descubre:
  es la memoria que evita repetir errores en fase 6.

## Criterio de salida

- [ ] `docs/ui/` tiene una ficha por cada pantalla del catálogo.
- [ ] Cada ficha tiene medidas + comportamiento + al menos una fuente visual.
- [ ] Las referencias externas están citadas con URL/minuto.
- [ ] La lista de pantallas casa 1:1 con las HUs de interfaz (si no, fase 3).
