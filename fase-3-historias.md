# Fase 3 — Historias de usuario con cobertura completa

**Objetivo:** traducir todo lo encontrado en las fases 0-2 en una especificación
de producto: historias de usuario con criterios de aceptación comprobables, de
tal forma que **no falte ninguna característica del original**.

## Fuentes de features (en orden de fiabilidad)

| # | Fuente | Qué aporta | Ejemplo |
|---|--------|-----------|---------|
| 1 | **Código fuente / decompilado gestionado** (si existe) | La verdad completa: clases, métodos, constantes con nombre | `CombatManager.cs` entero |
| 2 | **Strings/menús del juego** | Todas las opciones que el original ofrece | `@ENCOUNTER_PREDUEL` → las 6 salidas del reto (no 3) |
| 3 | **Readme / manual** | Reglas declaradas por el propio juego | «monstruos atados a su terreno», tamaños de mazo por dificultad |
| 4 | **Decompilado nativo** | Reglas no documentadas, umbrales, fórmulas | la tregua tras perder un duelo; condición de embestida |
| 5 | **Assets** | Cada pieza de UI implica una pantalla | `WINBK_Phase.bmp` → indicador de fases existe |
| 6 | **Datos** (JSON verificados) | Entidades que deben existir en el clon | 1601 cartas, 57 retratos, 19 guaridas |
| 7 | **Referencias externas** | Contexto y confirmación | manual, wiki oficial, MobyGames, gameplays (fase 5) |

## Procedimiento

### 3.1 Documento de visión (`00-vision.md`)

Antes de las HUs, un resumen operativo del original:

- Qué pantallas tiene y cómo se navega entre ellas.
- Qué recursos gestiona el jugador.
- Qué bucles de juego existen (viaje → encuentro → duelo → recompensa).
- **Qué queda fuera** del port y por qué (p. ej. el multijugador por cable
  original, DRM, compatibilidad Windows).
- Fuentes consultadas, en tabla.

### 3.2 Inventario de features

Recorrer sistemáticamente:

1. **Cada `@TAG`/cadena de menú** de los strings → una opción → una feature.
2. **Cada pantalla identificable** en los assets → una feature.
3. **Cada regla del readme/manual** → una feature.
4. **Cada mecánica vista en el decompilado** → una feature.

Agrupar el inventario por dominios (en Shandalar: mapa/estado, encuentros,
lugares, duelos, mundo/magias, guardado, online…).

### 3.3 Escribir las HUs

Formato (ver [plantilla](plantillas/historia-usuario.md)):

```
### HU-xx · <nombre>
**Como** jugador, **quiero** <capacidad> **para** <beneficio>.

- Criterio de aceptación comprobable 1
- Criterio 2 (con cifras/literales del original cuando existan)
```

Reglas de calidad:

- **Criterios comprobables**: «máximo 2 ventajas», «3 a 5 días», «F2 abre el
  mapa» — no «funciona bien».
- **IDs estables**: `HU-xx` no se reusa ni renumera; el backlog referencia
  `T-xx ↔ HU-xx`.
- **Agrupar por dominio**, un fichero por tanda (`01-historias.md`,
  `02-historias-avanzadas.md`, `03-duelos.md`, `04-mundo-y-final.md`).
- **Citar la fuente** cuando el criterio venga del original («según
  `@LAIR`», «el readme dice…»). Esto es lo que distingue una especificación
  fiel de una invención.

### 3.4 Matriz de cobertura — el paso que garantiza «que no falte ninguna»

Ver [plantilla](plantillas/matriz-cobertura.md). Dos direcciones:

1. **Feature → HU**: cada item del inventario aparece cubierto por ≥1 HU, o
   marcado como *descartado* con motivo.
2. **HU → fuente**: cada HU cita de dónde sale (string, readme, decompilado).
   Una HU sin fuente es sospechosa de ser inventada.

La matriz se revisa al final de la fase y tras cada tanda nueva: cuando el
decompilado revela mecánicas que ninguna HU cubría (en Shandalar pasó con
`Advstrings.txt`: los 7 World Magics restantes, los tributos del sabio…),
se abre una tanda adicional (P5).

### 3.5 Backlog (`BACKLOG.md`)

- Tareas `T-xx` de implementación mapeadas a `HU-xx` (una tarea puede cubrir
  varias HUs).
- Prioridades `P0..Pn`: **P0 = el bucle de juego mínimo jugable**; el resto por
  profundidad, no por orden de escritura.
- Estados `✅ 🟡 ⬜` y checkboxes internos — el backlog es el panel de control
  de todo el proyecto.

## Criterio de salida

- [ ] `00-vision.md` escrito.
- [ ] Inventario de features completo (strings + readme + decompilado + assets
  + datos).
- [ ] HUs agrupadas con criterios comprobables y fuente citada.
- [ ] Matriz de cobertura: 0 features sin HU ni justificación.
- [ ] Backlog con P0 acotado a «bucle jugable».
