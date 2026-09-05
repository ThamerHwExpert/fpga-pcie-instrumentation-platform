DEC-003 — Power Architecture

Status: Accepted — architecture only
Revision: A
Date: 2026-09-05


1. Objective

Define the FPGA power-domain architecture before selecting regulators.

This decision establishes:
- required supply rails
- nominal voltages
- rail consolidation
- sequencing strategy
- measurement requirements
- power-estimation method

Regulator part numbers and final current ratings are intentionally not
selected in this decision.


2. FPGA Supply Domains

For the baseline XC7A35T operating with a nominal 1.0 V core:

Rail        Nominal   Function                     Rev A decision
---------------------------------------------------------------------------
VCCINT      1.0 V     FPGA core                    Required
VCCBRAM     1.0 V     Block RAM                    Required
VCCAUX      1.8 V     FPGA auxiliary circuitry     Required
VCCO_0      TBD       Configuration bank           Depends on configuration architecture
VCCO_x      TBD       User I/O banks               Depends on I/O planning
VMGTAVCC    1.0 V     GTP analog circuitry         Required
VMGTAVTT    1.2 V     GTP termination circuitry    Required
VCCADC      1.8 V     XADC analog supply           Required if XADC used


3. Core-Rail Consolidation

VCCINT and VCCBRAM operate at the same nominal voltage.

Therefore Rev A shall use:

VCCINT + VCCBRAM -> common 1.0 V regulator

unless later power-integrity analysis identifies a reason to separate them.

Architecture:

+12 V
  |
  +----> 1.0 V CORE
             |
             +---- VCCINT
             |
             +---- VCCBRAM


4. GTP Supplies

The PCIe GTP transceivers require:

VMGTAVCC = 1.0 V
VMGTAVTT = 1.2 V

These shall be treated as high-priority low-noise power domains.

Concept:

+12 V
  |
  +----> 1.0 V supply ----> filtering ----> VMGTAVCC
  |
  +----> 1.2 V supply ----> filtering ----> VMGTAVTT

Whether VMGTAVCC shares the main 1.0 V regulator with VCCINT or uses
a separate regulator remains TBD.

The decision shall be based on:
- GTP noise requirements
- regulator noise
- load transients
- filtering requirements
- BOM complexity
- efficiency
- PCB area

The GTP power network shall follow AMD GTP-specific filtering guidance.


5. Auxiliary Supply

VCCAUX shall operate at:

1.8 V nominal

Concept:

+12 V
  |
  +----> 1.8 V AUX
             |
             +---- VCCAUX
             |
             +---- auxiliary circuitry where appropriate

If XADC is used, VCCADC may be derived from the 1.8 V domain through
appropriate filtering rather than automatically sharing the digital
auxiliary node directly.


6. I/O Bank Supplies

VCCO shall NOT yet be globally assigned.

Each FPGA I/O bank voltage shall be selected from the actual interface
requirements.

Examples may include:
- 1.8 V
- 2.5 V
- 3.3 V

The final VCCO architecture shall follow the FPGA pin-planning decision.

No I/O-bank voltage shall be selected merely because a convenient rail
already exists.


7. Preliminary Board Power Tree

                    PCIe EDGE
                   +12 V / +3V3
                         |
                    Input protection
                         |
                +--------+---------+
                |        |         |
                v        v         v
             1.0 V     1.8 V     1.2 V
              CORE      AUX       MGT
                |        |         |
          +-----+        |         |
          |              |         |
       VCCINT          VCCAUX    VMGTAVTT
       VCCBRAM           |
                          +--> VCCADC
                               via filter

                1.0 V MGTAVCC
                     |
                 filtering
                     |
                 VMGTAVCC

Final source-regulator consolidation remains TBD.


8. PCIe Input Power Strategy

The PCIe connector provides +12 V and +3.3 V supplies.

Rev A shall use +12 V as the primary conversion source for the major
FPGA power rails.

The +3.3 V slot rail shall not yet be assigned as a major FPGA source
until the complete PCIe CEM power budget and auxiliary requirements are
reviewed.

This avoids prematurely mixing host-side power constraints with FPGA
rail decisions.


9. Power-On Sequencing

AMD recommends the FPGA logic sequence:

VCCINT / VCCBRAM
        |
        v
     VCCAUX
        |
        v
      VCCO

