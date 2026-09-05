DEC-005 — Clock Architecture

Status: Accepted — baseline architecture
Revision: A
Date: 2026-09-05


1. Objective

Define the clock architecture for Rev A of the Open FPGA PCIe
Instrumentation Platform.

The design contains three fundamentally different timing functions:

1. PCIe reference clock
2. Local FPGA system clock
3. External instrumentation clock

These functions shall remain architecturally separate unless a later design
decision provides a specific reason to combine them.


2. Baseline Decision

Rev A shall use:

- the host PCIe slot 100 MHz differential reference clock for the PCIe GTP/core
- synchronous PCIe clocking
- a separate local 100 MHz oscillator for general FPGA/instrumentation logic
- an optional external instrumentation clock input
- dedicated FPGA clock-capable pins for local/external fabric clocks
- explicit clock-domain crossing between unrelated clock domains
- MMCM/PLL resources only where frequency synthesis, phase control, or clock
  conditioning is actually required

Exact oscillator part numbers, electrical standards, and FPGA package pins
remain subject to pin planning and component selection.


3. Clock-Domain Overview

Conceptual architecture:

                         PCIe HOST
                            |
                     100 MHz REFCLK+/-
                            |
                            v
                  Dedicated MGT REFCLK
                            |
                            v
                     GTP / PCIe Core
                            |
                            v
                    PCIe User Clock(s)
                            |
                      CDC boundaries
                            |
          +-----------------+-----------------+
          |                                   |
          v                                   v
  Local FPGA Logic                    Instrumentation Logic
          ^                                   ^
          |                                   |
   Local 100 MHz OSC                   External Clock Input
          |                                   |
          +-------------- FPGA ---------------+


The PCIe reference clock shall not be treated as the board's precision
instrumentation clock.


4. PCIe Reference Clock

4.1 Source

The PCIe reference clock shall be supplied by the host PCIe slot.

Nominal frequency:

100 MHz

The PCIe endpoint shall use synchronous clocking.

This is the normal architecture for an add-in card and avoids introducing an
independent PCIe reference clock and asynchronous PCIe clock mode.


4.2 FPGA Interface

The differential PCIe reference clock shall connect to a dedicated Artix-7
GTP reference-clock input associated with the selected PCIe transceiver quad.

The FPGA implementation shall use the dedicated GTP reference-clock input
buffer primitive required by the 7-Series PCIe design, such as IBUFDS_GTE2 as
generated/recommended by the AMD PCIe implementation.

The PCIe reference clock shall NOT:

- enter through arbitrary general-purpose FPGA I/O
- be routed through unnecessary external logic
- be used as a substitute for an instrumentation-grade local clock


4.3 Spread-Spectrum Clocking

Commercial PCIe hosts commonly provide a 100 MHz Spread Spectrum Clock (SSC).

Therefore the design shall assume that the incoming PCIe reference clock may
contain PCIe-compliant spread-spectrum modulation.

Consequence:

PCIe REFCLK is appropriate for the PCIe link but shall not be assumed to be a
low-drift or precision timing reference for measurement functions.


4.4 Routing

PCIe REFCLK routing shall:

- use controlled differential impedance
- maintain a continuous reference return path
- minimize unnecessary vias
- avoid stubs
- maintain suitable separation from high-speed aggressors
- route directly from the edge connector to the selected MGTREFCLK pins

No ordinary oscilloscope test point shall be placed directly on the
high-speed differential REFCLK pair because the added stub can degrade signal
integrity.

If validation requires clock observation, the measurement strategy shall be
defined specifically for the high-speed differential interface.


5. Local FPGA System Clock

5.1 Purpose

A dedicated local oscillator shall provide a stable board-level fabric clock
independent of PCIe link state.

The clock supports:

- startup logic
- control/status functions
- instrumentation logic
- debug logic
- test logic
- clock generation through MMCM/PLL when required


5.2 Frequency

Baseline frequency:

100 MHz

Reason:

