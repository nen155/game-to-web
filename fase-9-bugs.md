# Fase 9 — Caza recursiva de bugs por la interfaz

**Objetivo:** encontrar y corregir los bugs que la verificación por HU no
atrapa, jugando al clon **por su interfaz real** con todo tipo de pruebas.
No es una pasada final: es un **bucle** — cada bug confirmado genera
reproducción mínima, test de regresión y arreglo, y la caza continúa hasta
que una barrida completa sale limpia.

## Diferencia con la fase 7

La fase 7 responde «¿la HU se cumple?»; esta fase responde «¿dónde se
rompe?». No hay criterios de aceptación que validar: se juega **mal a
propósito** — el jugador curioso, el impaciente, el que pulsa todo dos
veces y el que cierra en mitad de una acción.

## Precondiciones

- MCP `mcp-playwright` contra `npm run dev` (+ relay si hay online). Si no
  está disponible, scripts `npx playwright` cumplen el mismo papel.
- Hooks de dev activos: `window.__state` legible, teletransporte de estado
  y seed fija. Sin ellos la caza cuesta el doble y no reproduce.
- Registro preparado: `docs/bugs/` con un `BUG-xx.md` por hallazgo
  ([plantilla](plantillas/informe-bug.md)) y tabla en
  `docs/bugs/README.md`.

## El bucle

```
┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐
│ 1. Barrer │──▶│ 2. Con-   │──▶│ 3. Repro  │──▶│ 4. Test + │──▶│ 5. Puerta │
│ (baterías)│   │ firmar    │   │ mínima    │   │ fix       │   │ de calidad│
└─────┬─────┘   └───────────┘   └───────────┘   └───────────┘   └─────┬─────┘
      ▲                                                             │
      └────────── reanudar la barrida donde quedó ◀─────────────────┘

Barrida completa sin bugs nuevos → SALIR
```

1. **Barrer**: ejecutar la batería completa sobre un área anotando todo lo
   sospechoso sin pararse en cada cosa (una interrupción por hallazgo mata
   el ritmo; mejor lista de sospechas y luego confirmar).
2. **Confirmar**: ¿es bug o fidelidad? Se contrasta con la ficha
   `docs/ui/`, la HU y, si hay duda, el original. Falso positivo → nota,
   no bug.
3. **Reproducir mínimo**: camino más corto desde un estado conocido
   (seed/teletransporte). Debe reproducir **dos veces**; si no, queda como
   nota de investigación con su contexto — no se «arregla» lo que no se ve.
4. **Test de regresión antes del fix**: unitario en `core/` si el bug es
   de dominio; spec Playwright si es de interfaz. El test debe fallar con
   el bug presente — si no falla, no captura el bug.
5. **Fix + puerta de calidad completa**
   (`npx tsc --noEmit && npx vitest run && npx vite build`) y evidencia de
   cierre (shot post-fix) en el informe del bug.
6. **Reanudar la barrida donde quedó**, no desde cero. Excepción: un fix
   en `core/` compartido o en el controlador obliga a re-barrer las
   baterías de estado/persistencia del área afectada.

## Las baterías

Se ejecutan **por pantalla/sistema**, no sólo globalmente: cada escena
merece su pasada.

| Batería | Qué se hace por la interfaz |
|---------|------------------------------|
| **Exploratoria** | Recorrer cada pantalla, menú, botón y opción con ojos de jugador nuevo: todo responde, se entiende y se puede volver |
| **Monkey / fuzz** | Clics y teclas aleatorias durante N segundos por pantalla, rápido y sin esperar; input inválido en campos (texto en números, pegar portapapeles, emoji, longitudes absurdas) |
| **Límites** | 0/1/máx/máx+1 en recursos y contadores; listas vacías y llenas; texto que desborda su caja; colección de 1 y de 999 elementos |
| **Transiciones de estado** | ESC/volver en mitad de cada acción, doble clic en confirmar, atrás/adelante del navegador, F5 en cada escena, dos acciones simultáneas |
| **Persistencia** | Guardar → recargar → cargar → comparar `window.__state`; saves de versiones anteriores; save truncado/corrupto; `localStorage` borrado o lleno en mitad de partida |
| **Determinismo** | Misma seed + mismo guion de inputs → mismo estado final, repetido 2-3 veces; divergencia = estado oculto o `Math.random`/`Date.now` fugado al core |
| **Consola y red** | Tras cada batería: cero `uncaught`, cero 404 de assets, cero warnings propios acumulándose; **todo error en consola es un bug** |
| **Responsive/entorno** | Redimensionar la ventana en vivo (mínimo y proporción extrema), zoom 50-200 %; el layout no debe romperse ni perder el input |
| **Sesión larga (soak)** | Partida extendida o autoguardado durante N ciclos observando `performance.memory` y listeners: crecimiento continuo = fuga |
| **Online** (si aplica) | Dos contextos de navegador por el relay: rival que se desconecta en mitad de acción, reconexión, mensajes fuera de orden, acciones simultáneas |
| **Fidelidad cruzada** | Cada pantalla contra su ficha `docs/ui/` y la referencia del original; diferencia no documentada = bug de fidelidad |

## Reglas de la caza

- **Un bug = un fichero** `docs/bugs/BUG-xx.md`: repro mínima, seed,
  shots, log de consola, severidad (`bloquea` / `deforma` / `menor`) y
  test que lo cubre. Resumen vivo en `docs/bugs/README.md`.
- **El test llega antes que el fix.** Un arreglo sin regresión volverá
  como bug en la próxima refactorización.
- **La severidad ordena**, no el hallazgo: bloqueantes primero aunque se
  hayan encontrado últimos.
- **El bug puede estar en la spec**: si la interfaz contradice la ficha
  `docs/ui/` y la ficha contradice al original, el bug es de
  especificación → corregir ficha y HU, y re-verificar. La recursión
  también alcanza a las fases anteriores.
- **Nada a ciegas**: lo que no reproduce dos veces es nota de
  investigación, no bug. Lo que reproduce pero no se entiende se acota con
  logging antes de tocar código.
- Todo fix actualiza `AGENTS.md` si descubrió una trampa nueva
  (encoding, orden de eventos, invariante roto).

## Paralelización (opcional)

Las baterías se reparten por **escena/sistema disjunto** con la misma
técnica de la fase 6: una pestaña (`devin desktop chat`) o subagente por
área, compartiendo sólo `docs/bugs/README.md` como tablón. Los **arreglos
nunca en paralelo** sobre ficheros compartidos: la pestaña principal
integra y arregla en serie para no mezclar causas.

## Criterio de salida

- [ ] Barrida completa de todas las baterías aplicables con **cero bugs
      nuevos** — «ya no veo más» no basta, tiene que cerrarse una ronda
      entera limpia.
- [ ] `docs/bugs/README.md` sin bloqueantes abiertos; el resto cerrado con
      test o declarado *known-issue* con motivo.
- [ ] Todo bug cerrado enlaza su test de regresión.
- [ ] Puerta de calidad verde tras el último fix.
- [ ] `AGENTS.md` actualizado con las trampas descubiertas.
