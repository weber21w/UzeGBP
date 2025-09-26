# UzeGBP — Game Boy (DMG) APU VGM Player for Uzebox

A lightweight **Game Boy (DMG) APU** player for **Uzebox**, capable of streaming **.VGM** from **SPI RAM** while mixing audio in real time. The SD card is used only to **prefetch/fill SPI RAM**, not for direct playback.

See discussion on the [Uzebox forums](https://uzebox.org/forums/viewtopic.php?t=11734

> **Status:** Working VGM player. **VGZ (gzip) is not yet implemented** (planned).

---

## Features

- DMG APU emulation: **2× pulse** (Ch1 w/ sweep, Ch2), **wave** (Ch3, 32×4-bit), **noise** (LFSR 15/7-bit).
- Real-time **VGM** parser (DMG command set → Uzebox mixer).
- **SPI-RAM streaming path**: background sector prefetch from SD → ring/linear buffer in SPI RAM → zero-seek playback.
- Optional **shadow for Ch3 waveform** updates to avoid mid-sample artifacts.
- File browser UI, pause/play/FF, on-screen timer, skin color cycling.

## Requirements

- **Uzebox** with **≥128 KiB SPI RAM** (works with larger parts up to 8 MB).
- SD card (FAT) for file loading (**Petit FatFs**).
- **AVR-GCC** toolchain compatible with Uzebox builds.

## Building

1. Install the Uzebox SDK/toolchain.
2. Place sources in a new project directory.
3. Build normally (e.g., `make`) to produce the ROM.

## Usage

1. Copy **`.VGM`** files to SD. (VGZ is **not** supported yet.)
2. Boot Uzebox, open the file selector, pick a track.
3. Playback reads from **SPI RAM**; the loader/prefetcher fills it from SD.  
   Controls: pause/play/FF, volume, skin. Timer shows **MM:SS:ff**.

## Technical Notes

- **Mixer rate:** `262 * 60 = 15720 Hz` (Uzebox line rate).
- **DMG base clock:** `4,194,304 Hz`; channel tick sources derived per hardware.
- **VGM timing:** 44.1 kHz waits are converted to 15.72 kHz samples with fractional carry to preserve long-term sync.
- **Frame sequencer (DMG):** **512 Hz**, 8-step  
  - Length: **256 Hz**  
  - Sweep (Ch1): **128 Hz**  
  - Envelope: **64 Hz**
- **Pulse (Ch1/Ch2):** 12.5/25/50/75% duty; Ch1 implements hardware sweep; both support length & envelope.
- **Wave (Ch3):** 32×4-bit RAM with 4-level output scaling; safe mid-sample write behavior; optional banked/shadow update.
- **Noise (Ch4):** Programmable LFSR (clock shift/divisor) with short-mode (7-bit); length & envelope supported.
- **Stereo/panning:** **NR50/NR51** respected and mixed down for Uzebox output.
- **I/O path:** SD → sector prefetch → **SPI RAM ring/linear buffer** → parser/decoder → mixer. Playback never seeks SD during active mixing.

## Roadmap

- **VGZ** (gzip) decompression.
- Optional **GBS** playback mode.
- Playlist support and additional UI polish.

## Credits

- Initial inspiration: **gbsplay** by **mmitch** — https://github.com/mmitch/gbsplay  
- Uzebox, Petit FatFs, and community resources for platform support.

## License

This repository is licensed under **GPL-3.0**.  
