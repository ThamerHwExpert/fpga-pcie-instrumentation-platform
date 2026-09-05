# System Requirements — Rev A

## 1. Purpose

This document defines the baseline requirements for Rev A of the
Open FPGA PCIe Instrumentation Platform.

The board is intended to be a public hardware engineering project demonstrating:

- FPGA hardware architecture
- PCI Express endpoint design
- FPGA power architecture
- high-speed PCB design
- clocking and reset design
- configuration and debug
- digital I/O expansion
- DFM / DFT
- bring-up and validation

This document defines WHAT the system shall achieve.

Implementation choices such as regulator part numbers, PCB stack-up,
connector models, and routing topology are documented separately as
engineering decisions.

---

## 2. Requirement States

Requirements use the following states:

- **LOCKED** — accepted baseline requirement
- **PROVISIONAL** — intended direction but still requires engineering validation
- **TBD** — not yet decided
- **DEFERRED** — intentionally excluded from Rev A

---

# 3. System Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| SYS-001 | The board shall operate as a PCI Express add-in endpoint card. | LOCKED | Host enumeration test |
| SYS-002 | The primary programmable device shall be from the AMD/Xilinx Artix-7 family. | LOCKED | Schematic/BOM review |
| SYS-003 | The selected FPGA package shall provide at least four GTP transceiver channels suitable for PCIe x4 operation. | LOCKED | Datasheet review |
| SYS-004 | The design shall be implemented as a fully documented public engineering project. | LOCKED | Repository review |
| SYS-005 | Rev A shall prioritize robust PCIe/FPGA operation over peripheral feature count. | LOCKED | Architecture review |

---

# 4. PCI Express Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| PCI-001 | The board shall support PCI Express Gen2 operation. | LOCKED | PCIe link test |
| PCI-002 | The board shall support up to four PCIe lanes. | LOCKED | Schematic + enumeration |
| PCI-003 | The FPGA shall operate as a PCIe endpoint. | LOCKED | FPGA/host validation |
| PCI-004 | The design shall use the PCIe slot reference clock unless later analysis justifies another architecture. | LOCKED | Schematic review |
| PCI-005 | PERST# shall be implemented according to PCIe and FPGA requirements. | LOCKED | Schematic + scope measurement |
| PCI-006 | PCIe transmit and receive differential pairs shall be routed as controlled-impedance high-speed interconnects. | LOCKED | PCB review |
| PCI-007 | PCIe routing shall maintain continuous reference return paths. | LOCKED | PCB review |
| PCI-008 | Lane polarity inversion and lane-order constraints shall be reviewed before routing. | PROVISIONAL | FPGA/PCIe documentation review |

---

# 5. FPGA Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| FPGA-001 | Initial FPGA target shall be XC7A35T. | PROVISIONAL | Device-selection study |
| FPGA-002 | Initial package target shall be FGG484. | PROVISIONAL | Package-selection study |
| FPGA-003 | Final speed grade shall be selected based on PCIe requirements, availability, cost, and timing needs. | TBD | BOM/device review |
| FPGA-004 | User I/O bank voltages shall be assigned based on actual interface requirements. | TBD | I/O planning review |
| FPGA-005 | FPGA pin assignment shall be completed before schematic finalization. | LOCKED | Pin-plan review |
| FPGA-006 | GTP placement and bank usage shall be validated against the selected package. | LOCKED | Package documentation review |

---

# 6. Configuration and Debug Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| CFG-001 | The FPGA shall support JTAG programming and debug. | LOCKED | JTAG programming test |
| CFG-002 | The board shall support autonomous FPGA configuration from QSPI flash. | LOCKED | Power-cycle boot test |
| CFG-003 | Configuration status signals shall be accessible for debug where practical. | LOCKED | Schematic + test |
| CFG-004 | JTAG shall remain accessible during initial board bring-up. | LOCKED | Physical inspection |
| CFG-005 | USB-to-UART debug capability shall be provided unless architecture review identifies a better alternative. | PROVISIONAL | Functional test |

---

# 7. Clocking Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| CLK-001 | PCIe reference clock shall be received from the host connector. | LOCKED | Scope measurement |
| CLK-002 | The FPGA shall have an independent local system clock. | LOCKED | Scope/FPGA test |
| CLK-003 | The board shall provide an external timing or clock input capability. | PROVISIONAL | Functional test |
| CLK-004 | Clock sources shall meet FPGA jitter, amplitude, and electrical-interface requirements. | LOCKED | Datasheet/calculation review |
| CLK-005 | Clock-capable FPGA pins shall be used where required. | LOCKED | Pin-plan review |

---

