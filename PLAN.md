# PLAN — ground-station-lcd

> **Migrated 2026-08-09** from the workbench harness repo into the portfolio
> meta-repo. Now `projects/T5-capstones/ground-station-lcd`, following the
> program's reference repository layout (`03-toolchain.md §2`). Gate numbers
> use the program's four test levels (L1 unit, L2 SITL, L3 HIL, L4 acceptance)
> from `03-toolchain.md §3`.

Durable state across context windows. Read this first; update it before you
stop. If a future session cannot resume from this file alone, it is not
detailed enough.

## Goal

A self-contained ground-station / HMI on `f469-disco`: bring up the DSI
display (LTDC + DSI host) and external SDRAM, then drive the LCD as the
readout surface for the bench — live values from the EdgeAI trio
(`tinyml-m0` / `gesture-imu` / `vibration-fault-predict`) rendered on-device
rather than on a laptop. This is the fifth leg of the portfolio: `tinyml-m0`
proves fitting a model under a memory budget, `gesture-imu` proves live sensor
classification, `vibration-fault-predict` proves the industrial
fault-detection framing, `kiln-supervisor` proves an industrial-comms
(RS-485/Modbus) client, and this one proves the display/interface half — the
"10+ models in production" CV line needs a place where a model's output is
actually shown to a human, and a DSI LCD + SDRAM ground station is that place.

Placed in **T5** (capstones): this is a system-integration project — the
bench's readout surface, companion to the HIL rig (P18) and the flight
controller ground station (P19), per [13-T5-capstones.md](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/13-T5-capstones.md).

**This is a display project, not an inference project.** The interesting
constraint is memory-bandwidth and clock tree, not RAM budget — the F469 has
2 MB flash / 256 KB+ SRAM and an FPU. What makes it worth a project is the
LTDC/DSI subsystem (its own bring-up session), external SDRAM initialization
ordering in `SystemInit`, and the framing protocol between the compute boards
and this display.

## Target

- Board: `f469-disco` (STM32F469NIH6 — see `docs/hardware/stm32f469i-disco.md`;
  ST-LINK/MCU identity verified via `st-info` 2026-07-27)
- Highest gate reached: **none — scaffolded, no code yet**
- Constraint that drives the design: external SDRAM must be initialized in
  `SystemInit` before any use of `.bss` placed there, and LTDC/DSI bring-up
  is its own subsystem — budget a session for it alone, don't entangle it
  with application work.

## Steps

Tick as gates pass, not as code is written. Gates per [02-process.md](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/02-process.md),
test levels per [03-toolchain.md §3](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/03-toolchain.md).

- [ ] **SRS + G1.** EARS requirements (framebuffer layout, refresh, framing
      protocol, what the readout shows) before code. Seed `docs/02-srs.md` +
      `docs/requirements/`.
- [ ] SDRAM + DSI/LTDC bring-up per `board-bringup` — readback check (SDRAM
      write/read a known pattern; DSI panel ID via DSI command, not just
      "panel lights up")
- [ ] L0 builds cross + host
- [ ] Static analysis clean, deviations recorded in `docs/adr/`
- [ ] L1 host unit tests — framebuffer layout, DSI/LTDC register build-up
      logic tested without hardware
- [ ] L2 SITL (Renode) — selftest runs in emulation (SDRAM model permitting;
      a `.repl` extension may be needed — see 03-toolchain.md §3.2)
- [ ] L3 HIL — flashes, `?selftest` passes — SDRAM pattern readback +
      panel-on confirmed on real hardware
- [ ] L4 acceptance — human sees the display actually render a known
      framebuffer, and (once the framing protocol lands) live values from a
      compute board

## Decisions

Link ADRs. Do not restate them here.

- Toolchain: [docs/adr/0001-cubemx-cmake-toolchain.md](docs/adr/0001-cubemx-cmake-toolchain.md) applies as to every STM32
  project — CubeMX-generated CMake project, not hand-written linker/startup.
- Not yet decided: framing protocol between the compute boards and this
  display (UART vs what the boards' debug protocol already speaks), and what
  "the readout surface" shows first (bare framebuffer test pattern, then a
  real metric).

## Open questions

Facts you need and do not have. Name the document and section. This is the
handoff list — the human answers these between sessions.

- [ ] SDRAM part, size, and bus width — `docs/hardware/stm32f469i-disco.md`
      has it unverified; get it from the board schematic + the SDRAM part
      datasheet before writing the `SystemInit` init sequence.
- [ ] DSI panel part and its datasheet (resolution, DSI lanes, init sequence)
      — unverified in the sheet.
- [ ] LTDC/DSI pin mapping — board schematic, unverified.
- [ ] On-board ST-LINK VCP presence — unverified; determines whether the
      debug UART has a free path to USB or needs an external adapter.
- [ ] Max SYSCLK / PLL config — DS10314 (F469 datasheet) / RM0386, unverified.
- [ ] User LEDs / button pins — board schematic, unverified.
- [ ] Full memory-region breakdown (the "324K" total-RAM figure vs `st-info`'s
      256 KB primary SRAM) — matters for the linker script once CubeMX
      generates one.
- [ ] Run STM32CubeMX for `f469-disco`, generate the CMake project (per ADR
      0001), and commit its output under `cubemx/` before hand-written code
      goes on top.
- [ ] Write the SRS (`docs/02-srs.md`) and sign G1 — blocks any code.

## Log

Newest last. One line per session: what moved, what broke, where you stopped.

- `2026-08-09` — Migrated from workbench `projects/05-hmi-lcd` into
  `projects/T5-capstones/ground-station-lcd` per the program layout. Content
  unchanged; ADR + hardware sheet copied in; gate vocabulary updated.
- `2026-08-09` (original scaffold) — Project scaffolded from `00-template`.
  No code yet. Board fact sheet (`docs/hardware/stm32f469i-disco.md`) exists
  but is seeded, not verified — the st-info MCU/ST-LINK identity rows are
  verified, everything that gates this project (SDRAM part, DSI panel,
  LTDC/DSI pins, VCP) is not.
