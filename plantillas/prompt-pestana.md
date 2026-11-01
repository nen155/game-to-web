# Plantilla — prompt para una pestaña de Devin (implementación paralela)

Pegar en una pestaña nueva de Devin abierta en el mismo workspace, una por
paquete disjunto (ver `docs/plan-paralelo.md`).

```markdown
Trabaja en el clon web de <juego> (<carpeta del clon>). Lee antes:
`AGENTS.md` (raíz y del clon), `docs/historias-usuario/<ficheros>` y la
ficha de UI `docs/ui/<pantallas>`.

## Tu paquete: <letra — nombre>

Implementa estas historias/tareas: <HU-xx, HU-xy / T-xx, T-xy>
<criterios de aceptación resumidos o referencia a los ficheros>

## Límites

- Puedes crear y editar: `<rutas/ficheros del paquete>`
- NO edites: `game.ts`, `styles.css`, `public/data/*`, `docs/BACKLOG.md`,
  ficheros de otros paquetes: <lista>
- Si necesitas tocar un fichero prohibido, no lo hagas: anótalo en
  `docs/notas-integracion-<paquete>.md`.

## Obligatorio antes de reportar

1. `npx tsc --noEmit && npx vitest run && npx vite build` en verde.
2. Tests nuevos para la lógica de `core/` que añadas.
3. Verifica tus HUs con el MCP playwright contra `npm run dev` y guarda
   evidencias en `docs/verificacion/HU-xx.md` + `shots/`.
4. Haz commit en la rama `feature/<paquete>` con mensaje descriptivo.
5. NO actualices `BACKLOG.md` (lo hace la integración).

Si algo te bloquea, deja el paquete compilando y documenta el bloqueo en
`docs/notas-integracion-<paquete>.md`.
```
