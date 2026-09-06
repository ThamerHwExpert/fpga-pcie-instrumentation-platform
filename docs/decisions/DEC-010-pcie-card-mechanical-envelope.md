# DEC-010 — PCIe Card Mechanical Envelope

Status: BASELINE SELECTED
Revision: Rev A
Project: fpga-pcie-instrumentation-platform

## Decision

Rev A will use a **low-profile PCI Express add-in-card mechanical architecture**.

This choice is now the baseline for:
- PCB outline development
- bracket design
- external connector fit checks
- component-height planning
- FPGA / power / connector floorplanning

## PCIe mechanical baseline

Current baseline:
- Form factor: Low-profile PCI Express add-in card
- Electrical interface: PCIe x4
- Card height class: Low-profile
- Maximum PCB height: 68.90 mm
- Board length: TBD
- Slot width: single-slot target
- I/O bracket: low-profile bracket
- Full-height adapter bracket: optional later, not the Rev A baseline

The final outline shall be derived from the applicable PCI Express Card Electromechanical mechanical drawings rather than from a generic custom outline.

## Immediate design consequence

The low-profile choice significantly reduces available front-panel / bracket area.

Therefore the previously selected provisional 68-position MDR external connector must now pass a dedicated fit check before its exact part number is locked.

For the 3M 102-series 68-position right-angle MDR family, the manufacturer drawing shows approximately:
- connector overall span: 63.8 mm
- contact-body span: 57.9 mm
- recommended panel cutout width: approximately 57.9 mm
- panel cutout height: approximately 8.1 mm

These dimensions are close enough to the available low-profile bracket envelope that connector orientation, bracket margins, screw-lock access, and mounting-tab geometry must be checked explicitly.

Therefore:
- 68-position MDR remains a candidate
- it is NOT yet mechanically approved
- no final connector part number shall be released until the bracket drawing is checked

## Preliminary floorplan intent

### Bracket side
- external instrumentation connector
- connector shield/chassis bonding
- ESD / protection elements directly behind connector where practical

### PCIe edge / FPGA region
- FPGA located to keep PCIe GTP traces short
- PCIe REFCLK routed directly from edge connector to selected MGTREFCLK pins
- PERST# routed to Bank 15
- avoid routing high-speed external I/O through the PCIe lane corridor

### Near FPGA
- configuration flash
- local oscillator
- JTAG header or compact debug connector
- local decoupling

### Power region
- PCIe slot power entry
- filtering / protection
- regulators
- sequencing / monitoring

## Mechanical verification gates

Before freezing the external connector:

1. Create a compliant low-profile PCIe board outline.
2. Add the low-profile I/O bracket datum and keep-outs.
3. Add the PCIe x4 edge connector geometry.
4. Import or draw the 68-position MDR candidate using the manufacturer mechanical drawing.
5. Check:
   - bracket cutout fit
   - screw-lock / latch access
   - connector shell clearance
   - PCB mounting-hole clearance
   - cable backshell clearance
   - insertion/removal clearance
6. Check component-height restrictions around the connector and FPGA.
7. Establish a preliminary FPGA placement.
8. Evaluate routing from Banks 16/34/35 to the connector.
9. If the 68-position MDR is too constrained, evaluate:
   - smaller/multiple MDR connectors
   - high-density shielded alternatives
   - Samtec Q-Pairs / twinax solution
   - two-connector architecture

## Open decisions

Still TBD:
- final board length
- exact low-profile bracket drawing used in Altium
- exact external connector part number
- connector orientation
- final connector pinout
- PCB thickness / stack-up
- final component-height plan
- chassis/shield bonding method

## Rationale

Low-profile is preferred as the Rev A baseline because it provides a compact, widely compatible PCIe form factor and forces the design to solve mechanical and routing constraints early.

The tradeoff is reduced bracket and PCB area, so connector choice and floorplanning become design-critical rather than secondary decisions.
