# apw-meister — CH32X035F8U6 revision

The `master` branch replaces the RP2354A with WCH's CH32X035F8U6. The complete RP2354A design and its initial routing are preserved on `main` at commit `2b5b7f9`.

## MCU and supply

U3 is CH32X035F8U6, LCSC/JLCPCB **C42442062**, in QFN20 (3 × 3 mm, 0.4 mm pitch). Its exposed ground pad is numbered **21** in this project's symbol and footprint; WCH's reference schematic calls it pad 0. It must connect to ground.

The MCU has 62 KB application flash, 20 KB SRAM, a 48 MHz core, an internal oscillator and an internal core regulator. U4 remains AP2112K-3.3, powered by USB VBUS, to supply 3.3 V to the MCU and APW bus pull-ups. C17/C18 are its 1 µF input/output capacitors, and C10 is the local 100 nF MCU bypass.

The RP2354A core buck regulator, inductor, crystal, load capacitors and unused bypass capacitors have been removed. No external flash or clock is required for this CH32 circuit.

## Firmware pin map

| Function | CH32 pin | GPIO | Net |
|---|---:|---|---|
| APW SDA | 2 | PA0 | MCU_PSU_SDA |
| APW SCL | 3 | PA1 | MCU_PSU_SCL |
| PSU enable drive | 6 | PA4 | MCU_PSU_ON |
| Addressable LED | 9 | PA7 / SPI MOSI | LED_DATA |
| Debug data | 14 | PC18 / DIO | SDI_DIO |
| Debug clock | 16 | PC19 / DCK | SDI_CLK |
| USB D− | 17 | PC16 / UDM | USB_DN_MCU |
| USB D+ / boot strap | 18 | PC17 / UDP | USB_DP_MCU |
| Supply | 1 | VDD | 3V3 |
| Exposed ground | 21 | GND | GND |

Use software I2C on PA0/PA1. On this package, hardware I2C alternatives conflict with USB or the debug pins. The bitcrane reference uses 400 Hz I2C (1,250 µs half-period) and a 50 ms clock-stretch timeout. Release SDA/SCL for a high state; never actively drive the APW bus high. Keep USB servicing responsive while waiting for the PSU. PA7 permits SPI/timer-based LED signaling without long interrupt blocking.

Initialize the USB peripheral for **3.3 V VDD**, including WCH's appropriate PHY/pull-up configuration. R2/R3 are 0 Ω series tuning positions matching the direct USB connection in WCH's reference circuit.

Existing RP2040/RP2354 firmware is not binary compatible. A CH32 RISC-V firmware port is still required; this repository change is the hardware migration. The smaller flash and RAM require an explicit firmware memory budget.

## APW12 connector and isolation

J2 follows bitcrane-v3.1 J10. Interface voltage is confirmed to be 3.3 V.

| J2 pin | Connection |
|---:|---|
| 1 | APW12_PIN1 → TP1 only |
| 2 | GND |
| 3 | PSU_SDA |
| 4 | APW12_PIN4 → TP2 only |
| 5 | PSU_EN, active low |
| 6 | PSU_SCL |

Q2/Q3 are DMG1012T-7 (C20512), the same part as Q1. Each has gate at 3.3 V, source at the MCU-side bus and drain at the APW-side bus. This arrangement isolates the MCU from PSU pull-ups when USB power is absent. R10/R11 are now **fitted** 4.7 kΩ pull-ups on the MCU side. The APW side relies on its existing PSU pull-ups. U6 protects the connector-side SDA, SCL and enable lines.

Q1 remains the enable pull-down stage. PA4 high turns Q1 on and pulls PSU_EN low. R13 holds Q1 off at reset and while unpowered. Firmware should establish the off state before other initialization and return to it on communication failure.

TP1/TP2 are bare copper pads, not purchased components.

## Boot, reset and debug

The F8U6 package has **no external reset pin**. Optional SW3 now grounds U4's enable input (`REG_EN`), while R7 pulls it up to USB 5 V. U4's internal output discharge pulls down 3.3 V when disabled, producing a power-cycle reset. Hold long enough for the rail to discharge. A powered external debugger must not hold the rail up through its I/O during this operation.

Optional SW2 pulls PC17/USB D+ toward 3.3 V through R6 = 4.7 kΩ, following WCH's F8U6 reference schematic. To enter the factory bootloader: hold BOOT, press/release RESET (or reconnect USB), then release BOOT and use WCH USB ISP tooling. This is not the RP UF2 workflow.

USB-C SBU1 carries SDI_CLK and SBU2 carries SDI_DIO for a dedicated programming fixture. These signals use WCH's debug protocol; an ARM SWD probe is not interchangeable. Ordinary USB cables are not the debug fixture. A programming fixture must observe the connector orientation and 3.3 V levels.

SW2 and SW3 remain bottom-side, **DNP** development options. All normally fitted SMD parts are on top.

## Libraries and 3D models

