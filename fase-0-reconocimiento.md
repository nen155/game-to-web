# Fase 0 — Reconocimiento del paquete

**Objetivo:** saber qué hay delante antes de tocar nada: **qué es el juego**
(plataforma, motor, lenguaje) y qué es cada fichero (código, datos, media,
texto). De aquí sale la cadena de herramientas de las fases 1-2 y la primera
lista de *features* candidatas.

## Entradas

Cualquier distribución del juego: carpeta de instalación, APK, ROM, imagen de
disco (ISO, BIN/CUE, NRG, CHD, DMG…), instalador (MSI, Inno Setup, NSIS,
InstallShield…), bundle web, repositorio con fuentes… Las imágenes y los
instaladores se materializan primero a ficheros — **nunca se instala el
juego** — siguiendo el paso previo.

## Paso previo — Materializar imágenes e instaladores

Cuando la entrada es una imagen de disco o un paquete de instalación (y no
una carpeta ya desplegada), el primer trabajo es convertirla a un árbol de
ficheros en `<juego>/extracted/`. Regla de oro: **extracción en sólo lectura,
sin ejecutar nada** del paquete.

### Detectar el formato

La extensión miente a menudo (`.bin`, `.img`, `.dat` sirven para todo):
confirmar con la magia de los primeros/últimos bytes y con los ficheros
hermanos.

| Pista | Formato probable |
|-------|------------------|
| `CD001` en offset 0x8001/0x8801/0x9001 | ISO 9660 (CD-ROM) |
| `NSR02`/`NSR03`/`BEA01` | UDF (DVD/BD) |
| `NERO` cerca del final | NRG (Nero) |
| `MComprHD` | CHD (MAME/emuladores) |
| `koly` en los últimos 512 B | DMG (macOS) |
| `.cue`/`.ccd`/`.mds` hermano | imagen de CD en pares (posible **modo mixto**) |
| OLE2 (`D0CF11E0`) + extensión `.msi` | Windows Installer |
| strings `Inno Setup`, `Nullsoft`, `InstallShield` en el `.exe` | instalador conocido |

### Extraer según el formato

| Entrada | Herramienta |
|---------|-------------|
| `.iso`, `.nrg`, `.dmg`, `.img`, `.vhd`, UDF | `7z x imagen -oextracted` (7-Zip abre casi todo esto) |
| `.bin` + `.cue` | `bchunk` (pista de datos → ISO; pistas de audio → WAV) |
| `.mdf`/`.mds`, `.cdi`, `.ccd`, `.uif` | convertir a ISO (`mdf2iso`, `cdi2iso`, `iat`, `uif2iso`) o `7z` |
| `.chd` | `chdman extracthd` (o `extractcd` para CD) |
| Inno Setup (`setup.exe`) | `innoextract` |
| NSIS (`install.exe`) | `7z x` (el payload suele quedar listo; mirar `$PLUGINSDIR`) |
| `.msi` | `lessmsi x`, o `msiexec /a setup.msi /qn TARGETDIR=...` (instalación administrativa: despliega ficheros sin instalar) |
| `.cab` InstallShield | `i6comp`/`i5comp` |
| `.pkg`/`.xip` (macOS) | `pkgutil --expand-full` |
| Nada lo abre | último recurso: montar sólo lectura (`Mount-DiskImage` necesita admin; `WinCDEmu`/`imdisk` menos) y copiar; desmontar al terminar |

### Reglas

- **Multi-disco** (CD1, CD2…): una subcarpeta por imagen en `extracted/` y
  tratar el conjunto como un único árbol lógico. Anotar qué vino de cada
  disco: algunos juegos comprueban la etiqueta del CD y hay ficheros cuya
  versión cambia entre discos.
- **CD modo mixto**: las pistas de audio se ripean a `extracted/audio/*.wav`
  — suelen ser la banda sonora y pasan por la fase 1 como cualquier WAV.
- **Protección anticopia** (SecuROM, SafeDisc…): no impide leer los ficheros;
  a lo sumo el `.exe` se niega a *ejecutarse*, lo cual es irrelevante aquí.
  Anotarla en `FORMATS.md` como limitación.
- Registrar formato, herramienta y resultado en `decompiled/FORMATS.md`.
  Desde ese momento `extracted/` se trata exactamente igual que una carpeta
  de instalación.

## 0.0 Clasificar el juego — decide todo lo demás

Antes del inventario de ficheros, responder:

| Pregunta | Cómo se detecta | Consecuencia |
|----------|-----------------|--------------|
| **¿Hay código fuente?** | repo git, `.cs/.java/.gd/.cpp` sueltos, `src/` | **Salta la fase 2**: se lee el código directamente |
| **¿Qué plataforma?** | `.exe/.dll` nativos → Windows/DOS; `.apk` → Android; `bundle.js`/`.asar` → web/Electron; `.swf` → Flash; `.app/.ipa` → macOS/iOS; ROM → consola | Herramienta de fase 2 |
| **¿Qué motor?** | Unity: `*_Data/`, `UnityPlayer.dll`, `.assets/.bundle`; Unreal: `.pak/.uasset`, `Engine/`; Godot: `.pck`; Ren'Py: `.rpa/.rpyc`; GameMaker: `data.win`; RPG Maker: `.rgss*` | Extractor de assets comunitario específico (fase 1) |
| **¿Nativo o gestionado?** | `.dll`/.exe con metadatos .NET (se ve en Ghidra/`Detect It Easy`); `.jar/.class/.dex` → JVM | Gestionado → decompilador real (código casi fuente); nativo → Ghidra (pseudocódigo) |
| **¿Está protegido?** | il2cpp, Denuvo/VMProtect, assets cifrados, ofuscación | Marcar como limitación; compensar con datos + strings + referencias externas |

