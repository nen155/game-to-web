# Fase 1 — Extracción de assets y datos

**Objetivo:** convertir todo lo reutilizable del paquete original a formatos
web (PNG/JPG/WAV/JSON) en `decompiled/`, con **verificación** de que nada se
ha perdido ni interpretado mal.

## Principios

- **Scriptable**: cada conversión es un script en `decompiled/tools/`, con
  `--game` (entrada) y `--out` (salida) como parámetros. Nada de pasos manuales
  irreproducibles.
- **Conservador**: copiar también los originales cuando la conversión es con
  pérdida (p. ej. mantener los `.JPG` de escaneos de carta además de los PNG).
- **Verificable**: todo volcado de datos binarios se cruza contra una
  referencia en texto plano y se exige ~100 % de coincidencia antes de
  confiar en él.

## 1.0 Elegir el extractor según el contenedor

La clasificación de la fase 0.0 decide la herramienta. Referencia rápida:

| Contenedor / motor | Herramienta típica | Resultado |
|--------------------|--------------------|-----------|
| Ficheros sueltos propietarios (`.pic`, `.spr`, `.dat`…) | Decodificador de comunidad o script propio | PNG/JSON |
| **Unity** (`*_Data/`, `.assets`, `.bundle`) | AssetRipper, AssetStudio | Texturas, mallas, `MonoBehaviour` serializados |
| **Unreal** (`.pak`, `.uasset`) | FModel, UnrealPak, QuickBMS | Assets + tablas de datos |
| **Godot** (`.pck`) | gdsdecomp / GDRE Tools | Escenas y scripts `.gd` recuperables |
| **Ren'Py** (`.rpa`, `.rpyc`) | rpatool + unrpyc | Assets + **guión fuente** |
| **GameMaker** (`data.win`) | UndertaleModTool | Sprites + código GML |
| **RPG Maker** (`.rgss*`) | Decrypter de comunidad | Recursos + eventos (que son la lógica) |
| **Flash** (`.swf`) | JPEXS Free Flash Decompiler | Sprites + ActionScript |
| **Electron/web** (`.asar`, bundles) | `npx asar extract`, beautifier | **Código fuente casi completo** |
| **Android** (`.apk`) | apktool, jadx | Recursos + smali/Java |
| **ROM consola** | herramientas del emulador/scene | tiles, mapas, código ASM |

Casos especiales:

- **Assets cifrados**: buscar el descifrador de comunidad; si no existe, se
  marca la limitación y se continúa con lo extraíble + referencias externas.
- **El juego ya trae los assets en formato web** (PNG/OGG/JSON): copia
  directa, la fase se reduce a inventariar.

## 1.1 Gráficos

Orden recomendado:

1. Localizar las paletas (`.tr`, `.pal`) — una imagen indexada sin su
   paleta correcta es ruido de colores. **Trampa real**: un `.pic` sin paleta
   embebida (`Cardback.pic`) convertido con la paleta equivocada parece
   «roto»; el `.bmp` hermano estaba limpio. Cuando un asset salga corrupto,
   probar primero otra paleta/otro original antes de tocar el decodificador.
2. Decodificar con herramientas de la comunidad cuando existan
   (en Shandalar: `mppic/` para `.pic`/`.spr`).
3. Generar además **hojas de contacto** (contact sheets) con el índice de cada
   fotograma dibujado: son la forma rápida de identificar «qué fotograma es
   cuál» sin leer código.
4. Escribir `data/asset_manifest.json`: origen → salida, tamaño, nº de
   fotogramas, errores. El manifest es lo que permite decir «3746 ficheros,
   0 fallos» en lugar de «parece que salió todo».

### Sprites animados / hojas

Los contenedores de sprites suelen tener rejillas implícitas que el juego
conoce por convención (columnas = dirección, filas = fase). Documentar la
rejilla al extraer (en Shandalar: `ego_*`/`m_*` son 8 direcciones × 5 fotos;
`land` son 5 columnas por región). Si se reempaquetan en atlas para la web,
calcular y guardar **`anchorX`/`anchorY`** (caja del contenido, no del
fotograma) — sin anclas los sprites «flotan».

## 1.2 Audio y vídeo

- `.wav`/`.mp3`/`midi`: copia directa.
- `.avi`/`.smk`: convertir con `ffmpeg` a lo que el uso dicte; para secuencias
  cortas tipo animación, el filtro `tile` genera hojas de fotogramas
  (`ffmpeg -i in.avi -vf "fps=15,scale=320:240,tile=8x8" out.jpg`).

## 1.3 Datos estructurados → JSON

1. Si hay CSV/tablas en texto: normalizar a `data/<dominio>.json`
   (el catálogo que consumirá el juego web).
2. Si hay binarios: deducir el *layout* (ver fase 0.2), volcar a JSON y
   **verificar campo a campo** contra el CSV/texto equivalente:
   `parse_cards_dat.py --verify` comparó 14 campos × 1617 registros.
   Un volcado sin verificar es una fuente de bugs silenciosos.
3. Exportar tablas de decodificación (enums de color, tipo, rareza…) a
   `enums.json`: el decompilado referencia índices numéricos que sin esa tabla
   son ilegibles.

## 1.4 Textos y strings

Volcar las tablas de strings indexadas (`@TAG` → texto). Son oro doble:

- Alimentan el inventario de *features* de la fase 3 (cada `@MENU_*` es una
  opción real del juego).
- Permitirán reutilizar los **textos literales** del original en la UI.

## Entregables

```
decompiled/
├── assets/images/...     PNG por cada .pic/.spr/.bmp
├── assets/media/...      WAV/AVI copiados o convertidos
├── data/
│   ├── asset_manifest.json
│   ├── <dominio>.json    catálogo(s) normalizados
│   ├── <binario>.json    volcados crudos
│   └── enums.json
└── tools/
    ├── extract_assets.py
    ├── extract_sprites.py / pack_sprites.py
    └── parse_*.py        (con --verify)
```

## Criterio de salida

- [ ] Todo fichero del inventario tiene destino: convertido, copiado o
  descartado con motivo en `FORMATS.md`.
- [ ] `asset_manifest.json` sin errores sin explicar.
- [ ] Volcados binarios verificados contra referencia (~100 %).
- [ ] Los strings están volcados y son navegables.
