## apw-meister

An RP2354A-based USB interface for the Antminer APW12 PSU data connector, derived from AditBoard v2.

The schematic uses the connector and GPIO mapping from bitcrane-v3.1 J10. See [schematic notes](doc/apw-meister-schematic.md) for the pinout and validation status.

The schematic now uses RP2354A with 2 MB in-package flash and its internal switching regulator for the 1.1 V core supply. The PCB, mechanical files, artwork, and manufacturing outputs still represent the original AditBoard design and need conversion before fabrication. APW12 control firmware is a separate task.

All purchased parts have JLCPCB/LCSC identifiers; see the [generated schematic BOM](doc/apw-meister-bom.csv). Optional pull-ups are marked DNP.
