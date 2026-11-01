# Fase 10 — Cobertura de modelos: todos los personajes cargan

**Objetivo:** que **todos** los personajes y entidades del juego carguen su
modelo (malla, sprite, animación base — lo que use el juego) a través del
pipeline real del clon, no sólo los que aparecen en el contenido ya
implementado. El fallo típico de un port es que el loader sólo se ha
probado con los assets del primer nivel: esta fase recorre el **roster
completo** y prueba, uno a uno, que cada modelo decodifica, tiene
geometría válida y produce píxeles al renderizar.

## De dónde sale el roster

La lista de personajes no se escribe de memoria ni del contenido jugado:
se construye desde la extracción y no se cierra hasta agotar las fuentes:

- `asset_manifest.json` / catálogo de la fase 1 (todos los ficheros de
  personaje: modelos, retratos, animaciones).
- Catálogos de la comunidad o generados en la fase 2 (tabla de
  personajes/enemigos del decompilado).
- Referencias en scripts/datos de niveles (cada `spawn`, cada NPC
  nombrado, cada enemigo instanciado en datos).
- Lo descubierto jugando: un personaje que aparece en una escena y no
  está en el roster **se encola** — la cola puede crecer; eso es la
  recursión.

## La cola

`docs/cobertura/modelos.md`: una fila por entidad —

| Entidad | Asset fuente | Loader | Estado |
|---------|--------------|--------|--------|
| david | `chars/david.CHR` | `chr2web` | `pendiente / carga / descartado(motivo)` |

Estados: `pendiente`, `carga` (pasa el test completo), `descartado(motivo)`
(sólo para assets huérfanos que el juego nunca instancia — documentar por
qué). El roster es la fuente de verdad del avance, no la memoria.

## El test de carga (obligatorio y automático)

Un spec de Vitest — p. ej. `tests/modelos.test.ts` — itera **todo el
roster** y por cada entidad:

1. **Decodifica** el asset por el loader real (el mismo código que usa el
   juego, no un camino paralelo de test).
2. **Valida la estructura**: geometría no vacía (vértices/caras/frames > 0),
   textura o paleta presente, bounding box finita, sin `NaN`.
3. **Renderiza**: al menos un frame por el renderer (software en Node, o
   canvas headless) y afirma que produce píxeles no transparentes —
   cargar «sin error» un modelo invisible no cuenta.

Para que esto sea posible, el loader/renderer deben estar **desacoplados
del DOM**: la capa de decodificación es core puro y el render acepta un
destino abstracto. Si el renderer actual exige navegador, separarlo es
parte de esta fase.

## El bucle

```
┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
│ 1. Siguien-│──▶│ 2. Pasarla │──▶│ 3. ¿Carga? │──▶│ 4. Marcar  │
│ te entidad │   │ por el test│   │            │   │ en la cola │
│ del roster │   │ de carga   │   │            │   │            │
└────────────┘   └────────────┘   │  sí → ok   │   └────────────┘
       ▲                          │  no → BUG  │
       │ entidad nueva            └─────┬──────┘
       │ descubierta                    │ repro + test rojo + fix
       └────────────────────────────────┘

Roster sin pendientes → SALIR
```

1. Siguiente entidad pendiente del roster.
2. Ejecutar el test de carga sobre ella (al añadir el spec, todo el roster
   corre de golpe: el «siguiente» suele ser depurar el primer fallo).
3. Si falla: **no se salta el personaje** — se abre `docs/bugs/BUG-xx.md`
   con la traza (offset, chunk, excepción), se arregla el
   decodificador/exportador y el test pasa a verde. Si el asset está
   corrupto también en el original, `descartado(motivo)`.
4. Marcar `carga` en la cola y commit (cola + fix + test en el mismo
   commit).
5. Repetir hasta roster vacío.

## Reglas

- **Siempre todos los personajes.** El roster manda: un personaje sin
  probar es un pendiente, aunque nadie lo haya visto aún en el clon.
- **Fallback ruidoso, nunca mudo.** Si en runtime un modelo falla, en dev
  se ve y se oye (error en consola + placeholder evidente); un placeholder
  silencioso esconde el pendiente. El fallback es red de seguridad, no
  solución.
- **El bug es del decoder, no del personaje.** Un modelo que no carga se
  arregla en el extractor/loader con su test de regresión — mismo
  protocolo que la fase 9.
- **Hoja de contacto** (igual que los sprites de la fase 1): generar un
  PNG/contact-sheet o galería `?debug=models` con todos los personajes
  renderizados; es la evidencia visual del 100 % y detecta de un vistazo
  texturas cambiadas o paletas equivocadas.
- Entidades que comparten modelo (paletas/variantes) cuentan como filas
  distintas si el juego las trata distinto.

## Criterio de salida

- [ ] `docs/cobertura/modelos.md` sin `pendiente`: toda entidad en
      `carga` o `descartado(motivo)`.
- [ ] `tests/modelos.test.ts` (o equivalente) verde iterando el roster
      completo en CI local.
- [ ] Hoja de contacto/galería con todos los modelos renderizados,
      enlazada desde la cola.
- [ ] Puerta de calidad verde y continuar a la **fase 11 — animaciones**.
