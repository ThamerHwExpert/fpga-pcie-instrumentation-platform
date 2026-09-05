# DEC-002 — PCI Express Architecture

**Status:** Accepted  
**Revision:** A  
**Date:** 2026-09-05

---

## 1. Decision Context

The board shall operate as a PCI Express add-in Endpoint using the
integrated PCIe capability of the selected Artix-7 FPGA.

This decision defines the baseline PCIe architecture before schematic
capture and PCB routing.

The design objectives are:

- PCIe Gen2 operation
- Four-lane endpoint capability
- Robust host compatibility
- Simple clock architecture
- Predictable reset behavior
- Minimum unnecessary sideband complexity
- PCB routing flexibility without compromising link-width fallback
- Straightforward bring-up and validation

---

# 2. Architecture Decision

The Rev A PCIe architecture shall use:

- PCI Express Gen2
- x4 maximum link width
- Endpoint mode
- Standard PCIe add-in card edge interface
- Host-provided PCIe reference clock
- Host-provided PERST#
- Native lane numbering
- No intentional lane reversal
- No intentional polarity inversion unless routing later justifies it

The FPGA PCIe core shall be configured initially for:

**Gen2 x4 Endpoint**

---

# 3. High-Level Architecture

```text
             HOST / ROOT COMPLEX
                     │
              PCIe CEM Slot
                     │
        ┌────────────┼─────────────┐
        │            │             │
        │            │             │
     REFCLK±       PERST#      PCIe x4
        │            │             │
        │            │     ┌───────┴───────┐
        │            │     │               │
        ▼            ▼     ▼               ▼
   MGT REFCLK     Reset/      RX[0:3]    TX[0:3]
     Input       Conditioning
        │            │          │           │
        └────────────┴──────────┴───────────┘
                          │
                          ▼
                  XC7A35T-FGG484
                  Artix-7 GTP Quad
                          │
                          ▼
               Integrated PCIe Block
                          │
                          ▼
                    FPGA Fabric
