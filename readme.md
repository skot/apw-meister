# apw-meister

A CH32X035F8U6-based USB interface for the Antminer APW12 PSU data connector, derived from AditBoard v2.

The `WCH32` branch contains the CH32 hardware migration. The RP2354A version, including its initial routing, is preserved on `main` at `2b5b7f9`.

The CH32 provides internal flash, clock and core regulation. The board retains its USB-powered 3.3 V LDO and adds MOSFET isolation to prevent APW I2C pull-ups from backfeeding an unpowered MCU. See the [schematic notes and firmware pin map](doc/apw-meister-schematic.md) and [BOM](doc/apw-meister-bom.csv).

The PCB is 16 × 35 mm with six copper layers, APW12 at the top edge and USB at the bottom. All normally fitted SMD parts are on top. BOOT and RESET remain optional bottom-side DNP development buttons. All purchased parts have JLCPCB/LCSC numbers and project-local 3D models; the MCU model is a generic QFN visualization.

The schematic and placement have been converted. Routing, planes, final fabrication checks and the CH32 firmware port remain. Old RP copper was removed from this branch because the new MCU has a different pinout. The inherited USB connector hole-clearance issues also need resolution before fabrication. Existing AditBoard artwork, mechanical and manufacturing outputs are historical and must not be used to order this revision.

Validation: ERC has zero violations; all 118 pad-to-net assignments match the schematic. Placement DRC has only the two inherited USB hole-clearance violations, with 73 connections unrouted. See the [validation record](doc/ch32-validation.json).