`wch32.kicad_sym` contains the package-specific symbol. The project-local `aditBoard` footprint library supplies the 3 × 3 mm QFN20 footprint with a 1.7 × 1.7 mm ground pad and segmented paste. This is within WCH's specified exposed-pad range (1.75 ± 0.2 mm). Q2/Q3 reuse the existing SOT-523 footprint and DMG1012T model.

The MCU uses the project-local standard KiCad UQFN20 3 × 3 mm STEP as a **generic visualization model**. It is not vendor-certified CH32 package geometry; do not use its height or exposed-pad details as a mechanical acceptance drawing. The electrical footprint is checked against WCH's package drawing. All purchased positions have JLCPCB/LCSC fields; see `apw-meister-bom.csv`.

## PCB status

The CH32 placement uses a 14 × 28 mm outline, reduced from the RP2354A layout's 16 × 35 mm (30% less area). APW12 remains at the top and USB at the opposite edge, with the USB connector moved inward along with the shorter board. The APW protection and isolation parts sit below J2; the MCU and bypass capacitor occupy the center; USB protection and CC resistors sit immediately above J1. The side-facing LED remains on the left edge. Normally fitted SMD parts remain on top; BOOT and RESET remain bottom-side DNP options. The six-layer stackup and manufacturing minima are unchanged.

The first CH32 routing pass is complete: 269 track segments, 49 through vias, and three zone objects covering all six copper layers. The ground zone spans F.Cu, B.Cu, In1.Cu and In4.Cu. In1.Cu and In4.Cu are ground reference planes without signal tracks. In2.Cu distributes 3.3 V with a short USB pin-order crossover; In3.Cu distributes 5 V with several control/configuration crossings. F.Cu and B.Cu carry signals and ground fills. The MCU exposed-pad ground escapes to a via outside its solder land. U5's two ground pins share a via in the gap between their solder lands. Neither connection requires a filled via in a solder pad.

R2/R3 are oriented and aligned with their USB protection channels; R9 faces its source/load connections. USB data tracks use 0.15 mm width and 0.45/0.30 mm through vias, with the reversible connector contacts joined locally. This is a connectivity- and clearance-checked first pass; USB impedance and overall fabrication review remain. Filled-board DRC reports zero unrouted connections and zero schematic-parity issues. The two inherited J1 locating-hole clearance errors remain (0.1944 mm versus the 0.20 mm rule). The latest manual edits move the DNP buttons toward the side edges, add a rear silkscreen logo, and round the board corners. Review also flags a leftover BOOT trace stub and a harmless missing-courtyard warning for the logo, which is excluded from BOM and position files. Do not use historical manufacturing outputs for this revision.

Power routing defaults remain 0.50 mm for VBUS/GND and 0.40 mm for 3.3 V, with 0.60/0.30 mm vias. Ordinary signals default to 0.1016 mm (4 mil) with 0.45/0.30 mm vias. Short neckdowns may be needed at fine-pitch pins. USB differential geometry must be checked against the chosen fabrication stackup. Unused RP core-regulator netclasses are historical settings with no connected loads.

## References

- [WCH CH32X035 datasheet](https://www.wch-ic.com/downloads/CH32X035DS0_PDF.html), F8U6 pinout and QFN20 package drawing.
- [WCH evaluation board reference](https://github.com/openwch/ch32x035/blob/main/EVT/PUB/CH32x035%20Evaluation%20Board%20Reference-EN.pdf), F8U6 USB boot and supply circuit.
- [WCH USB CDC example](https://github.com/openwch/ch32x035/tree/main/EVT/EXAM/USB/USBFS/DEVICE/SimulateCDC).
- [DMG1012T datasheet](https://www.diodes.com/datasheet/download/DMG1012T.pdf), pin assignment and low-voltage gate characteristics.
- [AP2112 datasheet](https://www.diodes.com/assets/Datasheets/AP2112.pdf), enable and automatic output discharge.
- Local `~/Bitcoin/bitcrane/bitcrane-v3.1` J10 and `~/Bitcoin/bitcrane/bitcrane-fw/src/control/psu.rs`.

## Track and via manufacturing limits

Imported from `~/Bitcoin/bitaxe/displays/bonanzaDisplay-v3/bonanzaDisplay.kicad_pro` on 2026-10-26 for the JLCPCB six-layer standard process:

| Setting | Value |
|---|---:|
| Minimum track width | 0.1016 mm (4 mil) |
| Minimum copper clearance (already matched) | 0.1016 mm (4 mil) |
| Minimum via pad diameter | 0.45 mm |
| Minimum through drill | 0.30 mm |
| Minimum via annular ring | 0.075 mm |

The Default signal netclass uses those track/via sizes, and the minimum sizes are available in the routing dropdowns. Wider power/ground netclasses are retained. This import covers track/via sizes, not the source project's other DFM rules or stackup.

[JLCPCB's published capabilities](https://jlcpcb.com/capabilities/pcb-capabilities/) support these dimensions for standard multilayer copper and place this via combination outside the listed small-via surcharge cases. This is a geometry check, not a manufacturing quote.

DRC after import: two existing J1 hole-clearance violations, 73 unrouted connections, zero schematic/PCB parity issues. No new violations.
