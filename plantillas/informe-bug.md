# Plantilla — informe de bug (caza recursiva)

Un fichero por bug: `docs/bugs/BUG-xx.md`. Tabla resumen viva en
`docs/bugs/README.md`.

```markdown
# BUG-xx · <título corto>

**Fecha**: <YYYY-MM-DD> · **Commit**: <id> · **Batería**: <exploratoria /
monkey / límites / transiciones / persistencia / determinismo / consola /
responsive / soak / online / fidelidad> · **Severidad**: bloquea /
deforma / menor

## Síntoma

<qué se vio por la interfaz y qué esperaba el jugador o la especificación>

## Reproducción mínima

1. Estado inicial: seed <n> / savegame `<nombre>` / teletransporte a <estado>
2. <acciones exactas por la interfaz: clics, teclas, navegación>
3. Observado: <…> · Esperado: <…>

Reproduce: 2/2 intentos · [shot antes](shots/BUG-xx-1.png) ·
[shot después](shots/BUG-xx-2.png)

## Consola / red

```
<errores uncaught, stack, 404s, warnings relevantes>
```

## Diagnóstico

<causa raíz; fichero:línea>

## Resolución

- Fix: <commit / descripción corta>
- Test de regresión: `<ruta/del/test>` (unitario en `core/` | spec
  Playwright)
- Puerta de calidad tras el fix: ✅
- Colateral re-barrido: <baterías re-ejecutadas | ninguna>

**Estado**: abierto / en curso / cerrado / known-issue <motivo>
```
