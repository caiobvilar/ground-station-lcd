# STM32F469I-DISCO (board_id: `f469-disco`)

MCU: STM32F469NIH6 · Cortex-M4F · 2 MB flash · DSI LCD + SDRAM
Board marking as read by CG, 2026-07-27: **"DK32F469I$AU1"** — recorded
verbatim, not reinterpreted. This likely corresponds to ST's official
`32F469IDISC1` order code for the `32F469IDISCOVERY` board, but that is an
inference, not a read fact — confirm against the physical silkscreen/box
before treating it as settled, same discipline as any other board-identity
claim in this repo.

## On-board ST-LINK / MCU identity

Confirmed by direct `st-info --probe` readout against the physically
connected board (not a vendor-doc lookup — see `docs/hardware/stm32f411e-disco.md`
for why this section is trusted differently than the rest of the sheet).

| Fact | Value | Source | Status |
|---|---|---|---|
| Chip ID | 0x434 | st-info readout, 2026-07-27 | verified — CG |
| Dev type | STM32F46x_F47x | st-info readout, 2026-07-27 | verified — CG |
| Flash size | 2097152 bytes (2 MB), pagesize 16384 | st-info readout, 2026-07-27 | verified — CG |
| SRAM size (st-info's reported figure) | 262144 bytes (256 KB) | st-info readout, 2026-07-27 | verified — CG |
| ST-LINK fw version | V2J45S31 | st-info readout, 2026-07-27 | verified — CG |
| ST-LINK serial | 0671FF515786534867191431 | st-info readout, 2026-07-27 | verified — CG |

Note: `an earlier workbench note` describes this part's total
RAM as "324K" — that figure (if accurate) is a sum across multiple SRAM
regions (main SRAM + CCM + backup SRAM); `st-info`'s single `sram` field
above is evidently just the primary region and is not a contradiction, but
the full memory-region breakdown is still unverified and matters for the
linker script once CubeMX generates one.

> This sheet is otherwise seeded, not verified. Nothing below the section
> above carries `status: verified`, which means **no agent may use any of
> it in code yet**. Verify a row when you need it, one at a time, against
> the document named in its `source` column, then date and initial it. See
> `the meta-repo's hardware-facts provenance rule`.
>
> Seeding a sheet with unverified rows is a to-do list, not a knowledge base.
> The distinction is the whole point.

## Clock

| Fact | Value | Source | Status |
|---|---|---|---|
| HSE source | ? MHz crystal or MCO from ST-LINK | board user manual, clock section | unverified |
| Max SYSCLK | ? | datasheet | unverified |
| PLL config for max | ? | reference manual | unverified |

## User LEDs / button

| Signal | Pin | Active level | Source | Status |
|---|---|---|---|---|
| LEDs (count/pins unknown) | ? | ? | board schematic | unverified |
| User button | ? | ? | board schematic | unverified |

## DSI LCD + SDRAM

Per `board-bringup`: external SDRAM must be initialised in `SystemInit`
before any use of `.bss` placed there, and DSI/LTDC bring-up is its own
subsystem — budget a session for it alone, don't entangle it with
application work.

| Fact | Value | Source | Status |
|---|---|---|---|
| SDRAM part/size/bus width | ? | board schematic + SDRAM part datasheet | unverified |
| DSI panel part | ? | board schematic + panel datasheet | unverified |
| LTDC/DSI pin mapping | ? | board schematic | unverified |

## Debug and serial

| Fact | Value | Source | Status |
|---|---|---|---|
| Programmer | on-board ST-LINK (assumed; confirm revision) | board user manual | unverified |
| Virtual COM port | unknown — confirm before assuming the diag UART has a free path to USB | board user manual | unverified |
| Diag UART plan | to decide once VCP presence is confirmed | — | to decide |

The real probe serial (`0671FF515786534867191431`, from the readout above)
is recorded here. This board cannot reach L3 (HIL) / L4 (acceptance) until
the on-board VCP is confirmed one way or the other.

## Errata

| ID | Summary | Applies to rev | Workaround | Status |
|---|---|---|---|---|
| | | | | |