- common FPGA development frequency
- convenient 10 ns period
- suitable for moderate fabric/control logic
- straightforward derivation of multiple internal rates
- does not force the PCIe clock to be reused for unrelated logic
- matches the first-pass XPE model already used for power estimation


5.3 Electrical Standard

The local oscillator electrical standard remains TBD until FPGA I/O bank
planning is complete.

Possible implementation:

- LVCMOS18
- LVCMOS25
- LVCMOS33

The oscillator supply voltage shall be selected together with the destination
FPGA bank VCCO.

The design shall not create an additional power rail solely to support a
clock oscillator unless there is a clear benefit.


5.4 FPGA Pin Selection

The local oscillator shall connect to a clock-capable FPGA input.

Preferred hierarchy:

1. MRCC/SRCC clock-capable input appropriate for the intended clock region
2. route through the normal clock input buffer
3. use BUFG/global clock resources as needed

A random user-I/O pin shall not be selected merely because it is convenient
on the schematic.

Exact package pins remain TBD until the XC7A35T-FGG484 pin plan is created.


6. Local Clock MMCM / PLL Usage

The baseline 100 MHz local oscillator does not automatically require an MMCM.

If the FPGA logic can operate directly at 100 MHz:

Local OSC -> Clock-capable input -> BUFG -> fabric

is preferred.

An MMCM/PLL shall be introduced only when needed for:

- frequency multiplication/division
- phase shifting
- duty-cycle control
- jitter filtering appropriate to the application
- generation of related synchronous clocks

Unnecessary MMCM/PLL use increases:

- power
- resource usage
- reset/lock dependencies
- verification complexity


7. PCIe Internal Clocking

The AMD PCIe core generates/uses its own required internal/user clocks based on
the selected PCIe architecture.

The board-level design shall not manually recreate those clocks.

The generated PCIe IP clock architecture and constraints shall be preserved.

Clock outputs from the PCIe core may be used by PCIe-facing logic as intended
by the generated design.

Any transfer between PCIe user-clock logic and local-clock logic shall be
treated as a clock-domain crossing.


8. Clock-Domain Crossing

The board architecture intentionally permits asynchronous clock domains.

Potential domains include:

- PCIE_USER_CLK
- SYS_CLK_100
- EXT_CLK
- internally generated derivative clocks

No unsynchronized control or multi-bit data path shall cross these domains
directly.

Appropriate CDC structures shall be used, for example:

Single-bit status/control:
- two-stage synchronizer where appropriate

Pulses:
- pulse synchronizer / toggle synchronizer / handshake

Multi-bit control:
- handshake-based transfer

Streaming/data:
- asynchronous FIFO

Clock-domain relationships shall be declared correctly in FPGA constraints.

CDC verification shall be part of FPGA design review.


9. External Instrumentation Clock

9.1 Purpose

Rev A shall reserve an external timing/clock input for instrumentation use.

Possible applications include:

- synchronization to laboratory equipment
- external acquisition timing
- trigger-aligned sampling logic
- synchronization between multiple boards


9.2 Baseline Architecture

Concept:

External connector
       |
       v
Protection / termination
       |
       v
Input conditioning
       |
       v
FPGA clock-capable input
       |
       v
Optional MMCM / BUFG
       |
       v
Instrumentation logic


9.3 Electrical Standard

The final external clock electrical interface remains TBD.

Options to evaluate later include:

- single-ended 50-ohm clock input
- LVDS differential input

The decision shall be based on:

- expected laboratory source
- cable length
- maximum frequency
- required jitter performance
- connector selection
- termination requirements
- ESD/protection requirements
- available FPGA bank voltage

The interface shall not be defined as generic GPIO.


10. External Clock Failure Behavior

The board shall remain:

- configurable
- JTAG-accessible
- PCIe-capable
- locally debuggable

when no external instrumentation clock is connected.

External clock presence shall therefore not be required for basic board boot or
PCIe enumeration.

Instrumentation logic that depends on EXT_CLK shall implement an appropriate
missing-clock/reset strategy.


11. Clock Reset / Lock Behavior

