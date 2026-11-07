# ChoralRoot + FM-1 Big Screen

[ChoralRoot](https://github.com/Quixotic7/ChoralRootFM1) by Quixotic7 (built on Leo Kuroshita's
[Felucca](https://github.com/hugelton/Felucca)), with one addition: while an app asks for it, the FM-1 sends its
screen over its USB cable, so [FM-1 Mirror](https://github.com/pfkellogg/fm1-mirror) (Mac, Windows, Linux) and
[FM-1 Mirror + Spyglass](https://github.com/pfkellogg/fm1-mirror-spyglass-android) (Android) show it big, live. For
players who can't read the FM-1's 1.54-inch screen.

This is an **unofficial** build, not a ChoralRoot release, and Quixotic7 has nothing to do with it. Everything else is
ChoralRoot 0.29 as published. Tested on a real FM-1 (2026-11-02): the main view, the chord names, the sound editor,
the layers and the Options, with FM-1 Mirror on a Mac and a USB keyboard playing it.

Questions: **pfkellogg@gmail.com**

A fourth: **[SLOOP + FM-1 Big Screen](https://github.com/pfkellogg/sloop-fm1/blob/big-screen/BIG_SCREEN.md)**, tested on a real FM-1 (2026-11-07). Watch it: [SLOOP on a Big Screen](https://youtu.be/Pd4Bm6JvtkU).

A fifth: **[Melodee + FM-1 Big Screen](https://github.com/pfkellogg/melodee/blob/big-screen/BIG_SCREEN.md)**, tested on a real FM-1 (2026-11-07).

A third community firmware with the same addition: **[Skiff + FM-1 Big Screen](https://github.com/pfkellogg/skiff/blob/big-screen/BIG_SCREEN.md)**;
all three come from Felucca ([the family tree](https://github.com/pfkellogg/skiff/blob/big-screen/FIRMWARE_FAMILY.md), [video](https://youtu.be/lmMT-FtAoSE)).

A second community firmware with the same addition: **[JIANT FM + FM-1 Big Screen](https://github.com/pfkellogg/JIANT/blob/big-screen/BIG_SCREEN.md)**
(juanjiant's music system; 38 lines, the same 7 doing the work). Watch it: [JIANT FM on a Big Screen](https://youtu.be/7o5s0EDJX5c).

## Installing it

ChoralRoot's installers install only over firmware they know is safe: **M-VAVE's original V15**, Felucca 1.0+,
Melodee, or ChoralRoot itself. Not over M-VAVE's Open firmware or the FM-1 Big Screen firmware (an install over an
unknown firmware has left FM-1s that no longer start; docs/INSTALL-COMPAT.md). So:

1. **If the FM-1 runs M-VAVE's Open firmware or Big Screen:** put the original V15 back first, on M-VAVE's
   [FM1 Workbench](https://open.m-vave.com/fm1-workbench) (Chrome or Edge): Device update › Recovery & force update ›
   Restore original V15. (On V15 or ChoralRoot already: skip this.)
2. **Charge the FM-1, plug it straight into the computer** (no hub), close every app that uses it.
3. With this repository and Python 3 (`pip3 install python-rtmidi`), from its folder:

   ```
   python3 tools/fm1_install.py ChoralRoot-0.29-BigScreen.fwsc --yes
   ```

   (the `.fwsc` from this fork's Releases). It checks what the FM-1 runs, writes, and ends with
   `done: the FM-1 runs FM-1_920`. Over ChoralRoot you can add `--backup .` to save your sounds and settings first.

**Going back:** ChoralRoot's own [web installer](https://quixotic7.github.io/ChoralRootFM1/webapp/installer/)
installs the official ChoralRoot over this one as an ordinary update; its "Return to official V15" puts back M-VAVE's
firmware.

## Seeing the screen

Open **FM-1 Mirror** (Mac, Windows, Linux) or **FM-1 Mirror + Spyglass** (Android) with the FM-1 plugged in by USB:
the screen appears, big, in colour or high-contrast looks. Over **USB only**: ChoralRoot doesn't use the FM-1's
Bluetooth MIDI, so FM-1 Mirror Bluetooth can't show it.

The apps' "Update FM-1" (Big Screen firmware) never shows on ChoralRoot (from FM-1 Mirror 1.2.1 and Mirror + Spyglass
1.3.1): it would replace it.

## Playing it from a USB keyboard

The FM-1 can't read a USB-only keyboard by itself; FM-1 Mirror's **Keys** passes one on. ChoralRoot's chord engine
listens on its **MIDI Keys** channel:

1. On the FM-1: tap **OPT** (printed GLO), turn **SELECT** to **MIDI Keys** (the 7th setting), turn **KNOB 1** to
   **ch 3**, tap **OPT** to close.
2. In FM-1 Mirror: **Keys** › your keyboard, **Send on** › **Channel 3**.
3. Play **F3 to G5**: **F3 to B3** are ChoralRoot's chord keys (they shape the chord, no sound alone), **C4 to G5** the
   roots (they sound). Keys outside F3–G5 do nothing.

**Velocity:** ChoralRoot's **Key Touch** (Options, three past MIDI Keys) is **Fixed** at first: every key plays at
**Velocity** (100). Set it to **Dynamic** and the keyboard's own velocity plays each key (the FM-1's own keys then sense
velocity too: Options › Calibrate).

Channel 1 (MIDI Perform) plays the chord sound as a plain synth instead, one note a key; channel 2 is the bass.

## What changed in the code

**32 lines in ChoralRoot's own files**, and only **7 of them do the work**; the rest are an on/off switch,
`CR_BIG_SCREEN` (1 in `choralroot.c` only), so Felucca and the update loader, which share these files, build exactly
as before:

| File | Lines | What |
|---|---|---|
| `firmware/src/lcd.c` | 7 | `mirror_lcd(...)` in `lcd_blit` and `lcd_fill`: every rectangle on its way to the LCD |
| `firmware/src/usb.c` | 13 | where `sysex_byte` finishes a message: a 6-byte `F0 7D 'F' 'S' .. F7` goes to `sm_sysex`; its SysEx send ring's size (`SXQ`, 64) can be set from outside |
| `firmware/src/main.c` | 3 | `mirror_service()` once a main-loop pass |
| `firmware/src/choralroot.c` | 9 | the switch, `#include "cr_mirror.c"`, and `SXQ 512`: with 64 the screen went out at ~25 KB/s and a big chord name took 0.9 s to reach the Mac; with 512, ~67 KB/s and 0.3 s |

Two new files, both MIT:

- `firmware/src/cr_mirror.c` (60 lines): the adapter. ChoralRoot's pixels are byte-swapped for the LCD: each row goes
  back into RGB565 for `sm_rows`; each message goes into usb.c's SysEx queue only when it has room (never waiting on
  USB); a new viewer, or rows that had to be dropped, get the whole screen again (`ui.force = 1`, at most twice a
  second). A draw never waits more than 0.4 s, so a computer that stops reading can't slow the FM-1 down.
- `firmware/src/screen_mirror.c` / `.h`: the firmware-neutral screen mirror from
  [pfkellogg/fm1_open, `screen_mirror/`](https://github.com/pfkellogg/fm1_open/tree/screen-out/screen_mirror), with a
  6 KB buffer (ChoralRoot's RAM is tight).

Cost: about 1 KB of flash and 8 KB of RAM (XIP 97.4%, RAM 90.5%). The boot stub is unchanged, so installing it
never rewrites the one flash sector an interruption can't recover from.

## Building it

As ChoralRoot (BUILDING.md), or on a Mac without Docker: `tools/build.py` also takes JieLi's **Windows** toolchain
2.5.3 under Wine (the same clang 4.0.1 for pi32v2):

```
JIELI_WINE=/path/to/wine JIELI_WIN_BIN=/path/to/JL/pi32/bin ./build.sh
```

The font step needs Pillow with libraqm (an Intel Python with Homebrew's x86 `libraqm` works on Apple silicon).

## Licence

ChoralRoot and Felucca: GPL-3.0-only (LICENSE, LICENSING.md), and so is this build as a whole; its source is this
branch. `cr_mirror.c`, `screen_mirror.c` and `screen_mirror.h`: MIT, © 2026 Patrick F. Kellogg, so any FM-1 firmware
can use them. M-VAVE and FM-1 are M-VAVE's names.
