DEC-004 — FPGA Configuration Architecture

Status: Accepted — baseline architecture
Revision: A
Date: 2026-09-05


1. Objective

Define the FPGA configuration and development-programming architecture for
Rev A of the Open FPGA PCIe Instrumentation Platform.

The architecture must support:

- autonomous FPGA configuration after power-up
- reliable development and recovery through JTAG
- PCIe cold-boot timing analysis
- configuration-status visibility during bring-up
- future recovery / MultiBoot capability without requiring a redesign
- straightforward PCB implementation


2. Baseline Decision

Rev A shall use:

- AMD/Xilinx Artix-7 XC7A35T-FGG484
- Master SPI configuration mode
- Quad SPI read mode (x4)
- 3.3 V configuration I/O domain
- minimum 64 Mbit external SPI NOR flash
- dedicated external JTAG header
- external access to PROGRAM_B, INIT_B, and DONE
- fixed Master SPI mode straps
- PUDC_B tied High to disable SelectIO internal pull-ups during configuration

Exact flash manufacturer and part number remain TBD.


3. Configuration Mode

The FPGA mode shall be Master SPI.

For 7-Series devices:

M[2:0] = 001

Therefore:

M2 = 0
M1 = 0
M0 = 1

Each mode pin shall be tied to its required logic level directly or through
a resistor no greater than 1 kohm, in accordance with AMD configuration
guidance.

Preferred implementation:

M2 -> 1 kohm -> GND
M1 -> 1 kohm -> GND
M0 -> 1 kohm -> VCCO_0

The mode pins shall not be allowed to float.


4. Quad-SPI Configuration Interface

The FPGA shall boot from a standard SPI NOR flash that supports the AMD
7-Series Master SPI x4 configuration sequence.

Required FPGA configuration signals include:

- CCLK
- FCS_B
- D00_MOSI
- D01_DIN
- D02
- D03

Concept:

                     +------------------+
                     |    SPI NOR       |
                     |                  |
FPGA CCLK ---------->| CLK              |
FPGA FCS_B --------->| CS#              |
FPGA D00 <---------->| IO0              |
FPGA D01 <---------->| IO1              |
FPGA D02 <---------->| IO2              |
FPGA D03 <---------->| IO3              |
                     +------------------+

Signal directions vary during the configuration sequence. The schematic shall
use the official 7-Series pin functions rather than treat the interface as a
generic application SPI bus.

CCLK shall be treated as a critical clock signal.

Series damping footprints on the flash-to-FPGA datapath may be included where
appropriate, but resistor values shall be selected from signal-integrity
analysis rather than copied blindly from a reference design.


5. Configuration Voltage

Rev A shall use a 3.3 V configuration interface.

Therefore:

VCCO_0  = 3.3 V
VCCO_14 = 3.3 V during and after configuration
CFGBVS  = tied to VCCO_0

Reason:

- broad availability of 3.3 V SPI NOR flash
- simple JTAG reference-voltage compatibility
- useful 3.3 V bank for low-speed control/debug I/O
- no level translator required between FPGA configuration pins and flash

Consequence:

Bank 14 becomes a 3.3 V user-I/O bank after configuration.

High-speed or lower-voltage external interfaces shall therefore use other FPGA
I/O banks as required.

This consequence shall be considered explicitly during I/O-bank planning.


6. CFGBVS

CFGBVS shall be tied High to VCCO_0.

For 7-Series Artix devices, CFGBVS must be tied High when VCCO_0 is 2.5 V or
3.3 V.

Incorrect CFGBVS connection can violate configuration-bank voltage limits and
must therefore be treated as a critical schematic-review item.


7. Configuration Flash Capacity

XC7A35T has an uncompressed configuration bitstream length of:

17,536,096 bits

AMD specifies a minimum configuration flash size of:

32 Mbit

for a single XC7A35T configuration image.

Rev A shall instead use at least:

64 Mbit

Reason:

Two complete uncompressed images require approximately:

2 x 17,536,096 bits
= 35,072,192 bits

which exceeds 32 Mbit but fits comfortably inside 64 Mbit.

This provides room for future implementation of:

- golden / recovery image
- application image
- MultiBoot / fallback experimentation

The exact memory map and fallback policy remain TBD.