VCCINT and VCCBRAM may ramp together because they use the same nominal
voltage.

For the GTP supplies, the preferred sequence shall ensure VMGTAVTT does
not precede both VCCINT and VMGTAVCC.

Preferred architecture:

VCCINT + VMGTAVCC
        |
        v
     VMGTAVTT

VCCINT and VMGTAVCC may start together.

The final sequencing implementation shall be validated against AMD
startup requirements.


10. Power-Good Strategy

Each major regulator should provide power-good information where
practical.

Concept:

CORE_PGOOD
AUX_PGOOD
MGT1V0_PGOOD
MGT1V2_PGOOD
      |
      v
Power / Reset Supervisor
      |
      +---- FPGA reset control
      |
      +---- debug/status

Reset release shall require valid power rather than rely only on RC
delays.

The exact supervisor/sequencer device remains TBD.


11. Rail Measurement

Bring-up access shall be provided for at least:

Signal            Access
---------------------------------------
+12V_SLOT         Test point
+3V3_SLOT         Test point
VCCINT/VCCBRAM    Test point
VCCAUX            Test point
VMGTAVCC          Test point
VMGTAVTT          Test point
VCCO domains      Test point where practical
PGOOD signals     Test point

Measurement points shall be positioned so that oscilloscope probing of
startup sequencing is practical.


12. Power-Rail Tolerances

For the baseline Artix-7 operating point:

Rail        Nominal   AMD operating range
------------------------------------------
VCCINT      1.00 V    0.95–1.05 V
VCCBRAM     1.00 V    0.95–1.05 V
VCCAUX      1.80 V    1.71–1.89 V
VMGTAVCC    1.00 V    0.97–1.03 V
VMGTAVTT    1.20 V    1.17–1.23 V
VCCADC      1.80 V    1.71–1.89 V

VCCO range depends on the selected I/O standard.

Regulator accuracy, DC drop, ripple, transient deviation, and PCB
distribution loss shall all fit inside these device-level limits.


13. Regulator Sizing

Regulators shall not be sized from quiescent FPGA current.

Final current requirements shall be generated using AMD Xilinx Power
Estimator (XPE).

The estimate shall include:
- exact FPGA
- speed grade
- temperature grade
- PCIe GTP usage
- logic utilization
- clock frequencies
- BRAM
- DSP usage
- I/O count
- I/O voltage
- output toggle rates
- ambient temperature
- process assumptions

Regulator sizing shall then include appropriate engineering margin.


14. Power-On Current

AMD specifies additional minimum startup current beyond quiescent
current for proper power-on/configuration.

For XC7A35T:

ICCINT_MIN = ICCINT_Q + 120 mA
ICCAUX_MIN = ICCAUX_Q + 40 mA
ICCO_MIN = ICCO_Q + 40 mA per bank
ICCBRAM_MIN = ICCBRAM_Q + 60 mA

These values are startup requirements and shall NOT be treated as the
final operating-current budget.


15. Ramp-Time Requirement

The FPGA supply rails shall meet AMD ramp-time requirements.

The relevant FPGA and GTP supply ramps shall remain within the vendor
specified 0.2 ms to 50 ms range.

This shall be verified during board bring-up with an oscilloscope.


16. PDN Design

Regulator selection alone does not constitute the FPGA power design.

The final PDN shall include:
- regulator control-loop design
- bulk capacitance
- high-frequency decoupling
- FPGA mounting inductance
- via inductance
- plane impedance
- GTP filtering
- rail-to-ground loop inductance
- transient-current requirements

Decoupling values and quantities shall follow AMD guidance and be
validated against the actual PCB implementation.


17. Open Items

The following remain TBD:

1. FPGA speed grade
2. XPE power estimate
3. regulator current requirements
4. regulator topology
5. regulator part numbers
6. VMGTAVCC regulator sharing vs dedicated supply
7. VCCO bank voltages
8. configuration-bank voltage
9. power supervisor/sequencer
10. PCIe input protection
11. final decoupling network
12. PDN impedance targets
13. thermal requirements


18. References

- AMD DS181 — Artix 7 FPGAs Data Sheet
- AMD UG440 — Xilinx Power Estimator User Guide
- AMD UG482 — 7 Series FPGAs GTP Transceivers User Guide
- AMD UG483 — 7 Series FPGAs PCB Design Guide
