# Fase 2 — Obtención del código (decompilación)

**Objetivo:** conseguir una representación legible de la lógica del juego —
**reglas, fórmulas y flujos** que no están en los datos ni en los textos.
La forma exacta depende de la plataforma decidida en la fase 0.0; lo que no
cambia es el objetivo ni el destino de la salida (`decompiled/code/`).

## 2.0 Elegir la rama

| Situación | Herramienta | Calidad de la salida |
|-----------|-------------|----------------------|
| **Código fuente disponible** (repo, fuentes en el paquete) | — | Perfecto: leer directamente, fase reducida a mapear módulos |
| **.NET / Unity (Mono)** `.dll` gestionadas | ILSpy, dnSpy | Casi fuente original (nombres reales, estructuras) |
| **Unity il2cpp** (`GameAssembly.dll` + `global-metadata.dat`) | Il2CppDumper → Ghidra | Pseudocódigo **con nombres** (mucho mejor que nativo pelado) |
| **JVM / Android** (`.jar`, `.class`, `.dex`, `.apk`) | CFR / Fernflower, jadx | Casi fuente original |
| **Nativo C/C++** (`.exe`, `.dll`, `.so`, ROM) | **Ghidra headless** | Pseudocódigo con nombres genéricos |
| **Flash** (`.swf`) | JPEXS | ActionScript casi fuente |
| **Python empaquetado** (`.pyc`, PyInstaller) | pycdc / decompyle3 | Casi fuente |
| **Electron/JS** (`.asar`, `bundle.js`) | `npx asar extract` + beautifier / source maps | A menudo fuente completa |
| **Motores con scripts propios** (Ren'Py, GameMaker, RPG Maker) | unrpyc, UndertaleModTool, decrypters | Guión/código del juego recuperable |
| **Protegido / imposible** (cifrado fuerte, VMProtect, plataforma cerrada) | — | Sin código: documentar el hueco y compensar con fases 3 y 5 |

Regla: **gestionado > nativo**. Un juego .NET/JVM decompila a algo casi
idéntico al fuente; un nativo sólo da pseudocódigo. Si hay duda de qué es un
binario, `Detect It Easy` o el propio import de Ghidra lo dicen.

## 2.1 Rama nativa — Ghidra headless

Requisitos: JDK 17+ y Ghidra en `tools/`.

```bash
# Patrón usado en decompiled/tools/run_ghidra.sh
$GHIDRA/support/analyzeHeadless \
    <proyecto_dir> <nombre_proyecto> \
    -import <binario> \
    -postScript DecompileAll.java <dir_salida> \
    -deleteProject
```

`DecompileAll.java` (script de Ghidra) recorre todas las funciones tras el
auto-análisis y vuelca, por binario:

- `<nombre>.c` — pseudocódigo por función
- `<nombre>.asm` — desensamblado
- `<nombre>.index.txt` — índice dirección → función
- `decompile.log` — registro del proceso

Analizar **todos** los ejecutables, también los pequeños: un DLL de 25 KB
puede ser justo donde vive la regla que buscas.

## 2.2 Rama gestionada — decompiladores reales

- **.NET/Unity-Mono**: ILSpy en línea de comandos (`ilspycmd -p -o out
  Assembly-CSharp.dll`) genera un proyecto C# navegable. Es la mejor salida
  posible: nombres de métodos, strings y estructuras originales.
- **Unity il2cpp**: `Il2CppDumper` extrae los metadatos → se genera un
  proyecto Ghidra/script que nombra las funciones; luego Ghidra como en la
  rama nativa pero con símbolos reales.
- **JVM/Android**: `jadx -d out app.apk` o CFR por `.jar`. Recursos de la APK
  con `apktool` (fase 1).
- La salida ya es «código»: tratarla como un repo de solo lectura dentro de
  `decompiled/code/` y citar clases/métodos en las HUs.

## 2.3 Cómo leer la salida

### Pseudocódigo de Ghidra (nativo)

Nombres genéricos (`FUN_004a95e0`, `param_1`, `DAT_00583a28`). No es un
obstáculo si se busca lo correcto:

| Qué buscar | Qué revela | Ejemplo real |
|------------|-----------|--------------|
| **Constantes y tablas** (`DAT_…`, arrays) | Rejillas, tamaños, enumeraciones | `DAT_00583a28 * 5 + fase` → sprites de 8 direcciones × 5 fotos |
| **Fórmulas aritméticas** | Costes, probabilidades, dispersión | `abs(y)*3 + abs(x)*7` → dispersión de decoración del mapa |
| **Llamadas a strings** (`@TAG`, `.rodata`) | Qué función pinta qué pantalla/menú | cruzar los strings volcados con las x-refs |
| **Máquinas de estado** (switch de estado) | Flujos de pantallas, fases de turno | la tira de fases del duelo |
| **X-refs entre funciones** | Quién llama a qué | la embestida de monstruos y su condición de terreno |

### Código gestionado/fuente

Búsqueda directa por nombre: `grep` sobre la salida con el texto de la opción
de menú o el nombre de la clase (`class CombatManager`). Todo es más fácil —
aprovecharlo para extraer reglas en bloque, no sólo bajo demanda.

## Flujo por *feature* (cualquier rama)

1. Localizar la cadena/símbolo relacionado (string de menú, nombre de clase).
2. Encontrar la función que la usa (x-ref en Ghidra, grep en código).
3. Leer esa función: ahí están condiciones, umbrales y bucles exactos.
4. Anotar el hallazgo en el `AGENTS.md` del proyecto — los valores «mágicos»
   descifrados son la documentación más valiosa del port.

## Expectativas realistas

- **Ratio parcial en nativo**: Ghidra no delimita funciones en código
  empaquetado o con saltos indirectos. Registrar el ratio por binario
  (Shandalar: 2233/2595 en el exe principal).
- **Binarios empaquetados/protegidos**: sólo se ve el stub. Anotarlo y seguir.
- **El pseudocódigo miente a veces**: convenciones de llamada mal inferidas o
  código muerto. Si una fórmula no cuadra, bajar al `.asm` de esa función.
- **Sin código ≠ sin port**: si nada es decompilable, la especificación sale
  de datos + strings + referencias externas (fases 3 y 5) y el hueco se
  documenta como limitación. El proceso no se detiene.

## Cuándo parar

No hay que «entender todo». Se obtiene la salida una vez y se consulta
**bajo demanda** durante las fases 3 y 6: cada HU que necesite una regla
exacta dispara una búsqueda puntual en `decompiled/code/`.

## Entregables

```
decompiled/code/            (o ghidra/out/ en la rama nativa)
├── <binario>.c / .asm / .index.txt    (nativo)
├── <assembly>/ ... .cs                (gestionado)
└── decompile.log + README de la rama usada
```

- Tabla binario → rol → cobertura (en `decompiled/README.md`).
- `FORMATS.md` actualizado con lo deducido del código (layouts, rejillas).

## Criterio de salida

- [ ] Rama elegida y documentada (plataforma + herramienta + por qué).
- [ ] Todo binario/script relevante procesado, o marcado como imposible con
  motivo y plan de compensación.
- [ ] Índice navegable (función/clase ↔ posición).
- [ ] Primeros hallazgos (rejillas, constantes de reglas) anotados para las
  fases siguientes.
