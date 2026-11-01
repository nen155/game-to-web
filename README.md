# Game-to-Web

A documented, repeatable pipeline to bring **any game** to a **playable browser clone**
that reuses the original data, art and sound whenever possible.

The process was generalized from the work done on *Magic: The Gathering – Shandalar*
(MicroProse, 1997, native Windows), but it covers any kind of game:

- native binaries (Windows, DOS, Linux);
- managed code (.NET/Unity, JVM/Android);
- web and Flash games already packaged;
- mobile apps (APK) and ROMs;
- **disc images** (ISO, BIN/CUE, NRG, CHD, DMG);
- **installers without installing** (MSI, Inno Setup, NSIS, InstallShield);
- projects with **available source code** (the decompilation phase is replaced by reading it).

> The pipeline is a **template, not a rigid script**. Phase 0 decides which branches apply
> depending on the platform, engine and language; every phase documents its alternatives.
> If something does not apply, the process adapts (equivalent tool, partial extraction or
> reconstruction from external references) instead of stopping. Any deviation is recorded
> in the project's `AGENTS.md`.

## Quick start

The process is automated with a Devin skill:

```
/juego-a-web <path to the game, image or installer>
```

It accepts an installed game folder, a disc image, an installer, an APK, a ROM or a
repository with sources. Images and installers are extracted to `<game>/extracted/`
without installing anything, and the pipeline continues on that tree.

The skill runs the thirteen phases autonomously. This repository is the detailed
reference for each phase: read it to understand what the skill does, adjust decisions
or run the process by hand.

## Phases

| Phase | Goal |
|---|---|
| 0 · Reconnaissance | Identify the platform, engine, language and every file |
| 1 · Extraction | Unpack images/installers and collect assets without modifying the originals |
| 2 · Decompilation | Recover code and data structures (or read the sources) |
| 3 · User stories | Turn the original mechanics into testable user stories |
| 4 · Stack | Choose the target stack and architecture for the web clone |
| 5 · Interface | Map the original screens and controls to the browser |
| 6 · Implementation | Build the clone, reusing the original assets where possible |
| 7 · Verification | Playwright checks, visual comparison and playthroughs |
| 8 · Recursive coverage | Measure which features, screens and files are covered |
| 9 · Bugs | Report, reproduce and fix defects |
| 10 · Models | Port game data models and rules to the target stack |
| 11 · Animations | Reproduce sprites, timing and effects |
| 12 · HUDs | Rebuild menus, HUDs and overlays |
| 13 · Final report | Summary, gaps and next steps |

## Repository layout

```
fase-0-reconocimiento.md … fase-12-huds.md   Detailed reference for each phase (Spanish)
plantillas/                                  Templates: user story, bug report,
                                             Playwright report, coverage matrix, prompt
README.es.md                                 Original Spanish README
```

## Status

Reference documentation and templates. Each clone produced with the pipeline lives in its
own repository; this one documents **how** to do it.

## Author

Emilio Chica Jiménez — [miliolab.com](https://miliolab.com) · [GitHub](https://github.com/nen155)
