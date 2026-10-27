# apw-meister

A CH32X035F8U6-based USB interface for the Antminer APW12 PSU data connector, derived from AditBoard v2.

The `WCH32` branch contains the CH32 hardware migration. The RP2354A version, including its initial routing, is preserved on `main` at `2b5b7f9`.

The CH32 provides internal flash, clock and core regulation. The board retains its USB-powered 3.3 V LDO and adds MOSFET isolation to prevent APW I2C pull-ups from backfeeding an unpowered MCU. See the [schematic notes and firmware pin map](doc/apw-meister-schematic.md) and [BOM](doc/apw-meister-bom.csv).

The PCB is 14 × 28 mm, reduced from 16 × 35 mm (30% less area), with six copper layers, APW12 at the top edge and USB at the bottom. Placement groups the APW interface below its connector, the CH32 in the middle, and USB protection beside the USB connector. All normally fitted SMD parts are on top. BOOT and RESET remain optional bottom-side DNP development buttons. All purchased parts have JLCPCB/LCSC numbers and project-local 3D models; the MCU model is a generic QFN visualization.

The first CH32 routing pass is complete, including ground and power planes. In1.Cu and In4.Cu provide ground references; In2.Cu distributes 3.3 V and In3.Cu distributes 5 V, with a few signal crossings on the power layers. Final USB geometry and fabrication review, the inherited USB connector hole-clearance issues, and the CH32 firmware port remain. Existing AditBoard artwork, mechanical and manufacturing outputs are historical and must not be used to order this revision.

Validation: the unchanged schematic has zero ERC violations; all 118 pad-to-net assignments match. Filled-board DRC has zero unrouted connections, zero schematic-parity issues, two inherited USB hole-clearance errors, and two warnings: a leftover BOOT trace stub and the decorative logo’s missing courtyard. See the [validation record](doc/ch32-validation.json).
