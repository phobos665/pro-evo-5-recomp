# Pro Evolution Soccer 5 (Xbox) — static recompilation

A native build of Konami's *Pro Evolution Soccer 5* (Xbox, 2005; EU disc,
title ID `0x4B4E0030`, XDK 5849) made with the
[xboxrecomp](https://github.com/phobos665/xboxrecomp) toolkit. The game's own
code is lifted to C from the XBE and runs natively; Direct3D 8 and
DirectSound are replaced at the API boundary.

**No game code or data is in this repository.** You need the files from a
copy of the disc you own.

## Current State

Bring-up in progress.

## Building

You need the toolkit checked out beside this repository, at `../xboxrecomp`.
Point `XBOXRECOMP_DIR` elsewhere to use another checkout, both for
`scripts/recompile.sh` and for CMake (`-DXBOXRECOMP_DIR=...`). The toolkit's
own requirements apply: see its `docs/GETTING_STARTED.md`.

1. **Game files.** Extract the disc into `game/`, so that `game/default.xbe`
   exists.
2. **XDK symbols.** From the toolkit:

   ```bash
   .venv/bin/python -m tools.xdk_symbols ../pes5-recomp/game/default.xbe
   ```
3. **Lift.** `scripts/recompile.sh`.
4. **Build.**

   ```bash
   cmake -S . -B build/Release -G Ninja -DCMAKE_BUILD_TYPE=Release
   cmake --build build/Release --target pes5_recomp pes5_launcher --parallel
   ```

## Running

```bash
build/Release/pes5_launcher      # settings, then the game
build/Release/pes5_recomp        # the game directly
```

F9 shows the frame rate, F10 steps the frame cap, F11 saves a capture of the
frame on screen. Test runs belong on a fresh `RECOMP_SAVE_DIR`.

## What is specific to this game

- **Data sections** (`config/sections/`): the online and overlay data
  sections are not code.
- **Seeds** (`config/seeds/`): functions the disassembler cannot find on its own.
