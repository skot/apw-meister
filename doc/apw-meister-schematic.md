# apw-meister schematic draft

The canonical schematic is `apw-meister.kicad_sch`. The APW12 interface is copied electrically from J10 in `~/Bitcoin/bitcrane/bitcrane-v3.1/bitcrane.kicad_sch`.

| J2 pin | Signal | Connection |
| --- | --- | --- |
| 1 | APW12_PIN1 | TP1 only; function unspecified in the reference design |
| 2 | GND | USB/RP2354A ground |
| 3 | PSU_SDA | U3 pin 18 / GPIO14 / I2C1 SDA |
| 4 | APW12_PIN4 | TP2 only; function unspecified in the reference design |
| 5 | PSU_EN | Q1 drain; active-low PSU enable |
| 6 | PSU_SCL | U3 pin 19 / GPIO15 / I2C1 SCL |

J2 uses the reference's `CONN-TH_6P-P2.00_PHD-2X3A` footprint, a 2×3 connector with 2 mm pitch, and LCSC identifier C2845614. Its footprint and STEP model are copied intact into this project. Pin numbers refer to the connector footprint, not a view of the cable plug.

## Changes

- Replaced the 18-pin hashboard connector with the APW12 connector.
- Removed R25/R27/R29 and the hashboard address, UART, reset, and plug-detect nets. GPIO8–GPIO11 are now marked unused.
- Retained USB power, USB data, SWD over the USB-C SBU pins and status LED; U3 is now RP2354A.
- Used the standard AP2112K-3.3 symbol for U4 to correct its VOUT electrical type to power output. The part, footprint, pin numbers, and electrical connections remain the same. Added power-source flags on USB 5 V and ground.
- Updated the title/revision and removed the inherited AditBoard drawing sheet from the project settings.

The user confirmed the APW12 interface is **3.3 V**. SDA and SCL retain their bitcrane GPIO assignments. No 3.3 V or 5 V supply is connected to J2.

## Review fixes

- Q1 (DMG1012T-7) now pulls PSU_EN low when **GPIO25 is high**. R12 is a 1 kΩ gate resistor; R13 is a 10 kΩ gate pull-down. Reset and USB power loss turn Q1 off, releasing EN. This requires the PSU to bias released EN high/off; confirm that behavior on the intended APW12. Firmware must initialize GPIO25 low and use watchdog reset on faults. A firmware crash that leaves GPIO25 high is not automatically detected by this circuit.
- R2/R3 are 27 Ω, C25100, matching the RP235x USB recommendation.
- Y1 is ABM8-272-T3 (C20625731), with C2/C4 = 15 pF C0G (C1548) and R8 = 1 kΩ. This follows Raspberry Pi's characterized crystal circuit.
- U1 has a dedicated 100 nF bypass capacitor C19 and a 330 Ω DIN resistor R9. Its exact part supports 3.3 V data at a 5 V supply.
- External flash U2, its bypass C5, and pull-up R1 were removed for RP2354A. R6 remains as the 1 kΩ BOOTSEL series resistor on QSPI_SS (U3 pin 60). QSPI SCLK and SD0–SD3 are unconnected externally.
- J1 is GCT USB4105-GF-A-120 (C5184243). The suffix specifies 1.20 mm mounting stakes; the existing GCT footprint's XY geometry is retained. MPN, manufacturer, LCSC and datasheet fields agree.
- SW2/SW3 are now **populated**, using stocked B3U-1000P switches (C231329) and their matching standard footprints. Their normal use is BOOTSEL and RESET; SWD remains available over USB-C SBU with a suitable adapter.
- U5 (TPD6E05U06RVZR, C962978) protects USB D+/D−, CC1/CC2 and the two SBU/SWD signals. U6 (TPD4E05U06DQAR, C138714) protects APW SDA, SCL and EN without a clamp to USB-derived 3V3. Place both arrays close to their respective connectors, with short ground returns. System ESD performance still needs hardware testing.
- R10/R11 provide optional 4.7 kΩ I²C pull-up footprints and are **DNP by default**. This avoids loading PSU pull-ups through an unpowered local 3V3 rail. The reference firmware uses internal pull-ups and 400 Hz bit-banging. Validate bus rise time and power-off loading before fitting these resistors or increasing speed.
- Local LED and MCU/regulator symbols and standard switch/crystal symbols replace missing or incorrectly typed legacy definitions. The sheet is A3 to accommodate protection and interface notes.

