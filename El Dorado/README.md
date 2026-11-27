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

- **North controller (master):** renders all 970 LEDs (465 in the older two-section layout below).
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

Halloween presets (IDs 1–8) live in `el-dorado-halloween-presets.json`, Thanksgiving presets (IDs 43–50) in `el-dorado-thanksgiving-presets.json`, and everything else in `el-dorado-primary-presets.json`. Every file includes the layout presets (37–42), so each works on its own. Restoring presets replaces all of them, so to restore everything, merge the files first (the layout entries are identical and the other IDs don't overlap).

### Thanksgiving (IDs 43–50)

Oranges, maroons, browns and harvest gold, with slow effects and 4–6 s crossfades.

| ID | Name | Notes |
|---|---|---|
| 43 | Harvest Waves | Colorwaves over a smooth pumpkin → maroon → gold gradient |
| 44 | Cornucopia | Palette effect slowly drifting pumpkin, chestnut brown and cranberry along the roofline |
| 45 | Falling Leaves | Twinklefox using the Autumn palette over a dim brown background |
| 46 | Maple Drift | Lake effect using the Orangery palette, a slow ripple of reds and oranges |
| 47 | Cranberry & Gold | Slow Breathe between harvest gold and maroon (the Thanksgiving counterpart to Blood Wash) |
| 48 | Harvest Aurora | Aurora waves of amber, maroon and brown |
| 49 | Hearth | Candle Multi in warm amber, a gentle fireplace flicker for dinner |
| 50 | **Thanksgiving Gathering** | Looping playlist: 43 → 45 → 44 → 47 → 46 → 48, 2–3 min each with 6 s crossfades |

- The Autumn palette (45) includes a few yellow-green tones, like leaves still turning. If you want strictly orange, maroon and brown, switch 45 to the Color Gradient palette.
- Browns on LEDs look like dim amber or chestnut. They read best next to the brighter oranges.
- **51 Lights Off** turns the lights off. It's in every presets file, so the 11 PM timer works whichever file is restored.

## Schedule (North controller)

- **Sunset:** applies preset 50 (Thanksgiving Gathering) every day. To use a different show, change the sunset timer's preset in Config → Time & Macros.
- **11:00 PM:** applies preset 51 (Lights Off) every day.
- The time server is on (`0.wled.pool.ntp.org`, US-Pacific with daylight saving). The location is San Jose city center (37.34, -121.89), which is close enough for sunset times.
- Timer order matters on this firmware (0.14.4). The list in `cfg.json` fills the regular slots first, then the **second-to-last hour-255 entry is sunrise and the last is sunset**. A sunset entry listed first ends up as sunrise. Check the result in Config → Time & Macros after editing.
- Applying preset 51 from a timer also stops the running playlist, so the lights stay off (tested).
- The sunset timer depends on the time server. If the master shows a wrong time (in /json/info), check that it's online and that NTP is still enabled.

| ID | Name | Notes |
|---|---|---|
| 1 | Two Dots Halloween | Original preset |
| 2 | Haunted Glitch | Full-white lightning over a dim violet background (Lightning) |
| 3 | Blood Wash | Slow Breathe fade between bright red and a dim blood-red, no flicker |
| 4 | Watching Eyes | Red Halloween Eyes over a dim purple background |
| 5 | **Halloween Haunt** | Looping playlist: Glitch 15 s → Blood Wash 6 s → Glitch 12 s → Eyes 10 s |
| 6 | Witch's Brew | Orange/purple/green palette effect |
| 7 | Pumpkin Patrol | Orange chase on black with purple accent |
| 8 | Eyes Front, Blood Wash Side | Split layout: Blood Wash on the side (0–244), Watching Eyes on the front (245–464) |
| 37 | Layout: Whole House | One segment, LEDs 0–464 (the default) |
| 38 | Layout: Split (Side + Front) | Two segments: Side 0–244 and Front 245–464, both on |
| 39 | Layout: Side Only | Split layout with the front segment off |
| 40 | Layout: Front Only | Split layout with the side segment off |
| 11–36 | Imported scenes | From illiam08's *Ultimate WLED Scenes Collection* (Starry Night … Desert Sky) |

- All presets other than preset 8 and the layout presets (37–40) cover segment 0 = LEDs **0–465**, so applying one puts the house back in whole-house mode.
- Layout presets only change segment bounds and on/off, not effects. Apply **Split**, then pick an effect for each segment in the UI. Applying **Side Only** or **Front Only** keeps the effects and just blanks the other part.
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
- Whole-house chases: once sections 3 and 4 are in, a ghost or comet can circle the whole house without breaking.


## Four-section update (North, East, South, West)

`el-dorado-north-side-wled-cfg.json` is the master and now renders **970 LEDs** as one virtual strip:

| LEDs | Section | Controller | IP | Output |
|---|---|---|---|---|
| 0–244 | North | master, GPIO16 | DHCP (reserve it) | local |
| 245–464 | East | `wled-8ddaa4` | 192.168.50.152 | DDP |
| 465–719 | South | `south-side-wled` | 192.168.50.33 | DDP |
| 720–969 | West | `west-back-wled` | 192.168.50.173 | DDP |

- Section order is N → E → S → W. If the sections are chained differently around the house, reorder the DDP outputs and the layout presets.
- All effect presets cover segment 0 = 0–970. Northern Lights 2 (27), Desert Sky (36) and preset 8 keep North on segment 0 and now run segment 1 over East + South + West (245–970).
- Layout presets: 37 Whole House (one segment), 38 Split (four segments), 39 North Only, 40 East Only, 41 South Only, 42 West Only.
- Reserve the South and West IPs in the router, same as East.
- South's Max current is now 12,000 mA, matching the others.
