DEC-007 — FPGA I/O Bank Architecture

Status: Accepted — baseline architecture
Revision: A
Date: 2026-09-05


1. Objective

Define the FPGA I/O-bank voltage architecture and functional allocation for
Rev A of the Open FPGA PCIe Instrumentation Platform.

This decision is intended to prevent several common FPGA-board mistakes:

- assigning FPGA pins before defining bank voltages
- mixing incompatible I/O standards in one bank
- creating unnecessary VCCO rails
- placing clocks on non-clock-capable pins
- consuming configuration pins accidentally
- discovering late in layout that the chosen bank cannot support the intended
  electrical standard
- treating all FPGA user I/O as interchangeable

Exact package pins are NOT selected in this document.

Pin-level assignment will be performed in the next design stage using the
XC7A35T-FGG484 package pinout and Vivado pin planning.


2. Device Bank Structure

Baseline FPGA:

AMD/Xilinx XC7A35T-FGG484

For this device/package combination, the relevant fully bonded HR I/O banks are:

- Bank 14
- Bank 15
- Bank 16
- Bank 34
- Bank 35

The package provides 250 user I/O pins total.

The GTP transceiver quad is separate from the SelectIO bank architecture and
is handled by the PCIe/GTP pin plan.

All Rev A SelectIO banks are HR banks.

Consequences of HR-bank architecture include:

- support for 3.3 V and 2.5 V LVCMOS standards
- support for LVDS_25
- no need to design around HP-bank-only voltage limitations


3. Baseline VCCO Architecture

Rev A shall use only two user-I/O VCCO levels:

3.3 V
2.5 V

Baseline allocation:

Bank       VCCO       Primary Role
-----------------------------------------------
Bank 0     3.3 V      Configuration domain
Bank 14    3.3 V      QSPI configuration / low-speed configuration-related I/O
Bank 15    3.3 V      Board control, debug, PERST#, local clock, low-speed I/O
Bank 16    2.5 V      Differential instrumentation / expansion
Bank 34    2.5 V      Differential instrumentation / expansion
Bank 35    2.5 V      Differential instrumentation / expansion

This allocation is accepted as the Rev A baseline but remains subject to
pin-level feasibility checking in Vivado.


4. Why Only 3.3 V and 2.5 V VCCO

The architecture intentionally avoids creating 1.8 V user-I/O banks in Rev A.

Reason:

- 3.3 V is already required by the configuration architecture
- 3.3 V is convenient for low-speed board-management interfaces
- 2.5 V enables native LVDS_25 operation in Artix-7 HR banks
- two VCCO domains simplify power conversion, sequencing, BOM, validation,
  FPGA pin planning, and board bring-up
- no current Rev A requirement requires a 1.8 V external digital interface

The existing 1.8 V VCCAUX supply shall therefore not automatically be reused
as a VCCO rail.

If a later interface genuinely requires 1.8 V, the bank architecture shall be
reopened rather than silently introducing an incompatible I/O standard.


5. Bank 0 — Configuration Domain

VCCO_0 = 3.3 V

Bank 0 is part of the FPGA configuration architecture.

From DEC-004:

- CFGBVS is tied to VCCO_0
- the configuration architecture uses 3.3 V
- JTAG reference voltage is 3.3 V

Bank 0 shall not be treated as a general-purpose expansion bank.

Configuration-specific pins and dedicated functions shall follow AMD pinout
requirements exactly.


6. Bank 14 — Configuration / Controlled 3.3 V I/O

VCCO_14 = 3.3 V

Primary role:

- Master SPI x4 configuration interface
- configuration-related status/control functions
- limited low-speed user I/O after configuration if required

The SPI x4 configuration interface uses Bank 14 configuration-capable pins.

Design rule:

Do not assign critical external instrumentation interfaces to configuration
pins simply because those pins become available as user I/O after
configuration.

Reason:

- configuration ownership exists during startup
- external loading can interfere with configuration
- external devices can drive the pins before FPGA configuration completes
- debugging becomes harder

Any dual-use configuration pin shall be explicitly documented as dual-use.


7. Bank 15 — 3.3 V Board-Control Bank

VCCO_15 = 3.3 V

Bank 15 is the preferred location for low-speed board-level interfaces.

Planned functions include:

- PCIe PERST# input
- local 100 MHz oscillator input
- USB-UART TX/RX
- low-speed SPI
- I2C
- board status/control
- trigger/status GPIO where 3.3 V logic is appropriate
- LEDs or control outputs through suitable buffering where required


8. PERST# Allocation

Preferred bank:

Bank 15, VCCO = 3.3 V

Reason:

PCIe PERST# is a 3.3 V sideband signal and Artix-7 HR banks support 3.3 V
LVCMOS operation.

Preferred architecture:

PCIe PERST#
     |
     +---- optional small series resistor footprint
     |
     +----> Bank 15 FPGA input

This direct architecture remains subject to final verification of:

- exact PCIe PERST# electrical limits
- Artix-7 input limits
- selected I/O standard
- power-off/back-power behavior
- FPGA pin availability