J2 preserves the reference's custom numbering. In its component-side footprint view at zero rotation, pin 1 is lower right:

```text
6 SCL     5 EN      4 TP2
3 SDA     2 GND     1 TP1
```

This is not the HCTL manufacturer's alternating pin numbering. Preserve the existing footprint and key orientation; verify the mating cable by continuity before substituting a footprint. The same drawing and warning appear on the schematic.

## RP2354A migration, 2026-10-22

U3 is RP2354A (LCSC C41378174), with 2 MB in-package flash, in a 60-pin QFN with exposed ground pad 61. This is a different pinout and footprint from RP2040. The GPIO functions remain GPIO1 = LED, GPIO14 = SDA, GPIO15 = SCL, GPIO25 = MCU_PSU_ON; USB is now pins 52 (D+) and 51 (D−). RUN and SWD retain their logical functions.

The MCU symbol matches all 61 pin names, numbers and positions in ampminer-hat. That project embeds a `bonanzaDisplay:RP2354A` symbol but does not carry its library; its matching source library was copied from `~/Bitcoin/bitaxe/Displays/bonanzaDisplay/` as project-local `rp2354.kicad_sym`. The original ampminer symbol library is also copied locally. The initial migration also copied the external LDO circuit; revision 3 below replaces it with the internal switching regulator.

The MCU QFN footprint and embedded `RP2354A.step` model were copied intact from ampminer-hat. Its SOT-23-5 regulator footprint and STEP file were copied during the initial migration and are retained as unused library assets after U7 removal. No dependency on the source project's location is required to resolve these footprints/models.

## Internal core regulator and BOM, revision 3

The external 1.1 V regulator U7 is removed. RP2354A's internal switching regulator supplies 1V1 using the [Raspberry Pi RP2350 reference circuit](https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf):

| Reference | Function / exact part | JLCPCB / LCSC |
| --- | --- | --- |
| L1 | 3.3 µH Abracon AOTA-B201610S3R3-101-T, polarity marked | C42411119 |
| R14 | 33 Ω Yageo RC0402FR-0733RL, 0402, 1%, 62.5 mW | C138002 |
| C8 | Local 1V1 output bypass: 4.7 µF Murata GRM155R60J475ME47D | C82453 |
| C9 | VREG_VIN input bypass: same 4.7 µF part | C82453 |
| C23 | VREG_AVDD filter capacitor: same 4.7 µF part | C82453 |
| C24 | Additional 1V1 bypass near DVDD pin 23: same 4.7 µF part | C82453 |

VREG_VIN (49) connects to 3V3 and C9. R14 feeds VREG_AVDD (46), with C23 to ground after the resistor. VREG_LX (48) connects to L1 pad 2; L1 pad 1 (the white-dot/marked end) connects to C8 and the 1V1 rail. VREG_FB (50) senses the output at C8. VREG_PGND (47) and the exposed pad (61) remain grounded. Three 100 nF capacitors C6/C7/C20 bypass the DVDD pins, and C24 is the additional output capacitor recommended at the bottom edge of the MCU, away from LX/C8. C10–C16/C21/C22 provide nine 100 nF capacitors for the six IOVDD pins, QSPI_IOVDD, USB_OTP_VDD and ADC_AVDD.

C8/C9/C23/C24 are the exact capacitor MPN used in Raspberry Pi's current R4-S1 minimal design: 0402, X5R, 6.3 V, ±20%. Their nominal 4.7 µF rating must not be interpreted as a guaranteed 4.7 µF at DC bias; retain the specified part and reference layout. The earlier copied fields incorrectly paired a Murata MPN with C368809, which actually identifies Samsung CL05A475KP5NRNC. This mismatch is corrected; the selected Murata ME47D part is C82453.

