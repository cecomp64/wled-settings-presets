# House WLED Lighting

Permanent addressable lighting on the house, driven by WLED. Two of four sections are installed (side and front). Both act as **one virtual strip of 465 LEDs**, controlled from the side controller.

## Hardware

| Section | Controller | IP | Power | Length | LEDs (ICs) | Outputs |
|---|---|---|---|---|---|---|
| Side | `wled-34f6b4` | DHCP (reserve it) | 24V 350W PSU | ~40 ft | 245 | GPIO16, one serial run |
| Front | `wled-8ddaa4` | 192.168.50.152 | 24V 350W PSU | ~40 ft | 220 | GPIO16 (110) + GPIO2 (110) |

- Each run draws about **185 W (≈7.7 A at 24 V)** at full white, so each PSU (≈14.6 A) has roughly 2× headroom.
- On the front, the controller sits in the middle of the run:
  - **GPIO16** runs from the middle toward the side (reversed in software).
  - **GPIO2** runs from the middle outward.
- Each front strip got 10 extra LEDs at its outer end, so 100 became 110 per strip.
- Network: WiFi `CELG204 Asus`, 192.168.50.x.

## Virtual strip layout

```
LED index:  0 ─────────── 244 | 245 ───────── 354 | 355 ───────── 464
            Side (GPIO16)     | Front GPIO16 (rev)| Front GPIO2
            side controller   |  ── sent over DDP to the front controller ──
```

- **Side controller (master):** renders all 465 LEDs.
  - Output 1: GPIO16, LEDs 0–244.
  - Output 2: virtual **DDP** output, LEDs 245–464 → `192.168.50.152`.
- **Front controller:** receives DDP data and goes into live mode automatically.
  - Its own outputs: GPIO16 0–109 (Reverse on), GPIO2 110–219.

To control the whole house, use the **side controller**. The front's own UI only matters for standalone testing.

## Brightness limiter

| | Side | Front |
|---|---|---|
| LED current (custom) | 32 mA | 35 mA |
| Max current | 12,000 mA | 12,000 mA |

- The per-LED value is 185 W ÷ 24 V ÷ number of ICs. The cap is about 80% of the PSU's rating, so the limiter only kicks in as protection.
- To dim the lights, use the brightness slider or each preset's brightness, not the cap.
- As far as I know, WLED leaves virtual (DDP) outputs out of its power estimate, so the side budgets only its own 245 LEDs.

## Presets (side controller)

| ID | Name | Notes |
|---|---|---|
| 1 | Two Dots Halloween | Original preset |
| 2 | Haunted Glitch | Full-white lightning over a dim violet background (Lightning) |
| 3 | Blood Surge | Red lightning at varied brightness over a dim blood-red glow |
| 4 | Watching Eyes | Red Halloween Eyes over a dim purple background |
| 5 | **Halloween Haunt** | Looping playlist: Glitch 15 s → Blood Surge 6 s → Glitch 12 s → Eyes 10 s |
| 11–36 | Imported scenes | From illiam08's *Ultimate WLED Scenes Collection* (Starry Night … Desert Sky) |

- All presets cover segment 0 = LEDs **0–465**.
- Northern Lights 2 (27) and Desert Sky (36) are split: side is 0–245, front is 245–465.
- When saving a preset, keep **Save segment bounds** ticked so it keeps covering 0–465.

## Files

| File | Restore on | How |
|---|---|---|
| `side_wled_cfg.json` | Side | Config → Security & Updates → Restore configuration |
| `front_wled_cfg.json` | Front | Same as above |
| `side_wled_presets.json` | Side | Config → Security & Updates → Restore presets |

- Restore the front config first, then the side config, then the presets.
- Back up the current files before restoring.

## Maintenance notes

- **Reserve both controllers' IPs** in the Asus router. The DDP link breaks if the front's IP changes.
- **Adding sections 3 and 4:**
  1. On the side controller, add one more virtual DDP output per section, continuing the LED numbering from 465.
  2. Raise the side's total LED count.
  3. Update every preset's segment stop to the new total.
- **Changing LED counts on the front:**
  1. Update the front's output lengths and starts.
  2. Update the length of the side's virtual output.
  3. Update the side's total LED count.
  4. Update every preset's segment stop.
- **Wiring for added LEDs:** a new piece's data arrows must point away from the controller (DO → DI). Reverse in software doesn't change the physical data direction.
- **Hardware to-dos:**
  - If white looks tinted at the far end of the side run, inject power there.
  - Put a 10 A inline fuse on the positive line of each run.
- **Importing community presets:** don't upload someone else's full `presets.json`, because it replaces yours. Add each preset to a free slot through the preset editor's API command box, then set its segment to 0–465.
