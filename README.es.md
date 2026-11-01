# Juego-a-Web — pipeline genérico para clonar un juego en el navegador

Documentación del proceso completo para llevar **cualquier juego** a un clon
jugable en navegador que **reutiliza los datos, el arte y los sonidos del
original** siempre que sea posible.

Está generalizado a partir del trabajo hecho con *Magic: The Gathering –
Shandalar* (MicroProse 1997, nativo Windows), pero el procedimiento cubre
cualquier tipo de juego: binarios nativos (Windows/DOS/Linux), código
gestionado (.NET/Unity, JVM/Android), juegos web/Flash ya empaquetados,
móviles (APK), **imágenes de disco** (ISO, BIN/CUE, NRG, CHD, DMG…) e
**instaladores sin instalar** (MSI, Inno Setup, NSIS, InstallShield…) — el
paso previo de la fase 0 los materializa a ficheros en sólo lectura — e
incluso proyectos con **código fuente disponible** — en ese caso la fase de
decompilación se sustituye por lectura directa.

> **El pipeline es una plantilla, no un guion rígido.** La fase 0 decide qué
> ramas aplican según la plataforma, el motor y el lenguaje del juego; cada
> fase documenta sus alternativas. Si algo no aplica o una herramienta no
> existe para ese juego, el proceso se adapta (herramienta equivalente,
> extracción parcial, o reconstrucción desde referencias externas) en lugar de
> detenerse. Toda desviación se anota en el `AGENTS.md` del proyecto.

## Uso rápido

El proceso está automatizado con el skill global de Devin:

```
/juego-a-web <ruta del juego o de su imagen/instalador>
```

Vale una carpeta con el juego instalado, una imagen de disco (ISO, BIN/CUE,
NRG, CHD…), un instalador (MSI, Inno Setup, NSIS…), un APK, una ROM o un
repositorio con fuentes. Si la entrada es imagen/instalador se extrae a
`<juego>/extracted/` sin instalar nada y el pipeline sigue sobre ese árbol.

El skill ejecuta las trece fases de forma autónoma. Esta carpeta es la
referencia detallada de cada fase: léela para entender qué hace el skill,
ajustar decisiones o ejecutar el proceso a mano.

## El pipeline

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ 0. Recono-  │──▶│ 1. Extrac-  │──▶│ 2. Decompi- │──▶│ 3. Historias│
│   cimiento  │   │   ción      │   │   lación    │   │   de usuario│
└─────────────┘   └─────────────┘   └─────────────┘   └──────┬──────┘
                                                            │
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌──────▼──────┐
│ 7. Verifi-  │◀──│ 6. Implemen-│◀──│ 5. Perfilado│◀──│ 4. Stack    │
│   cación    │   │   tación    │   │   de la UI  │   │   tecnol.   │
│ (Playwright)│   │  (paralela) │   │             │   │             │
└──────┬──────┘   └─────────────┘   └─────────────┘   └─────────────┘
       │
┌──────▼──────┐   ⟲ bucle recursivo:
│ 8. Cobertura│     elemento → HU → implementar → verificar →
│   recursiva │     siguiente; los sub-elementos descubiertos se
│             │     encolan; sale cuando la cola queda vacía
└──────┬──────┘
       │
┌──────▼──────┐   ⟲ bucle recursivo:
│ 9. Caza de  │     barrer → confirmar → repro mínima → test → fix →
│   bugs      │     puerta de calidad → re-barrer hasta una barrida
│ (interfaz)  │     completa sin bugs nuevos
└──────┬──────┘
       │
┌──────▼──────┐   ⟲ cola por personaje:
│ 10. Modelos │     cada entidad del roster pasa por el loader real;
│   por       │     spec que itera todo el roster + hoja de contacto;
│   personaje │     el que falla es BUG, nunca se salta
└──────┬──────┘
       │
┌──────▼──────┐   ⟲ matriz personaje × animación:
│ 11. Anima-  │     andar siempre + cada acción del original;
│   ciones y  │     prueba de movimiento por la interfaz con cada
│   acciones  │     personaje
└──────┬──────┘
       │