# 8. Power Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| PWR-001 | The board shall obtain its primary power from the PCIe connector. | LOCKED | Power test |
| PWR-002 | All FPGA core, auxiliary, I/O, and transceiver rails shall meet vendor voltage requirements. | LOCKED | Rail measurement |
| PWR-003 | FPGA and transceiver supply sequencing shall comply with vendor requirements. | LOCKED | Oscilloscope measurement |
| PWR-004 | Power-rail current capability shall be based on an FPGA power estimate plus engineering margin. | LOCKED | Power-budget review |
| PWR-005 | Critical rails shall include accessible measurement points. | LOCKED | PCB inspection |
| PWR-006 | Power-good/reset interaction shall be explicitly designed rather than relying on uncontrolled startup timing. | LOCKED | Schematic + startup test |
| PWR-007 | Final regulator topology and devices shall be selected after completion of the power budget. | TBD | Design review |

---

# 9. External I/O Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| IO-001 | The board shall expose general-purpose digital FPGA I/O. | LOCKED | Functional test |
| IO-002 | The board shall provide differential FPGA I/O for higher-speed external interfaces. | PROVISIONAL | Functional test |
| IO-003 | At least one external trigger input shall be provided. | PROVISIONAL | Functional test |
| IO-004 | FPGA I/O connectors shall include appropriate ground references. | LOCKED | PCB review |
| IO-005 | External I/O protection shall be evaluated based on connector accessibility and expected use. | LOCKED | Schematic review |
| IO-006 | I/O voltage domains shall be defined before connector pinout is finalized. | LOCKED | Architecture review |

---

# 10. PCB Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| PCB-001 | PCB layer count shall be selected from routing, return-path, PDN, BGA escape, manufacturability, and cost requirements. | TBD | Stack-up review |
| PCB-002 | Standard mechanically drilled vias shall be preferred where feasible. | PROVISIONAL | BGA escape study |
| PCB-003 | HDI/microvia technology shall only be used if technically justified. | LOCKED | Design review |
| PCB-004 | Controlled impedance shall be based on the selected manufacturer's actual stack-up. | LOCKED | Fabricator data review |
| PCB-005 | High-speed signals shall have continuous reference planes. | LOCKED | PCB review |
| PCB-006 | Plane splits shall not interrupt critical high-speed return-current paths. | LOCKED | PCB review |
| PCB-007 | FPGA decoupling placement shall minimize connection inductance. | LOCKED | Layout review |
| PCB-008 | PCB design shall include practical fabrication and assembly constraints from the selected manufacturer. | LOCKED | DFM review |

---

# 11. DFT / Bring-Up Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| DFT-001 | Critical power rails shall have accessible test points. | LOCKED | Inspection |
| DFT-002 | Reset and configuration signals shall be measurable during bring-up. | LOCKED | Inspection |
| DFT-003 | JTAG shall be available without requiring FPGA configuration. | LOCKED | Bring-up test |
| DFT-004 | Rev A shall support staged power validation before full functional testing. | LOCKED | Bring-up procedure |
| DFT-005 | Board status indicators shall be provided where they materially improve debugging. | PROVISIONAL | Functional test |

---

# 12. Documentation Requirements

| ID | Requirement | State | Verification |
|---|---|---|---|
| DOC-001 | Major architectural decisions shall have documented engineering rationale. | LOCKED | Repository review |
| DOC-002 | Calculations and assumptions shall be documented where they affect component sizing or PCB constraints. | LOCKED | Repository review |
| DOC-003 | Public design material shall contain no proprietary or confidential third-party information. | LOCKED | Publication review |
| DOC-004 | Reference designs shall be clearly identified as references and not represented as original work. | LOCKED | Repository review |
| DOC-005 | Changes to major requirements shall be recorded in Git history. | LOCKED | Git review |

---

# 13. Explicitly Deferred from Rev A

The following features are intentionally excluded from Rev A unless a later
engineering review provides a strong justification:

| Feature | Status |
|---|---|
| DDR3 / DDR3L | DEFERRED |
| PCIe Gen3 | DEFERRED |
| USB 3.0 data interface | DEFERRED |
| Ethernet | DEFERRED |
| HDMI / DisplayPort | DEFERRED |
| FMC connector | DEFERRED |
| HDI / microvias | DEFERRED unless required |
| On-board application MCU | DEFERRED |

The purpose of Rev A is to validate the core FPGA, PCIe, power, clocking,
configuration, external I/O, and PCB architecture before adding substantial
peripheral complexity.

---

# 14. Open Engineering Decisions

The following items remain to be decided:

1. Final FPGA device and package
2. FPGA speed and temperature grade
3. FPGA I/O bank allocation
4. PCIe edge-connector form factor
5. Board dimensions
6. Power budget
7. Regulator topology and part selection
8. Configuration flash capacity
9. Local oscillator frequency
10. External clock interface
11. External trigger electrical standard
12. Expansion connector architecture
13. PCB manufacturer
14. PCB layer count and stack-up
15. Controlled-impedance geometry
16. BGA breakout strategy
17. ESD/protection strategy
18. Debug interface implementation

Each significant decision shall be documented under:

`docs/decisions/`
