# DEC-009 — External Instrumentation Connector Architecture

Status: PROVISIONAL
Revision: Rev A
Project: fpga-pcie-instrumentation-platform

## Decision

Use one shielded 68-position external I/O connector as the Rev A baseline for the instrumentation interface.

Preferred family for the first mechanical/electrical study:
- 3M MDR 68-position boardmount / cable system

This is not yet a released part-number selection. The exact PCB connector orientation, backshell/cable assembly, mounting hardware, and mating cable must be verified against the PCIe bracket and board mechanical envelope before schematic lock.

## Why this family is the current baseline

The Rev A interface currently targets approximately:
- 20 differential pairs total
  - 4 timing/trigger pairs in Bank 34
  - 8 general LVDS pairs in Bank 16
  - 8 general LVDS pairs in Bank 35
- substantial ground allocation between/around high-speed signal groups
- a small number of reserved or low-speed contacts

A 68-position connector gives enough contact count for this architecture without forcing every contact to carry a signal.

The 3M MDR cable family is available in 68-position versions and is intended for shielded external I/O. 3M specifies 100-ohm balanced differential use for the pleated-foil MDR cable family and provides metal-shell / thumbscrew or latch options.

## Preliminary contact budget

Do not freeze exact contact numbers yet.

Target budget:
- 40 contacts: 20 differential signal pairs
- 24 contacts: signal-return / ground
- 4 contacts: reserved / low-speed / future use

Total: 68 contacts

This is a budgeting model only. The final pinout may move some reserved contacts to ground if signal-integrity review shows that is preferable.

## Functional partition

### Timing / trigger group — Bank 34
- EXT_CLK_P/N
- TRIG_IN_P/N
- TRIG_OUT_P/N
- CLK_OUT_P/N

### General differential group A — Bank 16
- 8 differential pairs

### General differential group B — Bank 35
- 8 differential pairs

## Grounding strategy

The external connector must not be treated as 40 signal pins plus a distant ground cluster.

Preferred approach:
- distribute ground contacts through the connector
- place grounds adjacent to or between high-speed differential groups where the connector geometry allows
- connect cable shield/chassis appropriately at the external interface
- keep signal-reference return paths distinct from arbitrary long PCB ground detours
- decide chassis-to-digital-ground coupling strategy during EMC/mechanical design

The exact connector contact map must be reviewed with the cable construction, not only with the PCB connector pinout.

## Electrical assumptions

Baseline:
- FPGA Banks 16/34/35: 2.5 V
- FPGA differential standard: LVDS_25
- target differential impedance: 100 ohm
- no assumption yet that all pairs support the maximum electrical capability of the connector/cable
- actual supported data rate will be determined by FPGA I/O behavior, board stack-up, routing, connector/cable insertion loss, receiver margin, and validation

## Alternatives considered

### Samtec Q Pairs / HQDP
Technically attractive:
- 100-ohm differential routing
- high-speed twinax cable
- high pair density
- screw-mount options

Reason not selected as the default external front-panel baseline yet:
- it is a denser high-speed board/cable system and may be less convenient than MDR for a conventional lab-instrumentation front-panel interface
- mechanical accessibility, bracket integration, cable cost, and sourcing should be evaluated before using it as the primary user-facing connector

Keep Samtec Q Pairs as the performance-oriented alternative if the eventual bandwidth requirement exceeds what is practical with the MDR implementation.

## Not yet locked

- exact 3M MDR PCB connector part number
- right-angle versus vertical orientation
- bracket cutout
- cable assembly part number and length
- shield termination
- exact connector contact assignment
- final number of ground contacts
- low-speed/reserved contact use
- ESD protection topology
- common-mode filtering, if any
- final maximum supported data rate

## Verification gates before final selection

1. Confirm available PCIe bracket/board edge space.
2. Select candidate 68-position boardmount connector orientation.
3. Obtain manufacturer drawing and STEP model.
4. Place connector and FPGA approximately in the PCB floorplan.
5. Check whether Banks 16/34/35 can route cleanly to the connector.
6. Define connector contact-to-pair grouping and ground distribution.
7. Check cable assembly availability and mating hardware.
8. Review impedance, insertion loss, skew, shielding, and return paths.
9. Freeze the exact manufacturer part numbers only after the mechanical and routing study.

## Current engineering conclusion

Proceed with a 68-position MDR-style shielded external connector as the Rev A mechanical/electrical baseline, but keep the exact part number provisional until the board floorplan confirms it.
