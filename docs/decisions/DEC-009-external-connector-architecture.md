# DEC-009 — External Instrumentation Connector Architecture

Status: BASELINE SELECTED
Revision: Rev A
Project: fpga-pcie-instrumentation-platform

## Decision

Rev A will use a single 68-position TE Connectivity CHAMP 0.8 mm right-angle external connector as the baseline instrumentation interface.

Selected baseline candidate:
- Manufacturer: TE Connectivity
- MPN: 5796055-1
- Interface family: CHAMP 0.8 mm
- Positions: 68
- Orientation: Right-angle
- Mounting: Through-hole, board/panel mount
- Shielded: Yes
- Recommended PCB thickness: approximately 1.6 mm

The part remains subject to final sourcing and production-BOM review, but it is now the Rev A mechanical/electrical baseline.

## Why this connector was selected

The low-profile PCIe mechanical envelope materially constrains the external connector.

The previously studied 68-position 3M MDR connector was rejected for Rev A low-profile use because its required panel-cutout / mounting span is too large for a clean standard low-profile bracket implementation.

The TE 5796055-1 provides:
- 68 contacts in a much more compact front-panel envelope
- right-angle PCB mounting
- shielded external interface
- 0.8 mm contact pitch
- compatibility with approximately 1.6 mm PCB thickness
- significantly better low-profile bracket fit margin

## Footprint audit

The imported CAD footprint was audited against the TE mechanical drawing before acceptance.

Verified signal-pad geometry:
- 68 signal contacts
- signal pad: approximately 1.00 mm diameter
- signal drill: approximately 0.65 mm
- horizontal contact pitch: 0.80 mm
- four-row staggered geometry

Representative verified signal coordinates:
- Pad 1:  X =  0.00 mm, Y =  0.00 mm
- Pad 2:  X = -0.80 mm, Y = -1.15 mm
- Pad 35: X = +0.40 mm, Y = -2.35 mm
- Pad 36: X = -0.40 mm, Y = -3.50 mm

Verified mechanical features:
- MH1: X =   3.175 mm, Y = -2.25 mm
- MH2: X = -29.175 mm, Y = -2.25 mm
- MH3: X =   7.95 mm,  Y = -4.88 mm
- MH4: X = -33.95 mm,  Y = -4.88 mm

The audited footprint is maintained in the project PCB library as:
TE_5796055-1_CHAMP68_RA

## Mechanical-layer convention

For the project-controlled footprint:
- M13: assembly / component body / 3D-body reference
- M15: courtyard

A manufacturer-derived PCB-edge datum is also included in the footprint for mechanical placement.

## Low-profile PCIe fit study

The connector was placed on the Rev A low-profile card mechanical study at:

- X = 6.30 mm
- Y = 19.35 mm
- Rotation = 270 degrees

This placement aligns the connector PCB-edge datum with the bracket-side PCB datum and centers the connector approximately within the usable low-profile bracket opening.

Result:
- mechanically acceptable as the Rev A low-profile baseline
- substantially better fit margin than the rejected MDR68 candidate
- final bracket cutout and sheet-metal drawing still required before release

## Preliminary contact budget

Target use:
- 20 differential pairs = 40 contacts
- remaining contacts primarily assigned to signal return / ground
- limited reserved / low-speed contacts as required

The exact connector pinout is not yet frozen.

## Functional partition

### Bank 34 — timing / trigger
- EXT_CLK_P/N
- TRIG_IN_P/N
- TRIG_OUT_P/N
- CLK_OUT_P/N

### Bank 16 — differential instrumentation group A
- target: 8 differential pairs

### Bank 35 — differential instrumentation group B
- target: 8 differential pairs

## Electrical baseline

- FPGA Banks 16/34/35: 2.5 V
- baseline FPGA differential standard: LVDS_25
- target differential impedance: 100 ohm
- exact termination and protection topology: TBD
- supported cable/data rate: TBD and must be validated

## Grounding / shielding intent

The final pinout shall:
- distribute ground contacts through the connector
- maintain controlled return paths around differential groups
- avoid a single remote ground cluster
- provide an intentional cable-shield / chassis-bonding strategy
- keep protection components physically close to the external connector where practical

## Rejected alternative

3M MDR 68-position right-angle connector:
- retained in the project library as a studied alternative
- rejected as the Rev A low-profile baseline because its mechanical panel-span requirement is too large for a clean standard low-profile implementation

## Remaining design gates

Before schematic release:
1. Create the final low-profile bracket cutout drawing.
2. Freeze the exact board outline and bracket datum.
3. Finalize connector placement.
4. Freeze contact-to-signal assignment.
5. Allocate final FPGA Bank 16/34/35 package pins.
6. Define shield/chassis grounding.
7. Define ESD / surge / filtering strategy.
8. Review routing escape and return-current paths.
9. Verify connector and cable sourcing.
10. Perform final mechanical / 3D interference review.

## Current engineering conclusion

TE Connectivity 5796055-1 is accepted as the Rev A external instrumentation connector baseline for the low-profile PCIe card.

The next design step is to freeze the actual low-profile PCB outline and bracket geometry around this connector and the PCIe edge interface.
