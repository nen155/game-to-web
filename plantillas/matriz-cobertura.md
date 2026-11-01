# Plantilla — matriz de cobertura

`docs/matriz-cobertura.md`. Se genera al final de la fase 3 y se revisa cada
vez que el decompilado o los strings revelen algo nuevo.

## Dirección 1 — toda feature tiene HU (o descarte justificado)

| # | Feature (del inventario) | Fuente | HU(s) | Estado |
|---|--------------------------|--------|-------|--------|
| F-01 | Pagar soborno en el encuentro | `@ENCOUNTER_PREDUEL`, `Uistrings` | HU-06 | ✅ cubierta |
| F-02 | Conexión TEN original | readme §red | — | 🚫 fuera de alcance (obsoleto) |
| F-03 | … | … | ⬜ **SIN HU** | ⬜ |

## Dirección 2 — toda HU tiene fuente

| HU | Fuente citada | ¿Verificable? |
|----|---------------|---------------|
| HU-01 | Manual + `Advinter800` | Playwright |
| HU-02 | Readme (consumo comida) + `stepDuration` | Test + Playwright |
| HU-xx | *(sin fuente — revisar: ¿inventada?)* | — |

## Reglas

- Una fila `SIN HU` ni `sin fuente` no puede quedar al cerrar la fase.
- `🚫 fuera de alcance` exige motivo (obsoleto, depende de hardware, legal…).
- Descubrimientos posteriores → nuevas filas F-xx + nueva tanda de HUs.
