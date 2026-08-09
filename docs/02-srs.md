# Software/System Requirements Specification — ground-station-lcd

| | |
|---|---|
| Project | ground-station-lcd |
| Version | 1.0 |
| Status | draft |

This SRS is forward-looking: the project has no code yet (scaffolded state).
All requirements are in draft status and will be baselined at G1 review.

## 1. Purpose and scope

The ground-station-lcd project shall be a self-contained ground station / HMI
on the STM32F469NIH6 (F469-DISCO) that brings up the DSI display (LTDC + DSI
host) and external SDRAM, then drives the LCD as the readout surface for the
bench — live values from the EdgeAI trio (tinyml-m0, gesture-imu,
vibration-fault-predict) rendered on-device rather than on a laptop.

## 2. Stakeholders and needs

| Need | Stakeholder | Need text |
|---|---|---|
| N-01 | Owner | LTDC/DSI display + SDRAM initialized in SystemInit before any framebuffer access; live telemetry dashboard rendered at 60 Hz. |
| N-02 | Owner | Framebuffer update latency (protocol receipt → visible update) <= 50 ms at 60 Hz. |
| N-03 | Owner | Host-testable domain logic via port interfaces (no hardware includes in domain). |

## 3. Definitions and abbreviations

- **LTDC/DSI** — LCD-TFT Display Controller / Display Serial Interface.
- **SDRAM init ordering** — External SDRAM must be configured in `SystemInit`
  before any `.bss` placed there is accessed.
- **L1 gate** — Host (native) unit-test gate.

## 4. System context

Board: F469-DISCO (STM32F469NIH6, 2 MB flash / 256 KB+ SRAM + 4 MB SDRAM,
DSI display, FPU). Compute boards (EdgeAI trio) send telemetry over a debug
protocol (UART/SPI, ADR TBD); this project receives and renders.

## 5. Assumptions and constraints

- SDRAM initialization must complete in `SystemInit` before any global
  variables placed in SDRAM are accessed (hardware constraint).
- LTDC/DSI bring-up is a dedicated subsystem; budget a session for it alone.
- This is a display/interface project, not an inference project — the
  constraint is memory bandwidth and clock tree, not RAM budget.

## 6. Requirements

### 6.1 Functional

1. **GS-FUN-001** (shall) — The ground station shall initialize the LTDC/DSI
   display subsystem and external SDRAM in SystemInit before any application
   code accesses the framebuffer.
2. **GS-FUN-002** (shall) — The ground station shall render a live telemetry
   dashboard received from the bench compute boards over the debug protocol.

### 6.2 Performance

3. **GS-PER-001** (shall) — The framebuffer update latency (protocol receipt
   → visible update) shall not exceed 50 ms at 60 Hz refresh.

### 6.3 Interface

4. **GS-INT-001** (shall) — The dashboard renderer shall communicate with the
   receive logic through the port interface without hardware-specific code.

### 6.4 Constraints

5. **GS-CON-001** (should) — The domain code shall compile on the host without
   any hardware or vendor includes.

## 7. Verification summary

| Method | Count |
|---|---|
| Test | 3 |
| Analysis | 2 |
| Inspection | 0 |
| Demonstration | 0 |

*Full traceability is in the generated `docs/06-rtm.md`.*

## 8. Open issues

| ID | Issue | Owner | Target |
|---|---|---|---|
| OI-01 | Debug protocol framing between compute boards and ground station to be defined in ADR. | Owner | before G1 |
| OI-02 | SDRAM/LTDC/DSI clock tree configuration to be validated against F469 datasheet. | Owner | before G1 |
| OI-03 | Framebuffer format (RGB565 vs RGB888) and memory layout to be specified in ADR. | Owner | before G1 |

## 9. Change log

| Version | Date | Change | Commit |
|---|---|---|---|
| 1.0 | 2026-08-09 | Initial draft baseline. | |