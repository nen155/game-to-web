# Plantilla — informe de verificación Playwright

Un fichero por HU: `docs/verificacion/HU-xx.md`.

```markdown
# Verificación HU-xx · <nombre>

**Fecha**: <YYYY-MM-DD> · **Rama/commit**: <id> · **Verificador**: Devin

## Criterios

| # | Criterio de aceptación | Estado | Evidencia |
|---|------------------------|--------|-----------|
| 1 | <texto del criterio> | ✅ PASS | [shot](shots/HU-xx-1.png) |
| 2 | … | 🟡 PARCIAL — <qué falta> | [shot](shots/HU-xx-2.png) |
| 3 | … | ⚫ N/A — cubierto por test `x.test.ts` | — |

## Recorrido ejecutado

1. `npm run dev` (seed <n> / savegame `<nombre>`)
2. <acciones: clics, teclas, navegación>
3. Observado: <qué se vio>

## Notas

- <diferencias frente al original, si son deliberadas>
- Follow-ups: <T-xx nuevo si procede>

**Veredicto**: PASS / PASS CON OBSERVACIONES / FAIL
```

Estados: `PASS` · `FAIL` · `PARCIAL` · `N/A` (sólo si un test unitario citado
cubre el criterio).
