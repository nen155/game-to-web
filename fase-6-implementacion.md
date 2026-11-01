# Fase 6 — Implementación en paralelo con varias pestañas de Devin

**Objetivo:** ejecutar el backlog lo antes posible dividiendo las tareas entre
**pestañas/sesiones de Devin independientes**, sin que se pisen.

## 6.1 Primero la base, solo

Antes de paralelizar, **una sola sesión** implementa lo compartido (P0):

- Scaffold + `AGENTS.md` del clon + puerta de calidad.
- `core/` con el estado del juego, PRNG y serialización.
- El controlador (`game.ts`) y la primera escena jugable de punta a punta
  (en Shandalar: mapa → encuentro → duelo mínimo → vuelta).

Paralelizar sobre una base inexistente produce N implementaciones de lo mismo.
La regla: **no se abre la fase paralela hasta que el bucle mínimo compila y
pasa la puerta de calidad.**

## 6.2 Agrupar el backlog en paquetes disjuntos

Un paquete es asignable a una pestaña cuando cumple:

1. **Superficie de ficheros disjunta** con el resto (o casi).
2. **Sin dependencias de tiempo**: no necesita que otro paquete exista ya.
3. **Criterios autocontenidos**: se puede verificar sin el resto.

Ejemplo real de agrupación (Shandalar):

| Paquete | HUs/Tareas | Ficheros típicos |
|---------|-----------|------------------|
| A. Duelo — visual y cromado | T-26, T-27, T-32, T-41, T-42 | `ui/duel*`, `assets/duel/` |
| B. Duelo — reglas | T-28, T-29, T-30 | `core/duel*`, tests |
| C. Mundo — lugares | T-45, T-51, T-53, T-54 | `ui/settlement*`, `overworld/` |
| D. Mundo — sistema | T-43, T-44, T-48, T-49, T-50 | `overworld/worldMagics.ts`, `adventure.ts` |
| E. Pantallas auxiliares | T-55, T-56, F3/F6 | `ui/` nuevas escenas |

Los ficheros **compartidos** (el controlador, `styles.css`, registro de
escenas, `cards.json`) quedan **bajo propiedad de la pestaña principal**:
los paquetes los *usan* pero no los reorganizan; si necesitan tocarlos, lo
dejan indicado y la integración lo resuelve.

## 6.3 Dos modos de paralelizar

### Modo A — pestañas de Devin (una sesión completa por paquete)

Las pestañas se pueden abrir **automatizadas** con el launcher de Devin
Desktop, que tiene un subcomando `chat` que crea una sesión de chat nueva en
el directorio actual y le envía el prompt:

```bash
# Devin Desktop (Windows)
devin-desktop.cmd chat "<prompt del paquete>"

# Equivalente a través del CLI del agente
devin desktop chat "<prompt del paquete>"
```

Ejemplo de lanzamiento de todos los paquetes desde Git Bash:

```bash
cd "<raiz del clon>"
for p in A B C D E; do
  devin-desktop.cmd chat "$(cat docs/plan-paralelo-$p.md)"
done
```

Cada llamada abre una pestaña de agente independiente en Devin Desktop ya
trabajando — cada pestaña es una sesión completa (contexto propio, sus propios
subagentes si los necesita), no un subagente de la sesión principal.

1. Generar `docs/plan-paralelo.md` y un `docs/plan-paralelo-<paquete>.md` por
   paquete con el prompt ya redactado ([plantilla](plantillas/prompt-pestana.md)).
2. Lanzar un `chat` por paquete (o abrir la pestaña y pegar el prompt a mano).
3. Cada pestaña trabaja, hace commit en su propia rama si el repo es git
   (`feature/t-<xx>`), y reporta.
4. La pestaña principal integra (merge), ejecuta la puerta de calidad y
   actualiza el backlog.

### Modo B — subagentes en segundo plano (automático, sin pestañas)

El skill `/juego-a-web` lanza `run_subagent` con `is_background: true` y
`profile: subagent_general`, uno por paquete disjunto, todos contra el mismo
workspace. Como los prompts acotan ficheros, los conflictos son raros; el
agente principal integra y verifica al completar cada notificación.

### Qué NO paralelizar

- Dos paquetes que toquen el mismo fichero de dominio.
- Cambios en el protocolo de red mientras otro paquete lo consume.
- Renombrados/movimientos de estructura mientras hay paquetes en vuelo.

## 6.4 Contrato de cada paquete

Todo prompt de paquete debe fijar:

- **HUs/T-xx exactos** y sus criterios de aceptación.
- **Ficheros permitidos** y ficheros **prohibidos**.
- Orden de hacer commits y en qué rama.
- Obligación de pasar `typecheck + test + build` antes de reportar.
- Actualizar `BACKLOG.md` solo en su sección (o dejarlo a la integración si
  hay riesgo de conflicto — en la práctica, asignar el backlog a la pestaña
  principal evita casi todas las colisiones).

## 6.5 Integración

Tras cada paquete completado:

1. `git merge` / revisión de diff.
2. Puerta de calidad completa.
3. Marcar estados en `BACKLOG.md` y referencias HU.
4. Si un paquete dejó «notas de integración» (toques a ficheros
   compartidos), aplicarlos ahora.

## Criterio de salida

- [ ] `plan-paralelo.md` con paquetes disjuntos y prompts listos.
- [ ] Cada paquete pasó la puerta de calidad antes del merge.
- [ ] `BACKLOG.md` refleja el estado real tras integrar.
