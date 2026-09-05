# System Architecture — Rev A

**Project:** Open FPGA PCIe Instrumentation Platform  
**Revision:** A  
**Status:** Baseline Architecture  
**Date:** 2026-09-05

---

# 1. Objective

Rev A is a PCI Express FPGA carrier intended to provide a robust,
well-documented hardware platform for:

- PCIe Gen2 x4 communication
- FPGA-based instrumentation
- digital I/O
- external trigger and timing
- FPGA development
- hardware bring-up and validation
- future application-specific expansion

The architecture intentionally prioritizes the core FPGA/PCIe platform
over peripheral feature count.

---

# 2. Top-Level Architecture

```text
                         HOST PC
                           │
                           │ PCIe Gen2 x4
                           │
                  ┌────────▼────────┐
                  │ PCIe Edge       │
                  │ Connector       │
                  │                 │
                  │ TX/RX x4        │
                  │ REFCLK±         │
                  │ PERST#          │
                  │ +12 V / +3V3    │
                  └───┬────┬───────┘
                      │    │
             PCIe     │    │ Power
                      │    │
        ┌─────────────▼─┐  │
        │               │  │
        │   Artix-7     │  │
        │               │  │
        │ XC7A35T       │  │
        │ FGG484        │  │
        │               │  │
        │ GTP Quad      │  │
        │ PCIe Block    │  │
        │ FPGA Fabric   │  │
        │ XADC          │  │
        └──┬──┬──┬──┬──┘  │
           │  │  │  │     │
      ┌────┘  │  │  └────┐│
      │       │  │       ││
      ▼       ▼  ▼       ▼▼

 Configuration  Clocking   External I/O
 & Debug        & Reset    / Instrumentation

 QSPI Flash     Local OSC  Digital GPIO
 JTAG           PCIe CLK   Differential I/O
 USB-UART       PERST#     Trigger Input
 Status LEDs    PGOOD      External Clock

                           │
                           │
                 ┌─────────▼─────────┐
                 │ Power Architecture│
                 │                   │
                 │ PCIe Slot Power   │
                 │        │          │
                 │   DC/DC stages    │
                 │        │          │
                 │ FPGA rails        │
                 │ GTP rails         │
                 │ I/O rails         │
                 └───────────────────┘