Bitstream compression shall not be relied upon when sizing the flash.


8. JTAG Architecture

JTAG shall remain available regardless of the normal Master SPI boot mode.

Required signals:

- TCK
- TMS
- TDI
- TDO
- VREF
- GND

Baseline physical interface:

- 14-pin
- 2 x 7
- 2.00 mm pitch
- shrouded / keyed header compatible with AMD/Xilinx programming cables

VREF shall be connected to VCCO_0 = 3.3 V.

The FPGA is the only JTAG device in the baseline Rev A chain.

TCK shall be treated as a clock-quality signal. Optional source-series
termination may be provided if later routing/simulation indicates a need.


9. PROGRAM_B

PROGRAM_B is the active-Low reset input for FPGA configuration logic.

Baseline implementation:

PROGRAM_B
    |
    +---- 4.7 kohm pull-up -> VCCO_0
    |
    +---- momentary push-button -> GND
    |
    +---- test point

The external pull-up shall be no weaker than the AMD-recommended maximum
resistance.

The push-button allows manual configuration restart during bring-up.

PROGRAM_B shall not be used as the primary method to delay initial power-up
configuration.


10. INIT_B

INIT_B is an active-Low bidirectional open-drain configuration-status signal.

Baseline implementation:

INIT_B
    |
    +---- 4.7 kohm pull-up -> VCCO_0
    |
    +---- test point
    |
    +---- optional buffered status LED

INIT_B behavior is useful for diagnosing:

- FPGA initialization
- configuration start
- configuration errors

The LED, if populated, shall not load the configuration pin directly.
Use a high-impedance buffer or equivalent low-loading implementation.


11. DONE

DONE indicates successful completion of FPGA configuration.

The pin has an internal pull-up, but Rev A shall provide:

DONE
   |
   +---- external 4.7 kohm pull-up -> VCCO_0
   |
   +---- test point
   |
   +---- optional buffered status LED

The external pull-up provides deterministic board-level behavior and improves
bring-up visibility.

The LED shall not materially load DONE.


12. PUDC_B

PUDC_B controls the internal pull-ups on SelectIO pins during FPGA
configuration.

Rev A decision:

PUDC_B -> 1 kohm -> VCCO_14

This holds PUDC_B High.

Result:

Internal SelectIO pull-ups are disabled during configuration and the affected
user I/O remain high impedance unless externally biased.

Reason:

The board will expose external instrumentation and expansion I/O. Keeping
those pins high impedance during configuration reduces the risk of unintended
external drive states.

Any external signal that requires a guaranteed state during power-up shall
receive an explicit external pull-up or pull-down rather than relying on
PUDC_B behavior.


13. VCCBATT

Rev A does not require storage of an AES decryption key in battery-backed
volatile memory.

Therefore:

VCCBATT shall be connected to GND

unless a later security requirement explicitly introduces battery-backed key
storage.

Bitstream-security architecture is outside the Rev A baseline.


14. EMCCLK

The external master configuration clock input is not required in the baseline
design.

Rev A shall use the FPGA internal configuration oscillator to generate CCLK.

Therefore:

EMCCLK is not part of the baseline configuration path.

The exact unused-pin treatment shall follow AMD pin guidance and final bank
planning.


15. Flash Device Selection Requirements

The final SPI NOR flash shall satisfy all of the following:

- capacity >= 64 Mbit
- 3.3 V-compatible I/O
- support for the read commands required by Artix-7 Master SPI x4
- supported clock frequency adequate for the final configuration-time target
- compatible power-up behavior
- compatible IO2 / IO3 default behavior
- current production availability
- documented endurance and data retention
- package suitable for manual/prototype assembly and normal PCB fabrication

The final flash part shall be selected in a separate component-selection
decision.


16. Vivado Bitstream Configuration

Baseline Vivado configuration intent:

CONFIG_MODE = SPIx4
CONFIG_VOLTAGE = 3.3
CFGBVS = VCCO
BITSTREAM.CONFIG.SPI_BUSWIDTH = 4
Startup clock = CCLK

Final values for the following remain TBD until flash timing is analyzed:

- CONFIGRATE
- SPI_FALL_EDGE
- compression
- MultiBoot / fallback
- watchdog settings