If any of those checks fail, a buffer/level translator shall be introduced.

A translator shall NOT be added merely because "PCIe reset usually has a
translator" in another reference design.


9. Local 100 MHz System Clock Allocation

Preferred bank:

Bank 15, VCCO = 3.3 V

Preferred I/O standard:

LVCMOS33

Required pin type:

clock-capable MRCC or SRCC input

The exact package pin shall be selected only after:

- reviewing Bank 15 clock-capable pins
- checking the intended clock region
- checking placement relative to the oscillator
- verifying Vivado clock routing

The oscillator shall not be assigned to an arbitrary GPIO.


10. Bank 16 — Differential Instrumentation Bank A

VCCO_16 = 2.5 V

Primary use:

- LVDS_25 input/output pairs
- differential timing signals
- high-speed digital expansion
- optional single-ended LVCMOS25 control signals associated with that
  connector

No external VREF is required for the baseline LVDS_25 / LVCMOS25 use case.

Clock-capable differential pairs in this bank shall be reserved preferentially
for timing-sensitive external interfaces.


11. Bank 34 — Differential Instrumentation Bank B

VCCO_34 = 2.5 V

Primary use:

- LVDS_25 instrumentation I/O
- differential expansion
- synchronous digital interfaces
- clock/trigger distribution where appropriate

This bank is intended to provide a second independent 2.5 V external-I/O
resource group.


12. Bank 35 — Differential Instrumentation Bank C

VCCO_35 = 2.5 V

Primary use:

- additional LVDS_25 pairs
- future daughtercard / expansion interface
- external differential clock or trigger if pin planning makes Bank 35 the
  preferred location

The exact split among Banks 16, 34, and 35 shall be driven by:

- connector placement
- clock-capable pin locations
- differential-pair locations
- FPGA internal clock-region reach
- PCB routing
- signal grouping

The three banks share the same 2.5 V nominal VCCO but shall still be treated
as separate FPGA I/O-bank resources.


13. External Clock Allocation

Baseline electrical intent:

Differential external instrumentation clock

Preferred electrical standard:

LVDS_25

Preferred bank class:

one of Banks 16, 34, or 35

Required FPGA pin type:

differential clock-capable pair where appropriate

The final bank is NOT locked in this decision.

Selection shall be based on:

- package pinout
- connector placement
- MMCM/CMT accessibility
- desired clock region
- PCB routing

This is deliberately left to pin planning rather than guessed here.


14. External Trigger Allocation

The external trigger connector will not connect directly to arbitrary FPGA
logic levels.

The external trigger interface shall include an electrical front end defined
from the intended laboratory standard.

Possible front-end outputs to the FPGA include:

Option A:
3.3 V single-ended conditioned trigger -> Bank 15

Option B:
2.5 V differential conditioned trigger -> Bank 16/34/35

The external connector voltage range and protection shall be defined in a
later interface decision.

Therefore the trigger FPGA bank is currently provisional.


15. Differential I/O Standard

Baseline differential user I/O standard:

LVDS_25

Reason:

- natively supported by Artix-7 HR banks
- 2.5 V VCCO
- appropriate for point-to-point differential digital interfaces
- avoids adding another user-I/O voltage domain

This does NOT mean every differential pair shall automatically enable internal
differential termination.

DIFF_TERM use shall be decided per interface based on:

- receiver topology
- external termination
- direction
- signal rate
- impedance environment


16. Single-Ended I/O Standards

Baseline standards:

3.3 V banks:
- LVCMOS33

2.5 V banks:
- LVCMOS25

Drive strength and slew rate shall not be left at arbitrary defaults.

For each output class, final FPGA constraints shall explicitly define:

- IOSTANDARD
- DRIVE where applicable
- SLEW where applicable

High drive strength and FAST slew shall not be used unless required by the
signal-integrity analysis.


17. VREF Strategy

The baseline Rev A interfaces do not require an external VREF-based I/O
standard such as SSTL or HSTL.

Therefore:

- no external VREF rail is required for Banks 14, 15, 16, 34, or 35
- VREF-capable package pins shall not automatically be tied to a reference
  voltage
- their final use shall follow AMD pin rules and the final pin plan

If a future interface requires SSTL/HSTL or another VREF-dependent standard,
the affected bank shall be reviewed as a new architectural decision.


18. Bank-Voltage Compatibility Rule

All output standards within one bank must be compatible with that bank's VCCO.

Inputs can have additional standard-specific rules.

No schematic pin assignment is accepted until the following are known:

1. bank number
2. bank VCCO
3. intended I/O standard
4. direction
5. termination
6. external voltage domain

A net name alone is not enough to establish FPGA electrical compatibility.


19. Power-Rail Consequences

This decision introduces two explicit user-I/O power domains.

3.3 V domain:

3V3_IO
  |
  +---- VCCO_0
  +---- VCCO_14
  +---- VCCO_15
  +---- QSPI flash
  +---- local oscillator if 3.3 V device selected
  +---- USB-UART / low-speed debug as appropriate