The project-local L1 footprint follows the [Abracon land pattern](https://abracon.com/datasheets/AOTA-B201610S3R3-101-T.pdf): two 1.0 × 1.6 mm pads, 2.0 mm centre spacing, for a 2.0 × 1.6 mm body. Pad 1 and the orientation dot are on the same side; the dot is represented on both fabrication and silkscreen layers. Pad 1 connects to 1V1, as in Raspberry Pi's regulator layout. Maximum DCR is 140 mΩ; rated current is 2.1 A and saturation current 2.4 A. Do not substitute an unmarked generic 3.3 µH inductor.

PCB layout must follow RP2350 datasheet section 6.3.8: keep CIN/LX/COUT on the MCU side, minimise the switching loop, put C8 between the VREG_VIN/VREG_PGND area as shown in the reference, sense FB from C8, keep AVDD quiet with its own short ground return, and respect copper keepouts under LX. These are schematic placement instructions; PCB placement and routing remain pending.

Firmware should use the internal regulator normally. The earlier instruction to set VREG.HIZ applied only to the external-LDO design and is removed. Do not disable the regulator while it supplies DVDD.

Every purchasable schematic component now has matching `MPN`, `Manufacturer`, `PARTNO`, `LCSC`, and `JLCPCB Part #` fields, checked against the PCB Parts JLCPCB catalogue on 2026-10-22. The [generated BOM](apw-meister-bom.csv) includes 46 component positions: 44 fitted and R10/R11 DNP. TP1/TP2 are bare copper test pads, excluded from the BOM and position files; they have no purchased part number. Power symbols and flags likewise are not purchases. Catalogue stock was not an order-time live check.

The existing ABM8-272-T3 12 MHz crystal, 15 pF load capacitors and 1 kΩ damping resistor also match the [RP2350 hardware design guide](https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf). The BOOTSEL switch retains its 1 kΩ resistor, allowing QSPI_SS to be driven safely with the button held. No external QSPI routing is needed. Firmware must target RP2354A / RP2350 A-series with 2 MB flash, not RP2040.

GPIO14/15 are non-ADC GPIOs; the RP2350 datasheet specifies 3.3 V failsafe operation when unpowered. Keep R10/R11 DNP to avoid a separate backfeed path through the local 3V3 pull-ups. The APW12 3.3 V confirmation applies to the bus; cable, EN bias and bus rise-time checks still require hardware.

## Validation

Standard KiCad ERC: **0 errors, 0 warnings**. Konnect: zero floating wire endpoints, unconnected pins, orphan items, shorts, or symbol-geometry overlaps. Explicit exported-netlist checks pass for all APW12 pins, enable polarity and gate pull-down, USB/CC/SWD/ESD, oscillator, LED, core and IO supplies, LX/inductor/feedback/AVDD-filter wiring, BOOTSEL, reset, unused QSPI pins, DNP pull-ups and all purchasing-field identities. Both ERC power flags on the filtered/output rails are explicit schematic declarations, not physical parts.

Backplane KiCad 10.0.6 at source revision `1c193606eddedbf98f8c9af5100b5e8f79da603d` continues to produce inconsistent net merges. It still reports false power/output conflicts and a USB_DP/ground net merge; its ERC reports **2 errors and 4 warnings**. Standard KiCad's netlist and Konnect's geometric analysis keep these separate. This is the same class of discrepancy reproduced before migration; Backplane ERC is **not passing**. Do not import its incorrect netlist into the PCB. Backups, exported comparisons, render and validation scripts are in ignored `build/rp2354-migration/` and `build/internal-regulator/`.

Remaining physical checks: released EN = off, cable continuity/key orientation, bus rise time and USB-off behavior, USB enumeration, clock startup, regulator startup/load-transient performance and ESD behavior. These checks have not been performed on hardware.

The PCB has not been synchronized or modified. Existing Gerbers, mechanical files, and renders are not manufacturing outputs for this schematic revision.