Any logic using MMCM/PLL-generated clocks shall remain in reset until the
corresponding clock resource reports a stable LOCKED state.

Concept:

Clock source
    |
    v
MMCM / PLL
    |
    +----> generated clock
    |
    +----> LOCKED
              |
              v
        reset-release logic


Reset deassertion into each synchronous clock domain shall be synchronized to
that domain.

The PCIe core reset architecture shall continue to follow AMD PCIe IP
requirements rather than being replaced by generic application reset logic.


12. Configuration Clock

FPGA configuration CCLK is a fourth clock-like signal but is functionally part
of the configuration subsystem rather than the normal application clock tree.

Rev A uses the FPGA internal configuration oscillator to generate CCLK in
Master SPI mode.

CCLK shall therefore remain separate from:

- PCIe REFCLK
- local SYS_CLK
- external instrumentation clock


13. Clock Power Estimation

The current first-pass XPE model includes:

Local fabric clock:
- SYS_CLK = 100 MHz
- Global clock
- provisional fanout = 10,419
- clock-buffer enable = 100 percent
- slice-clock enable = 100 percent

XPE result for the entered general clock network:

CLOCK power = 0.088 W

The PCIe Transceiver Configuration wizard also added required PCIe clocking
resources, including MMCM contribution in the current XPE model.

Current XPE MMCM power:

0.108 W

These are pre-implementation estimates.

The final Vivado implementation shall replace estimated fanout and generated
clock usage with implementation-derived values.


14. DFT / Validation

Clock validation shall include:

PCIe REFCLK:
- verify presence indirectly or with an appropriate high-speed measurement
  method
- verify PCIe core PLL/MMCM lock
- verify successful link training

Local SYS_CLK:
- verify oscillator supply
- verify frequency
- verify startup
- verify FPGA clock detection/operation

External clock:
- verify threshold/interface behavior
- verify frequency range
- verify jitter/quality as required by application
- verify missing-clock behavior

MMCM/PLL:
- verify LOCKED behavior
- verify generated frequencies
- verify reset release


15. PCB Requirements

Clock routing shall follow these baseline rules:

- provide an uninterrupted reference return path
- avoid routing across plane splits
- minimize vias
- avoid unnecessary stubs
- keep sensitive clocks away from switching-regulator nodes
- keep PCIe REFCLK separated from high-speed aggressors
- place the local oscillator reasonably close to its FPGA clock input
- place required oscillator decoupling at the oscillator supply pins
- treat external clock conditioning and termination as part of the connector
  interface, not as an afterthought


16. Open Items

The following remain TBD:

1. Exact local 100 MHz oscillator part number
2. Local oscillator supply voltage
3. Exact local clock FPGA pin
4. Exact PCIe MGTREFCLK pins / GTP quad
5. External clock connector type
6. External clock single-ended vs differential architecture
7. External clock voltage / common-mode standard
8. External clock maximum supported frequency
9. External clock termination
10. External clock protection
11. Whether any application MMCM/PLL beyond PCIe-generated resources is needed
12. Final CDC architecture
13. Final clock constraints


17. Consequences

This decision establishes three independent board-level clock roles:

A. PCIe REFCLK
   - 100 MHz
   - host supplied
   - differential
   - dedicated GTP reference clock
   - synchronous PCIe architecture

B. Local SYS_CLK
   - 100 MHz baseline
   - board oscillator
   - general FPGA/instrumentation logic
   - clock-capable FPGA I/O

C. EXT_CLK
   - optional external instrumentation timing
   - electrical standard TBD
   - clock-capable FPGA I/O

The next FPGA pin-planning step must reserve appropriate resources for all
three clock functions.


18. References

Primary references:

- AMD PG054 — 7 Series FPGAs Integrated Block for PCI Express
- AMD UG472 — 7 Series FPGAs Clocking Resources User Guide
- AMD UG482 — 7 Series FPGAs GTP Transceivers User Guide
- AMD UG475 — 7 Series FPGAs Packaging and Pinout Product Specification