Esta clasificación se escribe al principio de `decompiled/FORMATS.md` — es la
decisión de la que cuelgan las fases siguientes.

## Pasos

### 0.1 Inventario exhaustivo

Listar todos los ficheros agrupados por extensión, con tamaños:

```bash
# Linux/WSL/Git Bash
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn
# PowerShell
Get-ChildItem -Recurse -File | Group-Object Extension |
  Sort-Object Count -Descending | Format-Table Count, Name
```

Clasificar cada extensión/carpeta en una tabla:

| Extensión / carpeta | ¿Qué es? | ¿Cómo se abre? | Prioridad |
|---------------------|----------|----------------|-----------|
| `.exe`, `.dll` | Código | Ghidra (fase 2) | Alta |
| `.dat`, `.bin` | Datos estructurados | Análisis binario | Alta |
| `.csv`, `.txt`, `.ini` | Datos/texto plano | Lectura directa | **Muy alta** |
| `.pic`, `.spr`, `.bmp` | Gráficos | Decodificador propio/comunitario | Media |
| `.wav`, `.avi`, `.smk` | Media | Copia / ffmpeg | Baja |
| `.tr`, `.pal` | Paletas | Necesarias para los gráficos | Alta |

**El texto plano manda.** Los `.txt`/`.csv`/`.ini`/ficheros de strings del
juego (`Uistrings.txt`, `Advstrings.txt`, `Readme.txt`, manuales en la
carpeta) son la forma más barata de descubrir *features*: cada cadena de menú
es una funcionalidad; cada `@TAG` de diálogo, un evento. Leerlos **antes** de
decompilar orienta toda la búsqueda posterior.

### 0.2 Identificar formatos propietarios

Para cada extensión desconocida, mirar los primeros bytes (cabeceras mágicas)
y estructura:

```bash
xxd fichero.dat | head -20
strings -n 6 fichero.dat | head -50   # a veces el formato se delata solo
```

Reglas prácticas:

- **Los contenedores de motor van primero**: si el juego es Unity/Unreal/
  Godot/etc., casi todo vive en sus bundles (`.assets`, `.pak`, `.pck`…). No
  se inspeccionan byte a byte: se usa el extractor del motor (fase 1).
- Tamaño constante por registro → `(tamaño − cabecera) / nº de entidades`
  suele dar el tamaño del registro y el *layout* se deduce comparando campos
  con un equivalente en texto (así se descifró `Cards.dat` contra
  `Manalink.csv`).
- **Buscar primero, implementar después**: la mayoría de formatos propietarios
  ya están documentados o tienen herramientas de comunidad (`github.com`,
  `wiki.*`, foros de modding, QuickBMS scripts). Los `.pic` de Shandalar eran
  el *PICv3* de Darklands y se reutilizó `mp_pic_tools` en vez de reescribir
  el decodificador.
- Anotar endianness, compresión habitual (RLE, LZW, zlib, LZ4, o custom) y si
  las imágenes llevan paleta embebida o externa (`.tr`, `.pal`).

### 0.3 Mapa de ejecutables

Cada `.exe`/`.dll` a la tabla con su rol deducido por nombre, tamaño, imports
y strings:

| Binario | Rol (deducido) | ¿Empaquetado? |
|---------|----------------|---------------|
| `Shandalar.exe` | Modo aventura | No |
| `Magic.exe` | Motor de duelos | No |
| `CardArtLib.dll` | Arte de cartas | **Sí** (sólo se ve el stub) |

Los empaquetados se anotan y se siguen sin ellos: sus datos suelen ser
accesibles por otra vía (ficheros sueltos) o no aportan lógica jugable.

### 0.4 Primeras fuentes externas

Recoger (sin profundizar aún, eso es fase 5):

- Manual original (archive.org suele tenerlo escaneado).
- Readme / erratas dentro del paquete.
- Artículo de Wikipedia / MobyGames del juego.
- Comunidades de modding del juego concreto.

## Entregables

- `decompiled/FORMATS.md`: clasificación del juego (plataforma/motor/
  lenguaje/protección), formatos identificados, estado («decodificado»,
  «pendiente», «irrelevante») y herramienta usada.
- Tabla de binarios con rol.
- Lista preliminar de *features* extraída de los ficheros de texto.
- `decompiled/tools/` con los scripts iniciales de inspección.

## Trampas

- **Encoding**: los CSV/TXT de juegos antiguos suelen ser **cp1252/latin-1**, no
  UTF-8. Leerlos mal corrompe tildes y nombres silenciosamente.
- **Ficheros que parecen código y no lo son**: en Shandalar,
  `Editor/ManalinkEx.dpr` (2,6 MB) era relleno `DB '…'` (*code cave*), no
  fuente Pascal real. Comprobar siempre antes de ilusionarse.
- **Nombres ambiguos en Windows**: el sistema de ficheros insensible a
  mayúsculas hace que `CARDBK_Red.pic` y `CARDBK_Red.bmp` puedan pisar el
  mismo `CARDBK_Red.png` al convertir. Registrar en el manifest qué origen
  generó cada salida.
