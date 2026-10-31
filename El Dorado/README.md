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

Halloween presets (IDs 1–7) live in `el-dorado-halloween-presets.json`; everything else is in `el-dorado-primary-presets.json`. Restoring presets replaces all of them, so merge the two files before restoring.

| ID | Name | Notes |
|---|---|---|
| 1 | Two Dots Halloween | Original preset |
| 2 | Haunted Glitch | Full-white lightning over a dim violet background (Lightning) |
| 3 | Blood Wash | Slow Breathe fade between bright red and a dim blood-red, no flicker |
| 4 | Watching Eyes | Red Halloween Eyes over a dim purple background |
| 5 | **Halloween Haunt** | Looping playlist: Glitch 15 s → Blood Wash 6 s → Glitch 12 s → Eyes 10 s |
| 6 | Witch's Brew | Orange/purple/green palette effect |
| 7 | Pumpkin Patrol | Orange chase on black with purple accent |
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

## Future Ideas

### Astronomy
- Dark-sky mode: one tap, or a schedule on imaging nights, turns everything off so the house doesn’t light up your subs. For star parties or backyard observing, a very dim deep red keeps everyone’s night vision.
- ISS flyover: a short script pulls ISS pass times and sends one bright dot across the roofline in the same direction the station crosses the sky. Good outreach material for SJAA, too.
- Moon phase meter: light the fraction of the front matching tonight’s illumination, in a soft silver.
- Sky compass: treat the roofline as a horizon and map compass directions onto it. Then place colored dots where the Moon and bright planets are rising or setting.
- Meteor shower nights: run the Meteor effect during the Perseids or Geminids, and switch to dark-sky mode once you’re actually out watching.

### Music
- Sound-reactive mode: the front controller already has the AudioReactive feature turned on and pins assigned for a digital microphone. Wire a cheap INMP441 mic and the lights pulse to whatever’s playing. The side can share the same audio data over the network.
- Band nights: point that mic at the garage during rehearsal. Or use LedFx, which takes audio from a computer and streams it to WLED over DDP, the same protocol your side-to-front link already uses.
- Full light-and-music show: xLights lets you program a show to a song, with neighbors tuning in on a small FM transmitter. It’s the classic Christmas setup, and your hardware is already compatible.
Practical but fun
- Porch light zone: warm white over the front door while everything else runs an effect.
Bay Area air quality: shift the house’s color with the local AQI during wildfire season, green through orange to purple.
- Tomorrow’s weather: a quick color glow at dusk, such as blue for rain or white for a cold night.
- Bin-night reminder: a slow green pulse along the front the evening before garbage day.
- Countdowns: an LED bar that fills up toward Halloween, Christmas or a band gig.

### Interactive
- Trick-or-treat jump scare: a hidden button, or a motion sensor on the walkway, fires a burst of Blood Wash when someone reaches the door. Add a speaker for a scream.
- Neighbors pick the effect: a lawn sign with a QR code linking to a simple page with a few safe choices. The controllers shouldn’t be exposed to the internet directly, so this needs a little middleman service.
- Roofline Pong: a two-player game where the ball runs along the eaves.
- Holidays and the full house
- Ready-made sets for the year: candy-cane chase and slow warm twinkle for Christmas, Fireworks for the 4th, green waves for St. Patrick’s Day, and team colors on game days.
- Sunset schedule: have the lights come on automatically at sunset. This needs the time server turned on and your location set on the side controller; it’s currently off.
- Whole-house chases: once sections 3 and 4 are in, a ghost or comet can circle the whole house without breaking.
