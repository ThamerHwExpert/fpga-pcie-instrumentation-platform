Power Budget — Rev A

Project: Open FPGA PCIe Instrumentation Platform
Revision: A
Status: Working calculation
Date: 2026-09-05


1. Objective

Estimate FPGA rail currents and total board power before selecting regulators.

This is a pre-implementation estimate. It is not a measured result and shall
be updated later using Vivado post-implementation power analysis.


2. Baseline Device

Family: AMD/Xilinx Artix-7
Device: XC7A35T
Package: FGG484
Speed grade: -2 (provisional for first estimate)
Temperature grade: Commercial (provisional)
Core voltage: 1.0 V

The final ordering code is not yet frozen.


3. Estimation Method

Use AMD Xilinx Power Estimator (XPE) for 7 Series and Zynq-7000 devices.

For the first pass, use a conservative pre-implementation scenario rather
than pretending that final FPGA utilization is already known.

The estimate shall include:
- device static power
- FPGA fabric dynamic power
- PCIe integrated block
- four GTP channels at PCIe Gen2 rate
- clocks
- BRAM/DSP assumptions
- configuration/auxiliary loading where represented in XPE

User-I/O power is intentionally provisional until the FPGA I/O architecture
is defined.


4. First-Pass XPE Assumptions

These values are engineering assumptions for sizing exploration only.
They are NOT final design requirements.

Device settings:
- Device: XC7A35T
- Package: FGG484
- Speed grade: -2
- Process: Maximum
- Ambient temperature: 40 degC
- Airflow / heatsink: leave at default initially unless XPE requires entry

Fabric assumptions:
- Main fabric clock: 100 MHz
- LUT utilization: 25 percent
- FF utilization: 25 percent
- BRAM utilization: 20 percent
- DSP utilization: 10 percent
- Default/typical activity factors may be used for the first estimate
  unless a more appropriate value is known

PCIe / GTP:
- Protocol/use case: PCI Express
- Line rate: 5.0 Gb/s
- Active GTP channels: 4
- Direction: TX + RX active
- PCIe integrated endpoint block: enabled

I/O:
- Do not invent the final external-I/O population.
- Keep user-I/O loading minimal/provisional for this first core-power pass.
- VCCO regulator sizing remains open until DEC-007 I/O bank architecture.


5. Why Maximum Process Is Used

Static FPGA current varies strongly with silicon process and temperature.

For regulator sizing, the first conservative estimate should use the
maximum-process model rather than a typical-process value.

A second nominal scenario may later be created for expected operating power.


6. XPE Results to Record

After configuring XPE, copy the Power Supply panel results into the table.

Rail         Voltage       XPE Current       XPE Power       Notes
---------------------------------------------------------------------------
VCCINT       1.00 V        TBD               TBD             Core
VCCBRAM      1.00 V        TBD               TBD             BRAM
VCCAUX       1.80 V        TBD               TBD             Auxiliary
VCCO         TBD           DEFERRED          DEFERRED        Await I/O plan
VMGTAVCC     1.00 V        TBD               TBD             GTP analog
VMGTAVTT     1.20 V        TBD               TBD             GTP termination
VCCADC       1.80 V        TBD               TBD             If XADC enabled

Total FPGA power: TBD W


7. Startup-Current Check

After recording XPE operating currents, compare them with AMD minimum
power-on current requirements.

The regulator must satisfy the greater of:

A. estimated operating current + design margin

or

B. required FPGA power-on/startup current

Do not add startup current blindly on top of the operating estimate if AMD/XPE
already reports the minimum power-on requirement for that rail.


8. Preliminary Regulator Sizing Method

For each rail:

I_REG_MIN = max(I_XPE_operating, I_power_on_min)

Then choose a design margin based on uncertainty and expected future expansion.

For this early Rev A estimate, use 25 percent as a provisional sizing margin:

I_TARGET = 1.25 x I_REG_MIN

This 25 percent value is a design assumption, not an AMD requirement.

The final margin may change after:
- actual FPGA implementation
- transient analysis
- regulator thermal analysis
- PDN analysis
- future expansion requirements


9. Combined-Rail Calculation

VCCINT and VCCBRAM are planned to share a 1.0 V core regulator.

Therefore:

I_CORE_XPE = I_VCCINT + I_VCCBRAM

I_CORE_TARGET = 1.25 x max(I_CORE_XPE, applicable startup requirement)

Do not combine VMGTAVCC with the core rail yet.

That question will be evaluated after the GTP current and noise/filtering
requirements are known.


10. Input-Power Estimate

After rail powers are known:

P_FPGA = sum(V_rail x I_rail)

Then estimate regulator input power using provisional efficiency:

P_INPUT_EST = sum(P_OUT_rail / efficiency_rail)

Do not assume one common efficiency for the final design.

For an early feasibility check only, 85 to 90 percent can be used as an
assumption for low-voltage buck conversion from 12 V.

The actual efficiency shall later come from the selected regulator operating
point and datasheet.


11. PCIe Slot Power Check

After the FPGA and peripheral estimates are complete, compare total board
input power against the applicable PCIe CEM add-in-card power limits.

Do not assume that the connector can provide unlimited current simply because
both +12 V and +3.3 V pins exist.


12. Thermal Check

Record from XPE:

- total on-chip power
- estimated junction temperature
- effective thermal resistance assumptions
- ambient temperature

If the estimated junction temperature is too high, investigate:
- FPGA utilization/activity
- airflow
- copper spreading
- package thermal path
- heatsink requirement
- FPGA/device selection


13. Result Status

Current status:

[ ] XPE downloaded
[ ] Device configured
[ ] FPGA fabric assumptions entered
[ ] PCIe/GTP configuration entered
[ ] Power Supply results recorded
[ ] Startup-current comparison completed
[ ] Regulator target currents calculated
[ ] Total board power estimated
[ ] Thermal result reviewed


14. Next Actions

1. Run XPE with the assumptions above.
2. Capture the Summary and Power Supply panel results.
3. Record each FPGA rail current.
4. Review the values before selecting any regulator.
5. Update this file with verified XPE output.
6. Proceed to regulator topology/component selection only after review.


15. References

- AMD UG440 — Xilinx Power Estimator User Guide
- AMD Xilinx Power Estimator — 7 Series and Zynq-7000 spreadsheet
- AMD DS181 — Artix-7 FPGAs Data Sheet
- AMD UG482 — 7 Series FPGAs GTP Transceivers User Guide
- AMD UG483 — 7 Series FPGAs PCB Design Guide
