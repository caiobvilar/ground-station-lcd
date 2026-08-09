# ground-station-lcd — DSI/LTDC + SDRAM display ground station

**A self-contained ground-station / HMI on an STM32F469I-DISCO — DSI display
(LTDC + DSI host) and external SDRAM brought up as the bench's readout surface
for the EdgeAI trio — scaffolded; the ST-LINK/MCU identity is verified, and
the SDRAM/DSI facts that gate bring-up are explicitly not yet.**

The display/interface half of the portfolio: a place where a model's output is
actually shown to a human. Not an inference project — the interesting
engineering is the LTDC/DSI subsystem, external-SDRAM initialization ordering
in `SystemInit`, and the framing protocol that carries live values from the
compute boards to the screen.

> **Status: migrated scaffold.** No code yet. The board's MCU/ST-LINK identity
> is verified by tool readout; everything that gates this project (SDRAM part,
> DSI panel, LTDC/DSI pins, VCP) is documented as unverified and must be
> confirmed before bring-up.

## What this demonstrates (planned)

| | |
|---|---|
| **Hardware** | DSI LCD + LTDC, external SDRAM (init in `SystemInit`), F469 clock tree |
| **Firmware** | Framebuffer management, DSI/LTDC register build-up, framing protocol to compute boards |
| **Integration** | Readout surface for the EdgeAI trio — model output shown to a human |

## Verified facts in hand

| Fact | Value | Source |
|---|---|---|
| MCU | STM32F469NIH6, chip 0x434, 2 MB flash / 256 KB SRAM | st-info readout, 2026-07-27 |
| ST-LINK | V2J45S31, serial 0671FF515786534867191431 | st-info readout, 2026-07-27 |

Everything else — SDRAM part/size, DSI panel, LTDC/DSI pins, VCP, clock
limits — is **unverified** in [docs/hardware/stm32f469i-disco.md](docs/hardware/stm32f469i-disco.md).

## Architecture (planned)

```
EdgeAI boards (tinyml-m0 / gesture-imu / vibration-fault-predict)
        │  framing protocol (to be decided: UART / existing debug protocol)
        ▼
STM32F469 ◄── external SDRAM (SystemInit init order matters)
        │
        ▼
   LTDC + DSI host ──► DSI panel  (the readout surface)
```

## Repository layout

| Path | Contents |
|---|---|
| `src/domain/` | Framebuffer layout, register build-up logic — host-testable |
| `src/ports/` | Display + SDRAM + comms interfaces |
| `src/adapters/` | Real (stm32f4) + host fake implementations |
| `test/unit/` | Unity host tests |
| `docs/adr/` | Toolchain decisions (0001, 0003) |
| `docs/hardware/` | STM32F469I-DISCO fact sheet |

## Build and run

Not yet buildable — no CMake project exists, and the SDRAM/DSI facts that
gate the `SystemInit` and bring-up code are still unverified. SRS + G1 come
first.

## Documentation

- [PLAN.md](PLAN.md) — durable state, open questions, log
- [docs/hardware/stm32f469i-disco.md](docs/hardware/stm32f469i-disco.md) — board facts (identity verified; display/SDRAM rows pending)

## What I'd do differently

- The scaffold began before the display facts were gathered; the honest
  ordering is: schematic + datasheets first, then a fact sheet that is
  verified where it matters, then bring-up. The sheet now carries that
  distinction explicitly instead of a confident guess.
- SDRAM-in-`SystemInit` ordering is called out in the plan as a dedicated
  session — experience from other projects suggests display bring-up
  entangles itself with application work if it isn't kept separate.

## License

Code: **Apache-2.0** (`LICENSE`) · Documentation: **CC BY 4.0**
(`docs/LICENSE-docs.md`) — per the program's publishing rules
([06-publishing.md §2.3](https://github.com/caiobvilar/Embedded_Portfolio/blob/main/06-publishing.md)).
