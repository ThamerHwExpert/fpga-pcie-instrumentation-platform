DEC-006 — Reset and Startup Architecture

Status: Accepted — baseline architecture
Revision: A
Date: 2026-09-05


1. Objective

Define the reset and startup architecture for Rev A of the Open FPGA PCIe
Instrumentation Platform.

The design must distinguish between:

- FPGA power-on/configuration startup
- PCIe Fundamental Reset (PERST#)
- PCIe user-logic reset
- local FPGA application reset
- clock/MMCM reset and lock handling
- external peripheral reset
- manual/recovery actions

The objective is to avoid one large "global reset" net that mixes unrelated
reset requirements.


2. Baseline Decision

Rev A shall use separate reset domains:

A. FPGA configuration startup
   Controlled primarily by the FPGA power-on/configuration mechanism,
   PROGRAM_B, INIT_B, and DONE.

B. PCIe core reset
   Driven from the host PCIe PERST# sideband signal into the PCIe core
   sys_rst_n input.

C. PCIe-domain user logic reset
   Driven from the PCIe core user_reset_out signal and synchronized by the
   AMD PCIe implementation to user_clk_out.

D. Local FPGA logic reset
   Generated inside the FPGA for SYS_CLK_100 and released synchronously after
   the local clock is valid.

E. Optional external-clock-domain reset
   Generated separately for logic that depends on EXT_CLK.

F. External peripheral reset
   Derived from board power-good/startup conditions where required.

The architecture shall not gate or delay PCIe PERST# casually with generic
application reset logic.


3. Top-Level Reset Architecture

Conceptual architecture:

                       HOST / PCIe SLOT
                             |
                           PERST#
                             |
                             v
                    Electrical Interface
                             |
                             v
                    PCIe core sys_rst_n
                             |
                             v
                       PCIe Hard Core
                             |
                  user_clk_out + user_reset_out
                             |
                             v
                    PCIe-Domain Logic


       FPGA CONFIGURATION                    BOARD POWER
       ------------------                    -----------
       PROGRAM_B                               PGOODs
       INIT_B                                    |
       DONE                                      |
          |                                      v
          v                               Startup / Status
      FPGA Startup                                |
          |                                       |
          +-------------------+-------------------+
                              |
                              v
                        Local Reset Logic
                              |
                    +---------+---------+
                    |                   |
                    v                   v
              SYS_CLK_100 reset    Peripheral resets


4. FPGA Configuration Startup

The FPGA configuration subsystem shall follow the architecture defined in
DEC-004.

Key configuration signals:

- PROGRAM_B
- INIT_B
- DONE
- M[2:0]
- CCLK
- QSPI interface

The FPGA's internal power-on reset/configuration behavior shall be relied upon
for the initial device configuration sequence.

External board logic shall not hold PROGRAM_B Low as a normal startup delay
mechanism.

Reason:

Holding PROGRAM_B Low from power-up is not the recommended mechanism for
stalling the power-on configuration sequence.

If a future design requires intentional configuration delay, INIT_B and the
AMD-defined configuration behavior shall be evaluated explicitly.

Because this board is a PCIe endpoint, any intentional configuration delay
must also be checked against the PCIe cold-boot timing requirement.


5. PCIe PERST#

5.1 Function

PERST# is the host-provided PCIe Fundamental Reset signal.

The AMD 7-Series PCIe core uses:

sys_rst_n

as an asynchronous active-Low system reset.

For a normal PCIe add-in Endpoint, the slot PERST# signal shall drive the
PCIe core system reset path.


5.2 Minimum Assertion Time

AMD specifies that sys_rst_n must be asserted for at least:

1.5 us

during power-on and warm reset operations.

The host PCIe reset timing normally provides a substantially longer assertion
during cold boot.


5.3 Electrical Level

PCIe PERST# is a 3.3 V sideband signal.

Therefore the electrical connection to the FPGA shall not be finalized until
the destination FPGA I/O bank voltage and I/O standard are known.

Two implementation possibilities are allowed:

Option A — Direct connection
- destination FPGA bank configured for compatible 3.3 V I/O
- AMD input limits satisfied
- startup behavior verified

Option B — Level translation / buffering
- used if the selected FPGA bank is not directly 3.3 V compatible
- translator must preserve active-Low reset behavior
- translator must have deterministic behavior during power sequencing
- propagation delay must not compromise PCIe reset timing

This decision shall be closed during FPGA I/O bank planning.


6. PERST# Conditioning

PERST# shall not be passed through an arbitrary RC delay intended to "improve"
startup timing.

Reasons:

- the host controls PCIe Fundamental Reset timing
- RC thresholds vary with process, voltage, temperature, and leakage
- delayed deassertion can consume the PCIe link-training timing budget
- distorted edges can create ambiguous FPGA input timing

Allowed implementation elements include:

- level translation where electrically required
- a small series resistor footprint for signal-integrity/debug purposes if
  justified
- test access that does not materially load the signal

No local logic shall intentionally extend PERST# unless a later system-level
analysis proves it necessary and PCIe timing remains compliant.


7. PCIe Cold Reset

Cold Reset occurs during application of system power.

Expected sequence:

Host/system power applied
        |
        v
FPGA rails ramp
        |
        v
FPGA POR / initialization
        |
        v
QSPI configuration
        |
        v
FPGA DONE
        |
        +----------------------+
        |                      |
        v                      v
PCIe clock available       PERST# still asserted
        |                      |
        +----------+-----------+
                   |
                   v
        Host deasserts PERST#
                   |
                   v
         PCIe core exits reset
                   |
                   v
             Link training
                   |
                   v
              Enumeration


The board shall be designed so that FPGA configuration completes early enough
for the PCIe endpoint to be ready when PERST# is released.

The board shall not assume that the host will wait indefinitely.


8. PCIe Cold-Boot Timing Gate

For PCIe add-in-card startup, the platform can provide approximately a
100 ms-class interval from stable system power until PERST# deassertion.

AMD also notes that the PCIe port must be ready to begin link training within
the required interval after PERST# deassertion.

Therefore:

CONFIG_DONE_BEFORE_PERST_RELEASE

is a required system-level validation objective.

This is not implemented as a direct hardware AND gate between DONE and PERST#.

Instead, it is a timing requirement to be proven by:

- configuration-rate calculation
- QSPI flash selection
- power-rail startup measurement
- DONE measurement
- PERST# measurement
- cold-boot host testing

If Rev A cannot meet the available cold-boot configuration budget with
ordinary Master SPI x4 configuration, alternative AMD-supported PCIe
configuration methods shall be evaluated.


9. Warm Reset

A PCIe Fundamental Warm Reset can occur without removing board power.

During a warm reset:

PERST# asserts Low
      |
      v
sys_rst_n asserts
      |
      v
PCIe core / transceiver logic reset
      |
      v
PERST# deasserts
      |
      v
PCIe core resumes link training

FPGA configuration memory and unrelated local application logic are not
automatically assumed to be erased merely because PERST# asserted.

Application behavior during warm reset shall therefore be designed explicitly.


10. PCIe Hot Reset

PCIe Hot Reset is an in-band protocol event.

It is not implemented by externally asserting sys_rst_n/PERST#.

The AMD PCIe core reports/handles hot-reset behavior through its PCIe
interface logic.

Rev A shall not incorrectly wire a board-level reset circuit around the
assumption that every PCIe reset appears on PERST#.


11. PCIe User Logic Reset

The PCIe core provides:

user_reset_out

associated with:

user_clk_out

AMD defines user_reset_out as the reset indication for PCIe user logic.

It is asserted when conditions such as the following occur:

- Fundamental Reset
- PCIe/core clock PLL lock loss
- transceiver PLL lock loss

It deasserts synchronously relative to user_clk_out after the required
conditions recover.

Therefore:

PCIe-facing FPGA logic shall use the PCIe core's user_reset_out mechanism
rather than a separately invented asynchronous reset.

Concept:

PCIe Core
   |
   +---- user_clk_out ------+
   |                        |
   +---- user_reset_out ----+----> PCIe application logic


12. Local FPGA Logic Reset

The local 100 MHz domain shall have its own reset architecture.

Concept:

SYS_CLK_100
    |
    v
Clock buffer / optional MMCM
    |
    +---- clock valid / LOCKED
    |
    v
Reset synchronizer
    |
    v
LOCAL_RST

Baseline principle:

- reset assertion may be asynchronous where required
- reset deassertion shall be synchronized to SYS_CLK_100

This avoids releasing synchronous logic at arbitrary positions relative to the
destination clock.


13. MMCM / PLL Lock Handling

If an MMCM or PLL is used for a local clock domain:

Clock source
    |
    v
MMCM / PLL
    |
    +---- CLKOUT
    |
    +---- LOCKED
             |
             v
       reset-release logic
             |
             v
       synchronous domain

Logic driven from the generated clock shall remain in reset until the clock
resource is stable.

If the input/feedback clock is lost and the clock manager loses LOCKED, the
affected logic shall be returned to a safe reset state.

The exact reset synchronizer implementation belongs to the FPGA design, not
to the PCB schematic.


14. External Clock Domain

Logic depending on EXT_CLK shall have an independent reset.

EXT_CLK must not be required for:

- JTAG access
- FPGA configuration
- basic board status
- PCIe enumeration
- local debug

If EXT_CLK disappears, only the dependent instrumentation domain should be
affected unless the application architecture explicitly requires a broader
response.


15. Power-Good Signals

Regulator PGOOD signals are used for:

- board status
- sequencing/supervision where required
- external peripheral reset
- fault detection
- bring-up measurement

They shall NOT automatically be combined into PCIe PERST#.

The FPGA configuration subsystem already has defined power-on-reset behavior,
and the host owns PCIe Fundamental Reset timing.

A power supervisor/sequencer may be used to control the regulator enable
sequence and external peripheral resets, but the exact implementation remains
TBD.


16. DONE and Application Reset

DONE indicates successful FPGA configuration.

DONE alone does not prove that:

- all application clocks are stable
- PCIe link training has completed
- external peripherals are ready
- every FPGA clock domain is safely released from reset

Therefore DONE shall primarily be treated as:

- configuration-status information
- bring-up/debug information
- an input to higher-level startup logic only where justified

Application reset release shall be based on the requirements of each clock
domain.


17. Manual Reset Strategy

Rev A shall distinguish two manual actions:

A. FPGA reconfiguration
   PROGRAM_B push-button

B. Application reset
   Optional user/application reset push-button

These shall not be conflated.

PROGRAM_B clears/restarts FPGA configuration.

A future application-reset button, if fitted, should reset application logic
without necessarily forcing FPGA reconfiguration.

The need for the second button remains TBD.


18. External Peripheral Reset

Devices such as:

- USB-UART bridge
- external clock-conditioning IC
- future expansion peripherals

may require reset or enable sequencing.

Their reset release shall depend on:

- their own supply validity
- required clock validity
- device-specific startup timing

Do not place every peripheral on one generic FPGA reset line by default.


19. Reset Naming Convention

Recommended schematic / HDL naming:

PCIe connector:
PCIE_PERST_N

PCIe core input:
PCIE_SYS_RST_N

PCIe user logic:
PCIE_USER_RST

Local FPGA logic:
SYS_RST

External-clock logic:
EXTCLK_RST

Configuration:
FPGA_PROGRAM_N
FPGA_INIT_N
FPGA_DONE

Power status:
CORE_PGOOD
AUX_PGOOD
MGTAVCC_PGOOD
MGTAVTT_PGOOD
VCCO_PGOOD where required

Suffix convention:

_N = active Low

Do not mix _B, #, and _N randomly in internal board net names.
Vendor pin names can retain their official naming in component symbols and
documentation.


20. DFT / Measurement Requirements

Provide practical access to:

- PCIE_PERST_N
- FPGA_PROGRAM_N
- FPGA_INIT_N
- FPGA_DONE
- major PGOOD signals
- local reset/status signal where useful

During bring-up, oscilloscope captures shall include:

Capture A — Power / Configuration
- VCCINT
- VCCAUX
- VCCO_0
- INIT_B
- DONE

Capture B — PCIe Startup
- selected power-good reference
- DONE
- PERST#
- optionally a logic-level PCIe status signal from the FPGA

Capture C — Reset Recovery
- PERST#
- PCIe user reset/status
- link-up indication

The high-speed PCIe serial lanes are not part of this reset timing capture.


21. Validation Cases

At minimum, test:

TEST-RST-001
Cold power-on from completely unpowered state.

Expected:
FPGA configures and host enumerates the endpoint.


TEST-RST-002
Repeated cold boots.

Expected:
No intermittent configuration/enumeration failure.


TEST-RST-003
Host warm reset / PERST# assertion.

Expected:
PCIe core resets and retrains without requiring FPGA reconfiguration.


TEST-RST-004
Manual PROGRAM_B activation.

Expected:
FPGA configuration is cleared and reloaded from QSPI.


TEST-RST-005
Local FPGA application reset, if implemented.

Expected:
Application logic resets without unnecessarily resetting PCIe/configuration.


TEST-RST-006
Missing external instrumentation clock.

Expected:
Basic configuration, JTAG, local logic, and PCIe remain operational.


TEST-RST-007
Clock-manager lock loss, where applicable.

Expected:
Dependent logic enters reset and recovers deterministically.


22. Major Risks

RISK-001 — FPGA configuration completes too late for PCIe cold boot

Mitigation:
Measure power-to-DONE and PERST# timing. Increase validated SPI
configuration speed or use an AMD-supported PCIe configuration method if
required.


RISK-002 — PERST# electrical incompatibility

Mitigation:
Resolve destination FPGA bank voltage before schematic freeze. Add a
translator/buffer only if required.


RISK-003 — asynchronous reset release

Mitigation:
Synchronize deassertion separately in each destination clock domain.


RISK-004 — overcoupled reset architecture

Mitigation:
Keep FPGA configuration reset, PCIe reset, local application reset, and
peripheral reset logically separate.


RISK-005 — use of PGOOD to delay PCIe reset

Mitigation:
Do not locally extend PERST# without system-level PCIe timing analysis.


23. Open Items

1. Exact FPGA pin and bank used for PERST#
2. Whether PERST# requires level translation
3. Exact regulator PGOOD topology
4. Power supervisor/sequencer device
5. Whether local SYS_CLK uses an MMCM
6. Application reset push-button requirement
7. External peripheral reset requirements
8. Final FPGA reset-synchronizer implementation
9. Cold-boot timing result
10. Warm-reset host validation procedure


24. Consequences

This decision prevents a common design mistake:

There will NOT be one monolithic "RESET" net for the complete board.

Instead:

- FPGA configuration has its own startup mechanism
- PCIe Fundamental Reset follows host PERST#
- PCIe-domain logic follows the PCIe core user reset
- local clock domains use synchronized local resets
- external clock domains reset independently
- peripheral resets are generated from their actual dependencies

This architecture shall be reflected in both the Altium schematic hierarchy
and the FPGA HDL reset architecture.


25. References

Primary references:

- AMD PG054 — 7 Series FPGAs Integrated Block for PCI Express
- AMD UG470 — 7 Series FPGAs Configuration User Guide
- AMD UG472 — 7 Series FPGAs Clocking Resources User Guide
- PCI Express Card Electromechanical Specification
