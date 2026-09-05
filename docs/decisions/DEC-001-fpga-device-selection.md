# DEC-001 — FPGA Device and Package Selection

**Status:** Accepted  
**Revision:** A  
**Decision:** XC7A35T-FGG484  
**Date:** 2026-09-05

---

## 1. Decision Context

The FPGA is the central component of the Open FPGA PCIe
Instrumentation Platform.

The selected device must support:

- PCI Express Gen2 x4 endpoint operation
- Four bonded high-speed transceiver channels
- Sufficient programmable logic for PCIe control and instrumentation
- Internal block RAM for buffering and control functions
- General-purpose FPGA I/O
- Differential I/O
- JTAG configuration/debug
- QSPI boot
- Practical PCB breakout without requiring HDI unless justified
- Reasonable PCB manufacturing complexity

The objective is not to select the largest FPGA available.

The preferred device should provide sufficient capability while
minimizing unnecessary cost, power, PCB complexity, and design risk.

---

# 2. Candidates

Three primary device/package combinations were considered:

1. XC7A35T-CSG325
2. XC7A35T-FGG484
3. XC7A50T-FGG484

---

# 3. FPGA Resource Comparison

| Parameter | XC7A35T | XC7A50T |
|---|---:|---:|
| Logic cells | 33,280 | 52,160 |
| DSP48E1 slices | 90 | 120 |
| Block RAM | 1,800 Kb | 2,700 Kb |
| GTP transceivers | 4 | 4 |
| Integrated PCIe blocks | 1 | 1 |
| Maximum PCIe configuration | Gen2 x4 | Gen2 x4 |

The XC7A50T provides additional programmable logic, DSP resources,
and block RAM.

It does not provide additional GTP channels or higher PCIe capability
for this application.

---

# 4. Package Comparison

## CSG325

- 325-ball package
- 15 mm × 15 mm body
- 0.8 mm ball pitch
- Four GTP channels bonded out
- Supports PCIe Gen2 x4
- Smaller PCB footprint
- Higher BGA breakout density

## FGG484

- 484-ball package
- 23 mm × 23 mm body
- 1.0 mm ball pitch
- Four GTP channels bonded out
- Supports PCIe Gen2 x4
- Larger PCB footprint
- More favorable pitch for conventional PCB breakout
- Better suited to a design objective that prefers standard
  mechanically drilled vias over HDI

---

# 5. Candidate Evaluation

| Criterion | XC7A35T-CSG325 | XC7A35T-FGG484 | XC7A50T-FGG484 |
|---|---|---|---|
| PCIe Gen2 x4 | PASS | PASS | PASS |
| Four GTP lanes | PASS | PASS | PASS |
| Logic resources | PASS | PASS | PASS+ |
| BRAM resources | PASS | PASS | PASS+ |
| FPGA I/O flexibility | Moderate | High | High |
| Package pitch | 0.8 mm | 1.0 mm | 1.0 mm |
| Conventional BGA breakout | More difficult | Preferred | Preferred |
| PCB area | Best | Larger | Larger |
| Expected FPGA power | Lower | Lower | Higher |
| Architecture complexity | Low | Low | Low |
| Expansion margin | Moderate | High | Highest |
| Need for larger FPGA currently demonstrated | No | No | No |

---

# 6. Decision

The selected baseline FPGA is:

**XC7A35T-FGG484**

The final ordering suffix, speed grade, and temperature grade remain TBD.

---

# 7. Rationale

## 7.1 PCIe capability

XC7A35T supports the required PCIe Gen2 x4 endpoint architecture when
used in the FGG484 package.

The device provides four GTP transceiver channels, which is sufficient
for the four required PCIe lanes.

Therefore, moving to XC7A50T does not improve the PCIe interface.

---

## 7.2 FPGA resources

XC7A35T provides:

- 33,280 logic cells
- 90 DSP48E1 slices
- 1,800 Kb of block RAM
- four GTP transceivers

These resources are considered sufficient for the initial Rev A
objective:

- PCIe endpoint
- register/control architecture
- instrumentation logic
- trigger handling
- digital I/O
- moderate internal buffering

The project currently contains no requirement demonstrating that the
additional logic, DSP, or BRAM of XC7A50T is necessary.

Selecting the larger FPGA without such a requirement would add cost and
potentially power without solving an identified engineering problem.

---

## 7.3 Package selection

FGG484 is preferred over CSG325 primarily because of manufacturability
and PCB breakout considerations.

CSG325 uses a 0.8 mm BGA pitch.

FGG484 uses a 1.0 mm BGA pitch.

The larger pitch provides more routing space between BGA pads and makes
the use of conventional mechanically drilled vias more realistic.

This aligns with the Rev A strategy:

**Use standard PCB fabrication technology unless HDI is technically required.**

The final breakout strategy must still be verified against the selected
PCB manufacturer's design rules.

The 1.0 mm package pitch does not by itself guarantee that all routing
can be completed without HDI.

---

# 8. Why XC7A50T Was Not Selected

XC7A50T provides approximately:

- 57% more logic cells
- 33% more DSP slices
- 50% more block RAM

than XC7A35T.

However, both devices provide:

- four GTP transceivers
- one integrated PCIe block
- PCIe Gen2 x4 capability in the selected package family

No Rev A requirement currently needs the additional XC7A50T fabric.

XC7A50T therefore remains a possible future upgrade but is not justified
for the baseline design.

---

# 9. Why CSG325 Was Not Selected

The CSG325 package is attractive because of its smaller 15 mm × 15 mm
footprint.

However, compact board area is not a primary Rev A requirement.

The 0.8 mm pitch increases escape-routing density compared with the
1.0 mm FGG484 package.

For an educational/open hardware project in which PCB
manufacturability and visible routing methodology are important,
FGG484 provides the better engineering trade-off.

---

# 10. Risks

## RISK-001 — Device availability

Artix-7 is a mature FPGA family.

Actual distributor availability and pricing for the intended ordering
code shall be checked before the BOM is frozen.

---

## RISK-002 — BGA breakout

Although the 1.0 mm package improves routing feasibility, conventional
through-via breakout has not yet been proven.

A dedicated BGA escape study shall be performed before the PCB
technology is finalized.

---

## RISK-003 — FPGA capacity

The XC7A35T resource estimate is currently based on architecture rather
than a completed FPGA implementation.

Resource usage shall later be verified in Vivado.

If implementation demonstrates insufficient resources, migration to
XC7A50T or another compatible device shall be evaluated.

---

# 11. Consequences

Selecting XC7A35T-FGG484 establishes the basis for:

- FPGA symbol creation
- FPGA pin planning
- PCIe GTP allocation
- configuration architecture
- power architecture
- decoupling design
- clock architecture
- BGA footprint design
- PCB escape study

The final full part number shall not be frozen until speed grade,
temperature grade, lifecycle, price, and availability are reviewed.

---

# 12. References

Primary design references:

- AMD DS180 — 7 Series FPGAs Data Sheet: Overview
- AMD PG054 — 7 Series FPGAs Integrated Block for PCI Express
- AMD UG475 — 7 Series FPGAs Packaging and Pinout Product Specification
