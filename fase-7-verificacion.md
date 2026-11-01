# Fase 7 — Verificación de historias con Playwright

**Objetivo:** comprobar **en el navegador real** que cada HU cumple sus
criterios de aceptación. Los tests unitarios prueban el dominio; esta fase
prueba que el jugador ve y puede hacer lo que la historia promete.

## Herramienta

El MCP **`mcp-playwright`** (ya configurado en Devin): navega, hace clic,
lee la consola y hace capturas. Si no estuviera disponible, un script de
Playwright (`npx playwright`) cumple el mismo papel.

## Procedimiento por HU

1. **Arrancar** el dev server (`npm run dev`) y, si la HU es online, el relay
   (`npm run server`).
2. **Preparar el estado**: semilla fija, savegame precargado o acciones
   rápidas (inyectar estado por `localStorage`/consola si hace falta) para
   llegar a la situación de la HU sin jugarse media partida.
3. **Ejecutar el criterio**: navegar, hacer clic, escribir — como el jugador.
4. **Evidenciar**: captura de pantalla del estado resultante a
   `docs/verificacion/shots/HU-xx-<n>.png`.
5. **Juzgar**: cada criterio de aceptación → PASS / FAIL / PARCIAL, con
   nota de qué se vio.
6. **Registrar**: informe por HU en `docs/verificacion/HU-xx.md`
   ([plantilla](plantillas/informe-playwright.md)).

## Qué se comprueba (y cómo)

| Tipo de criterio | Técnica |
|------------------|---------|
| «Aparece X en pantalla» | Captura + `page.locator`/lectura del canvas; para canvas, captura + aserción visual descrita en el informe |
| «Al hacer clic pasa Y» | Acción en Playwright + captura antes/después + estado en consola |
| «El número se actualiza» | Exponer estado legible (`window.__state` en dev, o texto en DOM) y leerlo |
| «Suena tal efecto» | Difícil de afirmar: comprobar que se dispara la petición/llamada (spy/log en dev) |
| «Se ve como el original» | Comparar con la ficha `docs/ui/` y la referencia externa (captura lado a lado en el informe) |

**Consejo**: en modo dev, exponer el estado del juego en `window` y un método
de *teletransporte* de estado (cargar partida, fijar recursos). Sin eso,
verificar una HU de endgame puede costar horas.

## Orden y cobertura

- Verificar **al cerrar cada HU**, no al final del proyecto: un fallo de
  diseño detectado tarde cuesta rehacer paquetes.
- Las HUs que tocan flujo entre escenas (viaje → duelo → recompensa) se
  verifican además como **camino completo** una vez por paquete.
- En paralelo (fase 6), cada pestaña verifica **sus** HUs antes de reportar;
  la integración re-verifica los caminos que crucen paquetes.

## Informe y estados

Estados por criterio: `PASS`, `FAIL` (con qué se vio + captura), `PARCIAL`
(qué parte falta), `N/A` (no verificable en navegador — debe tener test
unitario que lo cubra, citado en el informe).

Resumen consolidado en `docs/verificacion/README.md` con una tabla
HU → estado → evidencia. Una HU sólo es `✅` en el backlog cuando su informe
está en PASS (o sus PARCIAL están aceptados y anotados como follow-up T-xx).

## Puerta de calidad del proyecto

```bash
npx tsc --noEmit && npx vitest run && npx vite build
```

Se ejecuta tras cada integración y antes de cerrar cualquier tanda.

## Criterio de salida

- [ ] Informe por HU implementada en `docs/verificacion/`.
- [ ] Capturas de evidencia enlazadas desde cada informe.
- [ ] Tabla resumen HU→estado actualizada.
- [ ] Backlog coherente con los informes (nada en ✅ sin PASS).