These settings shall be stored in version-controlled XDC/Tcl configuration
files rather than being undocumented GUI-only settings.


17. PCIe Cold-Boot Timing Risk

FPGA configuration timing is a system-level PCIe requirement, not merely a
convenience issue.

The PCIe Endpoint must be configured early enough during cold boot to enter
link training and become visible to the host.

AMD identifies a 100 ms-class PCIe boot-time constraint and notes that the
available FPGA configuration time depends on the host power/reset behavior.

For XC7A35T:

Uncompressed bitstream = 17,536,096 bits

Ideal raw transfer time for x4 SPI is:

T_TRANSFER = bitstream_size / (CCLK x 4)

Examples, ignoring initialization and protocol overhead:

At 33 MHz:
T_TRANSFER = 17,536,096 / (33e6 x 4)
           = approximately 133 ms

At 50 MHz:
T_TRANSFER = approximately 87.7 ms

At 66 MHz:
T_TRANSFER = approximately 66.4 ms

These are ideal transfer-only estimates.

Actual cold-boot time also includes:

- FPGA power-on reset
- initialization
- flash command/address phases
- actual CCLK tolerance
- selected flash timing
- regulator startup
- host PERST# timing

Therefore the final PCIe compliance timing shall NOT be assumed from the raw
transfer equation alone.

A dedicated configuration-time calculation and bench validation are required.

If normal full-image Master SPI x4 cannot meet the required cold-boot timing
with adequate margin, the design shall evaluate:

- higher validated configuration rate
- negative-edge SPI configuration
- alternate compatible SPI flash
- Tandem PROM configuration
- other AMD-supported PCIe configuration methods

This is a design gate before Rev A is considered PCIe-production-ready.


18. Bring-Up Sequence

Configuration bring-up shall proceed in this order:

1. Verify VCCINT, VCCAUX, VCCO_0 and VCCO_14.
2. Verify CFGBVS connection.
3. Verify mode straps M[2:0] = 001.
4. Check INIT_B behavior at power-up.
5. Detect FPGA through JTAG and read device ID.
6. Load a minimal FPGA image through JTAG.
7. Verify PROGRAM_B restart behavior.
8. Verify DONE assertion after JTAG configuration.
9. Program the external SPI flash indirectly through JTAG.
10. Power-cycle the board.
11. Verify autonomous Master SPI x4 boot.
12. Measure CCLK and configuration duration.
13. Verify PCIe link training after cold boot.
14. Verify warm-reset / re-enumeration behavior.


19. DFT Requirements

Accessible measurement points shall be provided for:

- PROGRAM_B
- INIT_B
- DONE
- CCLK
- FCS_B
- VCCO_0
- VCCO_14
- flash 3.3 V supply

Direct test points on all four high-activity SPI data lines are optional and
shall be balanced against routing stubs and signal-integrity impact.


20. Open Items

The following remain TBD:

1. Exact 64 Mbit or larger SPI NOR part number
2. Flash package
3. Final CONFIGRATE
4. SPI_FALL_EDGE setting
5. Need for series damping resistors
6. Exact JTAG connector part number
7. Optional JTAG termination
8. Configuration-status LED circuit
9. MultiBoot / golden-image implementation
10. PCIe cold-boot timing margin
11. Tandem PROM requirement
12. Configuration flash current contribution to the board power budget


21. Consequences

This decision adds a required 3.3 V board rail.

It also fixes:

- VCCO_0 = 3.3 V
- VCCO_14 = 3.3 V
- CFGBVS = High
- Master SPI mode
- SPI x4 configuration
- minimum 64 Mbit configuration flash
- dedicated JTAG accessibility
- Bank 14 as a 3.3 V user-I/O bank

These decisions shall be carried into the FPGA pin-plan, power tree, schematic,
and XPE I/O model.


22. References

Primary AMD references:

- UG470 — 7 Series FPGAs Configuration User Guide
- XAPP586 — Using SPI Flash with 7 Series FPGAs
- PG054 — 7 Series FPGAs Integrated Block for PCI Express
- UG908 — Vivado Design Suite User Guide: Programming and Debugging
- UG912 — Vivado Design Suite Properties Reference Guide
- DS593 — Platform Cable USB II Data Sheet
