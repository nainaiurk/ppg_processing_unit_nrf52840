# Design review — 2026-11-09

Reviewed the saved schematic netlist and PCB, ERC, schematic/PCB parity, and DRC after an in-memory zone refill. Original design files were not modified. Expanded checks were run on `expanded.kicad_pcb` and `expanded.kicad_pro`, which are review copies, not production files.

## Highest-priority findings

1. **Battery charger incompatible with the stated 4.2 V maximum-charge cell.** U3 is BQ24232HARGTR. The BQ24232HA electrical-characteristics table specifies battery regulation of 4.25 V minimum, 4.31 V typical, and 4.35 V maximum. Select a charger designed for the actual battery's charge voltage; check pinout, charge current, input limit, and surrounding components before substitution. Do not charge the stated 4.2 V cell with this configuration. Source: https://www.ti.com/lit/ds/symlink/bq24232ha.pdf (page 7).

2. **U6 has no DC supply connection.** Both schematic and PCB place U6 pin 8 (VDD) and C16 pin 2 on `Net-(U6-VDD)`, with no source on that net. C16 pin 1 is GND. Connect this supply net to the intended regulated supply in the schematic and PCB, and verify device voltage/decoupling requirements. Ordinary DRC passes because the existing two-terminal net is completely routed.

3. **Solder-mask openings bridge different nets.** The project disables `solder_mask_bridge`. Enabling it on a review copy reports 52 violations. Examples include SYS_PWR routing beside U4 GND pad 4, U6 MISO pad 6 beside MOSI routing, J3 adjacent signal pads, and U3 perimeter pads versus its exposed GND pad. Several footprints use +0.1016 mm mask expansion. Review the actual mask artwork and adjust expansion/routing to the manufacturer's capabilities. These are assembly shorting risks, not proof of existing copper shorts; some gang mask openings may be intentional.

## Other findings

- **Battery temperature is not measured.** R4 is a fixed 10 kOhm resistor from U3 TS to GND, and J1 is a two-pin battery connector. This simulates an acceptable temperature rather than monitoring the cell. If temperature-qualified charging is needed, provide an appropriate cell thermistor connection. Charger die thermal protection does not measure cell temperature.
- **Eight incomplete thermal-relief checks.** One spoke is present where the zone requires two, including U4 GND, C16 GND, several U5 central GND lands, and U5 antenna pads 57 and 60. These are not open circuits. Review direct/short ground connections, particularly around the regulator and antenna. `starved_thermal` is currently disabled.
- **U5 schematic and footprint numbering differ.** The schematic still defines separate GND pins 56–67, whereas the PCB footprint uses twelve physical pads numbered 56 and antenna pads 57–60. Parity reports missing pads 61–67. All affected schematic pins are GND, so this does not establish a wrong signal connection, but the symbol should be reconciled with the footprint before later PCB updates.
- **TP1/TP2 are absent from the PCB.** These are SDA/SCL test points in the schematic. Add them if intended or mark/remove them appropriately.
- **Library/hierarchy cleanup is needed.** ERC reports 214 items; many are missing library references, unspecified imported pin types, and unused hierarchy labels, not 214 independent functional faults. BAR_CS/BAR_INT, MIC_CLK/MIC_OUT, and HAP_EN/HAP_TRIG currently have only U5 on their nets; if those functions are intended, their peripherals are absent. Many footprint parity warnings are only a missing `Vault:` library prefix.

## Interpretation of additional checks

Current configured DRC: 0 violations and 0 unconnected items. Schematic parity: 45 items. Expanded DRC: 52 solder-mask bridges, 8 incomplete thermals, and 22 padstack warnings. The latter include 14 IMU U2 pads whose built-in paste is suppressed but replaced by 14 custom B.Paste shapes; do not remove those shapes or assume U2 has no stencil openings. P1 has suppressed paste and may intentionally be a bare contact interface.

This review does not validate final enclosure RF performance, battery capacity/current suitability, firmware initialization, external sensor-board voltages, or fabrication/assembly process limits. The existing NINA corner-placement RF compromise remains unchanged.

Machine-readable evidence: `netlist.xml`, `erc.json`, `drc.json`, and `expanded-drc.json` in this directory.
