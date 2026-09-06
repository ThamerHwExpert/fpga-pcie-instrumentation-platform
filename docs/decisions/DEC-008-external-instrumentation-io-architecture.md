# DEC-008 — External Instrumentation I/O Architecture

Status: PROVISIONAL
Revision: Rev A
Project: fpga-pcie-instrumentation-platform

## Decision

Rev A will use the three 2.5 V HR banks (16, 34, 35) primarily for differential instrumentation I/O.

The external I/O architecture will be partitioned by function rather than allocating all available pairs immediately.

### Bank 34 — timing / trigger bank

Preferred functions:
- EXT_CLK_P/N: 1 differential clock input on an MRCC pair
  - Preferred candidate: V4/W4, IO_L12P/N_T1_MRCC_34
- TRIG_IN_P/N: 1 differential trigger input
- TRIG_OUT_P/N: 1 differential trigger output
- CLK_OUT_P/N: 1 differential clock/output-strobe pair
- Remaining Bank 34 differential pairs reserved for future timing-related I/O

Reason:
Keep time-critical clock/trigger signals concentrated in one bank and preserve clean routing and clocking options.

### Bank 16 — general differential expansion

Baseline allocation target:
- 8 differential general-purpose instrumentation pairs
- Pair direction defined by the implemented FPGA design
- Prefer ordinary differential pairs for data
- Preserve at least one clock-capable pair until the board floorplan is frozen

### Bank 35 — general differential expansion

Baseline allocation target:
- 8 differential general-purpose instrumentation pairs
- Pair direction defined by the implemented FPGA design
- Prefer ordinary differential pairs for data
- Preserve at least one clock-capable pair until the board floorplan is frozen

## Electrical baseline

- VCCO for Banks 16, 34 and 35: 2.5 V
- Baseline differential I/O standard: LVDS_25
- Differential pair polarity should follow FPGA P/N naming unless there is a documented routing reason to swap
- Pair-to-pair and intra-pair routing constraints will be defined after stack-up and connector selection
- External termination strategy remains interface-dependent and must be defined before schematic release

## Connector strategy

The exact connector is not yet locked.

Selection criteria:
- sufficient differential-pair count
- abundant ground contacts for return-path control
- suitable for controlled-impedance routing
- mechanically robust for lab/instrumentation use
- practical breakout on the selected PCB stack-up
- available mating connector/cable ecosystem
- reasonable cost and availability

The connector pinout should interleave grounds with high-speed differential groups where practical.

## Reserved resources

Do not consume all MRCC/SRCC pairs during preliminary pin assignment.

At least:
- one clock-capable pair in Bank 16
- one clock-capable pair in Bank 35

should remain uncommitted until the physical floorplan is reviewed.

## Deferred decisions

The following remain TBD:
- exact connector family and part number
- final number of routed LVDS pairs
- exact trigger electrical levels at the external connector
- whether any pairs require AC coupling
- external termination topology
- maximum supported edge rate / data rate
- ESD/protection strategy
- cable length assumptions
- final bank-to-connector pin mapping
- whether any low-speed 3.3 V control signals share the same connector

## Verification gates

Before final pin lock:
1. Choose connector family and physical location.
2. Freeze approximate FPGA orientation and PCIe-edge placement.
3. Confirm stack-up and routing feasibility.
4. Allocate timing-critical pairs first.
5. Allocate bulk LVDS pairs.
6. Run Vivado I/O DRC.
7. Review schematic connectivity and bank VCCO consistency.
8. Review return paths, termination, protection, and connector ground distribution.

## Rationale

The purpose of this decision is to avoid assigning package pins only because they are electrically available. Final FPGA I/O assignment must also satisfy board floorplan, connector breakout, clocking, signal integrity, and manufacturability constraints.
