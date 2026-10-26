## apw-meister

An RP2354A-based USB interface for the Antminer APW12 PSU data connector, derived from AditBoard v2.

The schematic uses the connector and GPIO mapping from bitcrane-v3.1 J10. See [schematic notes](doc/apw-meister-schematic.md) for the pinout and validation status.

The schematic uses RP2354A with 2 MB in-package flash and its internal switching regulator for the 1.1 V core supply. The PCB is 16 × 35 mm with six copper layers, APW12 at the top edge and USB at the bottom. All normally fitted SMD parts are on top. BOOT and RESET are optional bottom-side development buttons, marked DNP. Initial power routing, crystal connections, local MCU bypass connections, development-button signals, and APW-enable/test-pad signals are in place; 78 connections remain unrouted. Mechanical files, artwork, and manufacturing outputs still represent the original AditBoard design and need conversion before fabrication. APW12 control firmware is a separate task.

All purchased parts have JLCPCB/LCSC identifiers; see the [generated schematic BOM](doc/apw-meister-bom.csv). Optional pull-ups and development buttons are marked DNP (42 fitted positions, four DNP).

The placement passes courtyard and pad-to-pad clearance checks. Two inherited USB connector pad-to-locating-hole clearances measure 0.1944 mm against the 0.20 mm rule and remain to be resolved before fabrication. The board is not ready for manufacture.

All component models are project-local or embedded, including copies of the standard KiCad models in `3d/kicad/`, so Backplane does not depend on KiCad environment variables. Model paths were visually checked in Backplane.

Initial routing netclasses:

| Net group | Default track width | Via diameter / drill |
|---|---:|---:|
| 5 V | 0.50 mm | 0.60 / 0.30 mm |
| 3.3 V, 1.1 V, regulator LX | 0.40 mm | 0.60 / 0.30 mm |
| Ground | 0.50 mm | 0.60 / 0.30 mm |
| Filtered regulator AVDD | 0.20 mm | 0.50 / 0.30 mm |
| Ordinary signals | 0.15 mm | 0.50 / 0.30 mm |

The inherited 0.1016 mm clearance is retained. Short neckdowns accommodate fine-pitch pins; the LX passage between capacitors is 0.19 mm. USB differential geometry still requires verification against the chosen fabrication stackup before USB routing. Ground planes and final stitching remain pending.
