# Plantilla — fichero de historias de usuario

Un fichero por tanda/dominio: `NN-<dominio>.md` dentro de
`docs/historias-usuario/`. El `00-vision.md` siempre es el primero.

```markdown
# Historias de usuario — <dominio>

Cada historia lleva su **criterio de aceptación**: es lo que se comprueba
(con test unitario o con Playwright) antes de darla por terminada. Los IDs
`HU-xx` son estables y se referencian desde `docs/BACKLOG.md`.

---

## <Grupo funcional>

### HU-xx · <nombre corto y verbal>
**Como** jugador, **quiero** <capacidad concreta> **para** <beneficio>.

- <criterio comprobable 1 — con cifras/literales del original si existen>
- <criterio 2>
- <criterio 3>

> Fuente: <@TAG / fichero de strings · readme §x · ghidra/out/X.c FUN_… ·
> manual pág. n · vídeo minuto m>

### HU-xx+1 · …
```

Reglas:

- Criterios **comprobables**: números, teclas, textos literales.
- **Fuente citada** en cada HU (o marcada como decisión de diseño propia).
- Una HU = una capacidad. Si crece, se parte.
- Nunca reutilizar ni renumerar un `HU-xx`.
