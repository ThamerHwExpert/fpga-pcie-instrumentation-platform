Power Budget — Rev A

Project: Open FPGA PCIe Instrumentation Platform
Revision: A
Status: First-pass XPE estimate completed
Date: 2026-09-05


1. Objective

Estimate FPGA rail currents and total FPGA power before selecting regulators.

This is a pre-implementation estimate generated with AMD Xilinx Power
Estimator (XPE). It is not a measured result and shall later be updated with
Vivado post-implementation power analysis.


2. Baseline Device and Environment

Family: Artix-7
Device: XC7A35T
Package: FGG484
Speed grade: -2 (provisional)
Temperature grade: Commercial (provisional)
Process: Maximum
Characterization: Production

Ambient temperature: 40 degC
Airflow: 0 LFM
Heat sink: None
Board size model: Medium
Board layer model: 8 to 11


3. First-Pass FPGA Utilization Assumptions

These are engineering assumptions for early power sizing only.

Fabric:
- Main fabric clock: 100 MHz
- LUTs as logic: 5,200
- Registers: 10,400
- Logic toggle rate: 12.5 percent
- Logic average fanout: 3
- BRAM36 blocks: 10
- DSP48E1 slices: 9

PCIe / GTP:
- Protocol: PCIe Gen2
- Channels: 4
- RX line rate: 5.0 Gb/s
- TX line rate: 5.0 Gb/s
- Data path: 16
- Encoding: 8b/10b
- Operation mode: Transceiver
- Hard PCIe block: enabled

User I/O:
- Not yet modeled
- VCCO power remains TBD until FPGA I/O bank planning is completed


4. XPE Summary Results

Total on-chip power: 1.164 W
Estimated junction temperature: 48.6 degC
Thermal margin: 36.4 degC
Effective theta-JA: 7.4 degC/W

Power breakdown:

Resource          Power
------------------------
Clock             0.088 W
Logic             0.043 W
BRAM              0.020 W
DSP               0.003 W
PLL               0.000 W
MMCM              0.108 W
Other             0.000 W
PCIe              0.058 W
GTP               0.673 W
Device static     0.172 W

The GTP block is the dominant contributor in this first-pass estimate.


5. XPE Power Supply Results

Rail         Voltage    XPE Current    Approx. Rail Power
---------------------------------------------------------
VCCINT       1.00 V     0.378 A        0.378 W
VCCBRAM      1.00 V     0.060 A        0.060 W
VCCAUX       1.80 V     0.085 A        0.153 W
VMGTAVCC     1.00 V     0.313 A        0.313 W
VMGTAVTT     1.20 V     0.246 A        0.295 W
VCCADC       1.80 V     0.030 A        0.054 W
VCCO         TBD        TBD            TBD

Sum of the listed supply-sizing powers is approximately 1.253 W.

This is intentionally not expected to equal Total On-Chip Power exactly.
XPE can display minimum power-on supply current instead of the lower estimated
operating current when Maximum Process is selected. The Power Supply panel can
also include power associated with off-chip loading.


6. GTP / PCIe Result

GTP channels: 4
Protocol: PCIe Gen2
RX data rate: 5.0 Gb/s
TX data rate: 5.0 Gb/s

GTP-related power:
- VCCINT contribution: 0.143 W
- VMGTAVCC contribution: 0.297 W
- VMGTAVTT contribution: 0.291 W

Total GTP block power reported by XPE: 0.673 W

Hard PCIe block contribution: 0.058 W


7. Preliminary Regulator Current Targets

A provisional 25 percent design margin is used for early sizing exploration.
This is an engineering assumption, not an AMD requirement.

Combined FPGA core rail:

VCCINT + VCCBRAM current
= 0.378 A + 0.060 A
= 0.438 A

Provisional target:
0.438 A x 1.25 = 0.548 A

Therefore:
CORE 1.0 V preliminary target >= 0.55 A


VCCAUX:

0.085 A x 1.25 = 0.106 A

Therefore:
AUX 1.8 V preliminary target >= 0.11 A


VMGTAVCC:

0.313 A x 1.25 = 0.391 A

Therefore:
MGTAVCC 1.0 V preliminary target >= 0.40 A


VMGTAVTT:

0.246 A x 1.25 = 0.308 A

Therefore:
MGTAVTT 1.2 V preliminary target >= 0.31 A


VCCADC:

0.030 A x 1.25 = 0.038 A

VCCADC will normally be derived from an appropriate 1.8 V source through the
recommended filtering architecture rather than receiving a dedicated large
regulator solely for this current.


8. Important Limitation

These numbers are NOT the final regulator ratings.

The estimate still excludes or does not fully define:

- user VCCO bank loads
- configuration flash
- USB-UART/debug circuitry
- local oscillators
- external clock/trigger interface
- LEDs/status circuitry
- power supervisors
- regulator losses
- expansion connector loads
- future FPGA implementation changes

The final regulator selection shall therefore include the completed board-level
power budget, not only the FPGA XPE result.


9. Thermal Interpretation

Under the current XPE assumptions:

- Ambient = 40 degC
- No forced airflow
- No heatsink
- Estimated junction temperature = 48.6 degC

This indicates no immediate FPGA thermal concern for the current first-pass
load.

This result is only an estimate. Final thermal validation requires the actual
PCB, copper distribution, FPGA activity, regulator losses, enclosure/airflow,
and measured operating conditions.


10. Current Status

[x] XPE device configured
[x] Maximum-process model selected
[x] Fabric clock entered
[x] Logic estimate entered
[x] BRAM estimate entered
[x] DSP estimate entered
[x] PCIe Gen2 x4 / GTP estimate entered
[x] First-pass rail currents recorded
[x] Thermal estimate reviewed
[ ] User I/O / VCCO model completed
[ ] Board peripheral power added
[ ] Final regulator current targets frozen
[ ] Regulator topology selected
[ ] Regulator components selected
[ ] Post-implementation Vivado power analysis completed
[ ] Hardware measurements completed


11. Next Engineering Actions

1. Define FPGA configuration architecture.
2. Define configuration-bank voltage.
3. Define FPGA I/O bank allocation and VCCO rails.
4. Add configuration/debug/peripheral loads to the board-level power budget.
5. Re-run XPE with the I/O model.
6. Freeze regulator current requirements.
7. Select regulator topology and components.
8. Perform PDN and sequencing review.


12. References

- AMD UG440 — Xilinx Power Estimator User Guide
- AMD DS181 — Artix-7 FPGAs Data Sheet
- AMD UG482 — 7 Series FPGAs GTP Transceivers User Guide
- AMD UG483 — 7 Series FPGAs PCB Design Guide