2.5 V domain:

2V5_IO
  |
  +---- VCCO_16
  +---- VCCO_34
  +---- VCCO_35
  +---- external LVDS support circuitry where appropriate


20. VCCO Power Sequencing

The VCCO rails shall follow the FPGA power-sequencing architecture.

Preferred FPGA logic sequence:

1. VCCINT / VCCBRAM
2. VCCAUX
3. VCCO

Therefore 3V3_IO and 2V5_IO shall not be enabled before the prerequisite FPGA
rails unless AMD sequencing analysis explicitly permits the selected
alternative.

The final sequencer/regulator-enable architecture shall account for both VCCO
rails.


21. XPE Consequence

The current XPE model has zero user-I/O power because VCCO loading was not yet
defined.

After pin planning, XPE shall be updated with:

- number of 3.3 V outputs
- number of 3.3 V inputs
- number of 2.5 V outputs
- number of 2.5 V inputs
- LVDS transmitter count
- LVDS receiver count
- clock rates
- toggle rates
- external loading
- termination assumptions

Only after this update can the 3.3 V and 2.5 V regulators be sized properly.


22. Pin-Planning Rules

The following rules shall be applied during the next stage.

RULE-IO-001
PCIe GTP pins are fixed by the selected PCIe/GTP architecture before general
I/O placement.

RULE-IO-002
Configuration pins are assigned before ordinary user I/O.

RULE-IO-003
Clock-capable pins are reserved before generic GPIO assignment.

RULE-IO-004
Differential pairs shall remain on true FPGA P/N differential-pair pins.

RULE-IO-005
Functions crossing a PCB connector should remain grouped within a bank where
practical.

RULE-IO-006
Signals shall be assigned with connector placement and BGA escape direction in
mind.

RULE-IO-007
Do not consume MRCC/SRCC pins with low-value GPIO while clocks remain
unassigned.

RULE-IO-008
Do not rely on pin swapping until Vivado confirms the intended pins are
functionally interchangeable.

RULE-IO-009
Configuration-capable pins shall be marked explicitly in the pin-plan.

RULE-IO-010
Unused pins shall not be assigned a final electrical treatment until the FPGA
design and board-level requirements are understood.


23. Preliminary Functional Bank Map

Bank 14 — 3.3 V
- QSPI configuration
- configuration signals
- limited low-speed spare I/O

Bank 15 — 3.3 V
- PCIE_PERST_N
- SYS_CLK_100
- USB_UART_TX/RX
- I2C
- low-speed SPI
- board control/status
- general low-speed GPIO

Bank 16 — 2.5 V
- LVDS expansion group A
- clock-capable external timing resources
- associated LVCMOS25 control

Bank 34 — 2.5 V
- LVDS expansion group B
- instrumentation I/O

Bank 35 — 2.5 V
- LVDS expansion group C
- optional EXT_CLK / differential trigger
- future daughtercard expansion


24. Resources Intentionally Reserved

Before generic GPIO allocation, reserve:

- all PCIe GTP lane pins
- required MGTREFCLK pins
- QSPI/configuration pins
- JTAG pins
- PROGRAM_B
- INIT_B
- DONE
- CFGBVS
- mode pins
- PUDC_B
- Bank 15 clock-capable pin for SYS_CLK_100
- at least one differential clock-capable pair for EXT_CLK
- sufficient differential pairs for the instrumentation connector
- PCIE_PERST_N input


25. Open Items

1. Exact package pins for all reserved signals
2. Exact SYS_CLK_100 clock-capable pin
3. Exact EXT_CLK bank and differential pair
4. Exact PCIE_PERST_N package pin
5. Number of external LVDS pairs
6. Number of low-speed GPIO
7. Expansion-connector architecture
8. External trigger electrical standard
9. Exact 3.3 V regulator loading
10. Exact 2.5 V regulator loading
11. Final I/O slew and drive constraints
12. DIFF_TERM strategy
13. Whether any Bank 14 configuration pins are reused after configuration


26. Design Gate

The Altium FPGA symbol shall not be wired arbitrarily.

Before schematic connectivity is finalized, the project shall produce a
pin-planning table containing at least:

- package pin
- FPGA pin/function
- bank
- bank voltage
- P/N differential mate
- clock capability
- configuration function
- board net
- direction
- I/O standard
- interface group
- notes / restrictions

The pin plan shall be validated in Vivado before being treated as frozen.


27. Consequences

Rev A now has a clear user-I/O voltage architecture:

3.3 V:
- configuration
- board control/debug
- PCIe reset
- local fabric clock

2.5 V:
- LVDS instrumentation
- differential expansion
- high-speed external digital I/O

This architecture intentionally favors simplicity and manufacturability over
maximum I/O-standard flexibility.


28. References

Primary AMD references:

- UG475 — 7 Series FPGAs Packaging and Pinout Product Specification
- UG471 — 7 Series FPGAs SelectIO Resources User Guide
- DS181 — Artix-7 FPGAs Data Sheet
- PG054 — 7 Series FPGAs Integrated Block for PCI Express