┌──────▼──────┐   ⟲ cola de HUDs:
│ 12. HUDs    │     comparar lado a lado con la ficha y el original;
│   fieles    │     estados y variantes se encolan; diferencia =
│ (recursivo) │     bug de fidelidad o bug de spec
└─────────────┘
```

| Fase | Documento | Entregable |
|------|-----------|------------|
| 0 | [fase-0-reconocimiento.md](fase-0-reconocimiento.md) | Inventario del paquete y de los formatos |
| 1 | [fase-1-extraccion.md](fase-1-extraccion.md) | `decompiled/assets/` + `decompiled/data/` verificados |
| 2 | [fase-2-decompilacion.md](fase-2-decompilacion.md) | `decompiled/code/` (o `ghidra/out/`) + mapa binario→rol |
| 3 | [fase-3-historias.md](fase-3-historias.md) | `docs/historias-usuario/` + matriz de cobertura + backlog |
| 4 | [fase-4-stack.md](fase-4-stack.md) | Scaffold `web-game/` (o similar) con toolchain |
| 5 | [fase-5-interfaz.md](fase-5-interfaz.md) | Fichas de pantalla + referencias visuales externas |
| 6 | [fase-6-implementacion.md](fase-6-implementacion.md) | Paquetes de HUs paralelizables + prompts por pestaña |
| 7 | [fase-7-verificacion.md](fase-7-verificacion.md) | Informe Playwright por HU + evidencias |
| 8 | [fase-8-cobertura-recursiva.md](fase-8-cobertura-recursiva.md) | `docs/cobertura/` con todos los elementos cubiertos (HU → implementación) |
| 9 | [fase-9-bugs.md](fase-9-bugs.md) | `docs/bugs/` con repro mínima + tests de regresión |
| 10 | [fase-10-modelos.md](fase-10-modelos.md) | `docs/cobertura/modelos.md`: roster completo de personajes cargando + test |
| 11 | [fase-11-animaciones.md](fase-11-animaciones.md) | `docs/cobertura/animaciones.md`: matriz personaje × animación + pruebas de movimiento |
| 12 | [fase-12-huds.md](fase-12-huds.md) | `docs/cobertura/huds.md`: todos los HUDs conformes al diseño |

Las fases 0-2 son el **pipeline de ingeniería inversa**; las 3-5 convierten lo
encontrado en **especificación**; las 6-9 son la **construcción y el control
de calidad**. La fase 8 tiene dos usos: como **barrido de cierre** tras el
pipeline lineal (lo que quedó sin HU se especifica e implementa allí) o como
**motor principal** en modo recursivo, donde sustituye la secuencia 3-6 y
cada historia se implementa en cuanto se crea. En ambos casos termina con la
cobertura total de los elementos del juego y da paso a la caza de bugs.

Las fases 10-12 son **barridos de cobertura especializados** que cierran los
tres frentes donde un port suele quedar cojo aunque «funcione»: los assets de
los personajes que el contenido jugado aún no tocaba (10), las animaciones y
acciones de cada uno (11) y la fidelidad visual de todos los HUDs frente al
diseño (12). Los defectos que encuentran entran por el mismo protocolo de la
fase 9: repro, `BUG-xx`, test de regresión y puerta de calidad.

## Estructura de directorios que produce

```
<juego>/
├── extracted/            # sólo si la entrada era imagen/instalador:
│                         # su contenido materializado, en sólo lectura
├── decompiled/           # todo lo generado por ingeniería inversa
│   ├── FORMATS.md        # formatos de archivo documentados
│   ├── assets/           # imágenes/audio/vídeo convertidos a formatos web
│   ├── data/             # datos normalizados a JSON + manifest
│   ├── code/             # salida de la decompilación (herramienta según plataforma)
│   └── tools/            # scripts del pipeline (reutilizables)
└── <web-game>/           # el clon
    ├── docs/
    │   ├── historias-usuario/   # 00-visión + HUs con criterios de aceptación
    │   ├── ui/                  # fichas de pantalla (fase 5)
    │   ├── verificacion/        # informes Playwright + capturas (fase 7)
    │   ├── cobertura/           # cola de elementos (fase 8) + rosters de
    │   │                        # modelos, animaciones y HUDs (fases 10-12)
    │   ├── bugs/                # informes BUG-xx + shots (fase 9)
    │   └── BACKLOG.md           # tareas T-xx ↔ HU-xx con estado
    ├── public/assets/    # assets del original servidos al navegador
    └── src/
        ├── core/         # dominio puro, testeable en Node, sin DOM
        └── ui/           # escenas / render
```

## Principios rectores

1. **El original es la fuente de verdad.** Datos, textos, assets y decompilado
   mandan sobre recuerdos, wikis o vídeos. Las fuentes externas sólo rellenan
   lo que el código no revela (ritmos, transiciones, *feel*).
2. **Nada se inventa sin verificar.** Todo volcado binario se valida contra
   una referencia en texto plano (CSV, strings) campo a campo antes de usarse.
3. **Trazabilidad total.** Cada historia de usuario cita su fuente y cada
   característica encontrada termina en una HU o se descarta con motivo.
4. **Dominio puro y determinista.** La lógica del juego vive en un núcleo sin
   DOM, con toda la aleatoriedad saliendo de un PRNG con semilla: partidas
   reproducibles y testeables en Node.
5. **Automatizable.** Cada paso es un script o un comando; los criterios de
   aceptación son comprobables (tests unitarios + Playwright).
6. **Adaptable por plataforma.** No se asume «juego de Windows de los 90»:
   primero se clasifica el juego (plataforma, motor, lenguaje, formato de
   distribución) y se elige la cadena de herramientas adecuada. Si el código
   no se puede extraer (cifrado, VM protectora, plataforma cerrada), el
   proceso continúa con datos + strings + referencias externas y marca el
   hueco como limitación documentada — nunca se para a esperar.

## Plantillas

En [`plantillas/`](plantillas/):

- [`historia-usuario.md`](plantillas/historia-usuario.md) — formato de HU y
  fichero de historias.
- [`matriz-cobertura.md`](plantillas/matriz-cobertura.md) — features ↔ HUs.
- [`prompt-pestana.md`](plantillas/prompt-pestana.md) — prompt listo para
  pegar en cada pestaña de Devin en la implementación paralela.
- [`informe-playwright.md`](plantillas/informe-playwright.md) — informe de
  verificación por historia.
- [`informe-bug.md`](plantillas/informe-bug.md) — informe de bug de la caza
  recursiva.

## Caso de estudio

Todo el documento usa Shandalar como ejemplo real: `E:\Magic
Shandalar\decompiled\` (extracción y decompilado) y `E:\Magic
Shandalar\browser-game\` (el clon, TypeScript + Vite + Canvas).
Sus `AGENTS.md` son un buen modelo de cómo registrar las trampas descubiertas
durante el proceso.
