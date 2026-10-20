# Pro Evolution Soccer 5 (Xbox) — static recompilation

A native build of Konami's *Pro Evolution Soccer 5* (Xbox, 2005; EU disc,
title ID `0x4B4E0030`, XDK 5849) made with the
[xboxrecomp](https://github.com/phobos665/xboxrecomp) toolkit. The game's own
code is lifted to C from the XBE and runs natively; Direct3D 8 and
DirectSound are replaced at the API boundary.

**No game code or data is in this repository.** You need the files from a
copy of the disc you own.

## Current State

Runs on macOS (Apple Silicon) from boot into matches: the Konami, JFA and
adidas screens, the intro movie, the title screen, option-file creation and
saving, the main menu, Match mode (team and kit select, match settings,
formation) and the match itself -- walk-out, kick-off, play, replays -- at
60 fps with sound. Six minutes into a match with no crash.

- **60 fps, natively.** The game asks for a 60 Hz mode with a presentation
  interval of one, and presents one frame per vblank in a match (59.7-59.9
  fps measured). Its match clock runs at the configured match length.
- **Widescreen** (`widescreen = 1`, launcher or `RECOMP_WIDESCREEN=1`).
  PES5 has no 16:9 mode, so its camera is widened (Hor+, below). The HUD
  stays 4:3 in the middle; full-screen menu backdrops fill the width.

Not yet checked: the other modes (Master League, League, Cup, Training,
Edit), online (XBOX LIVE), and how it sounds by ear.

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
- **Widescreen declaration.** `recomp_title_widescreen(HOR_PLUS REGISTER 154)`:
  world draws project through one screen-space matrix at c154..c157, whose x
  and y rows are in the ratio 240:224 (a 4:3 projection on the game's
  640x448 viewport), so 0.75 on c154 widens the view to 16:9. The game
  renders its match into a 640x448 texture and composites it onto the
  screen; the toolkit treats that texture as the screen.

The toolkit fixes this game needed (on `title/pes5-play`): jump tables read
to their bounds check (a 1007-case switch), `IDirectSound::SynchPlayback`,
an interlaced display field that alternates, one-operand `mul`/`imul` at the
operand's width (115 byte-wide sites here; one zeroed libpng's image height
and corrupted the heap), and the scene-texture rules for widescreen.
