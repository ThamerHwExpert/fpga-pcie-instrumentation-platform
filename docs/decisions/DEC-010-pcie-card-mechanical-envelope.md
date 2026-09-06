# DEC-010 — PCIe Card Mechanical Envelope

Status: PROVISIONAL
Revision: Rev A
Project: fpga-pcie-instrumentation-platform

## Decision

Before selecting the exact external connector part number or final FPGA-to-connector pin mapping, Rev A will define and verify the PCIe card mechanical envelope.

The current design shall use a standard PCI Express add-in-card mechanical architecture.

The exact card height class is not yet locked:
- full-height
- low-profile

This must be decided before the external connector and bracket geometry are frozen.

## Mechanical items to define

### PCIe edge
- PCIe x4 electrical interface
- connector keying and edge-finger geometry per PCIe add-in-card requirements
- board thickness and gold-finger stack-up to be defined with the PCB manufacturer

### Bracket
- bracket height class: TBD
- external connector must be mechanically accessible through the bracket
- connector retention hardware must not interfere with bracket fastening
- cable insertion/removal forces must be supported by the mechanical structure, not only by solder joints

### Board outline
The final board outline is TBD.

Before schematic release:
1. establish PCIe edge position
2. establish bracket datum
3. choose full-height or low-profile envelope
4. reserve keep-outs around the edge connector
5. reserve the external I/O connector region
6. reserve FPGA and power-conversion regions
7. check component-height restrictions
8. check enclosure/chassis interference

## External connector study

Current provisional architecture:
- one shielded 68-position external instrumentation connector
- 3M MDR family is the preferred first candidate
- exact part number and orientation remain TBD

The connector shall not be locked until:
- bracket height is chosen
- board outline is established
- connector mechanical drawing is checked against the bracket opening
- cable backshell and latch/thumbscrew access are verified
- FPGA-to-connector routing feasibility is reviewed

## Preliminary floorplan intent

Preferred functional zoning:

PCIe bracket side:
- external instrumentation connector
- ESD / protection / optional filtering close to connector

PCIe edge side:
- x4 PCIe edge connector
- PCIe REFCLK and PERST# routing

Central high-speed region:
- FPGA
- short PCIe GTP routing between FPGA and edge connector

Near FPGA:
- configuration flash
- JTAG access
- local oscillator
- FPGA decoupling

Power region:
- slot power entry
- input filtering/protection
- FPGA rail regulators
- sequencing / supervision

## Mechanical verification gates

Before exact connector part-number lock:
1. Decide full-height versus low-profile PCIe card.
2. Create preliminary board outline in Altium.
3. Add PCIe edge connector geometry and bracket datum.
4. Import candidate connector STEP model.
5. Verify bracket cutout and mating-cable clearance.
6. Check FPGA placement relative to PCIe edge and external connector.
7. Confirm major routing channels.
8. Only then freeze connector part number and detailed LVDS package-pin mapping.

## Open decisions

- full-height or low-profile
- exact board length
- exact board thickness
- bracket type
- exact connector part number
- connector right-angle versus vertical orientation
- external cable exit direction
- mounting/retention hardware
- component-height limits
- final chassis/shield bonding method

## Rationale

The external connector, FPGA orientation, PCIe edge, and board outline are strongly coupled. Freezing any one of them independently can create avoidable routing, mechanical, EMC, or manufacturability problems.

Therefore the mechanical envelope is the next design gate before final external-I/O pin allocation.
