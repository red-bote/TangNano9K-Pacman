# Pac-Man - Digilent Basys 3

Pac-Man (© 1980 Namco Ltd.), FPGA core by MikeJ (www.fpgaarcade.com), Tang Nano 9K port
by pinballwiz.org (<https://github.com/pinballwizz/TangNano9K-Pacman>), built here for
the Digilent Basys 3 (Artix-7 `xc7a35tcpg236-1`).

## Hardware

- Digilent Basys 3.
- VGA monitor (640x480, 31 kHz), or a 15 kHz TV/CRT (sw13, not tested).
- USB keyboard on the Basys 3 USB-HID host port.
- Optional: Atari-style joystick on Pmod JA, Digilent PmodAMP2 on Pmod JXADC.

## Build

Requirements:

- Vivado 2020.2, gcc, make, unzip, openFPGALoader.
- Romsets `pacman.zip` and its MAME parent `puckman.zip` (graphics, PROMs), supplied by
  the user. Roms are not part of this repository.

```
make roms        # compile make_vhdl_prom, unzip romsets from ~/roms, write PROM VHDL
make bitstream   # Vivado batch build of basys3/pacman_basys3.xpr
make load        # program the Basys 3 (SRAM), building first if the bitstream is outdated
make clean       # remove roms, PROM VHDL and Vivado outputs
```

- `ROMDIR=<dir>` or `ROMZIP=<zip>[,<zip>...]` selects the romsets.
- `VIVADO=<path>` selects the Vivado binary.

## IO

| Input           | Function                                      |
|-----------------|-----------------------------------------------|
| Arrow keys      | move                                          |
| 1 / 2           | start 1 player / 2 players                    |
| 5               | coin                                          |
| btnU, btnD      | coin                                          |
| btnL / btnR     | start 1 player / 2 players                    |
| btnC            | reset                                         |
| JA 1/2/3/4      | joystick right/left/down/up (active low)      |
| sw0-sw7         | DIP switches (down = default, see below)      |
| sw12 up         | core scan doubler output (31 kHz, see VGA)    |
| sw13 down       | 31 kHz VGA                                    |
| sw13 up         | 15 kHz TV, composite sync on HS (not tested)  |
| sw14 up         | scanlines, alternate lines at 50 %, 31 kHz    |
| sw15 up         | audio off (PmodAMP2 shutdown)                 |

DIP switches (switch up inverts the default bit):

| Bits | Setting     | Default         |
|------|-------------|-----------------|
| 1:0  | coinage     | 1 coin 1 credit |
| 3:2  | lives       | 3               |
| 5:4  | bonus       | 10000           |
| 6    | difficulty  | normal          |
| 7    | ghost names | normal          |

## Resource use

From a Vivado 2020.2 build:

| Resource  | Used       | Util.                          |
|-----------|------------|--------------------------------|
| LUTs      | 1607       | 7.73%                          |
| Registers | 667        | 1.60%                          |
| Block RAM | 9.5 tiles  | 19.00%                         |
| DSPs      | 0          | 0.00%                          |
| Clock     | 24.571 MHz | WNS +23.831 ns, WHS +0.057 ns  |
| VGA clock | 25.595 MHz | WNS +33.993 ns, WHS +0.056 ns  |

The core runs on one 24.571 MHz clock (6.143 MHz pixel rate, original board 6.144 MHz)
with clock enables.

## VGA

The 31 kHz output is VGA 640x480 timing from `scandoubler_vga.v`, a changed copy of the
MiST scan doubler of the core (the core file stays unchanged for the Tang Nano 9K
build): 800 clocks per line at 25.595 MHz, negative syncs,
32.0 kHz line rate, 60.6 Hz frame rate. Each pixel and each line is shown twice
(576 x 448 picture, centred). The 25.595 MHz clock is a second output of the MMCM at
exactly 25/24 of the system clock, so the output timing stays locked to the core. The
core scan doubler output (sw12 up) has 384 clocks per line, which LCD monitors may
sample with a wrong pixel clock (auto adjust needed, edge columns cut off).

## Core changes for the Basys 3 port

Changes to the core files in `src/` (shared with the Tang Nano 9K project):

- DIP switch input, native video and timing outputs and scanlines input of `pacman` for
  the Basys 3 top (`pacman.vhd`). Unconnected in the Tang Nano 9K top, defaults keep its
  settings.
- Vivado elaboration error: palette PROM address connected in full
  (`pacman_video.vhd`).
- Vivado synthesis error: scan doubler `scanlines` input through a signal
  (`pacman.vhd`).
- Sprite errors (sprites one pixel early, errors where sprites overlap): sprite line
  buffer read on the pixel enable, as the original design (`dpram.vhd`,
  `pacman_video.vhd`).

## Third-party additions

PROM tool, unmodified, from <https://github.com/pinballwizz/DE2-Xevious> (commit
`a5d9eed`):

| File                                        | Function                   | Author  |
|---------------------------------------------|----------------------------|---------|
| `tools/tools_prom_src/src/make_vhdl_prom.c` | PROM-to-VHDL converter     | darfpga |

640x480 output in `basys3/pacman_basys3.srcs/sources_1/new/`, derived from the MiST scan
doubler `src/scandoubler_new.v` of the core (changes listed in its header):

| File                | Function                   | Author       |
|---------------------|----------------------------|--------------|
| `scandoubler_vga.v` | 640x480 VGA scan doubler   | Till Harbaum |

## License

- The additions and changes made for the Basys 3 port are MIT licensed.
- Core files (`src/pacman*.vhd`): (c) MikeJ, terms as stated in their headers. Other
  code keeps the terms stated in its files. `make_vhdl_prom.c` states no license.
- `scandoubler_vga.v`: GPL-3.0 or later, as the MiST scan doubler it is derived from
  (including the Basys 3 changes).
- Roms and anything generated from them (PROM VHDL, bitstreams) are not distributed.
