# 3. Hardware & Bill of Materials

Prices are rough USD street prices and will vary. "Verify" marks a part number or fitment you should confirm for your exact car or software version before ordering.

## 3.1 Core BOM (Option A)

| # | Part | Recommended spec / example | Qty | Est. $ | Notes |
|---|------|----------------------------|-----|--------|-------|
| 1 | Single-board computer | **Raspberry Pi 5, 4 GB** (alt: Pi 4 B 4 GB) | 1 | 60 | 2 GB works but leaves no headroom. |
| 2 | Pi cooling | Official Pi 5 Active Cooler | 1 | 5 | Needed for summer cabin temperatures. |
| 3 | CarPlay adapter | **Carlinkit CPC200-CCPA** (wireless CarPlay + Android Auto) | 1 | 60–90 | Verify the model and firmware against the pi-carplay README. Buy from a seller with easy returns. |
| 4 | Touch display | 7" IPS capacitive, **high-brightness (≥ 800 nits preferred)**. Options: **Raspberry Pi Touch Display 2** (7", 720×1280, DSI, single-cable power and touch) or a Waveshare 7" HDMI IPS (1024×600, USB touch) | 1 | 60–110 | DSI is tidier. HDMI is easier to run over longer cables if the Pi sits in the glovebox. **See 3.3.** |
| 5 | Storage | **SanDisk High Endurance / Samsung PRO Endurance 32–64 GB microSD**, or an NVMe HAT + 128 GB SSD | 1 | 12 (SD) / 35 (NVMe) | Use "endurance" or industrial-rated cards. |
| 6 | 12V→5V converter | **Automotive buck converter, 8–36 V in, 5.1 V / 5 A (25 W) out**, potted/waterproof | 1 | 15–20 | The Pi 5 needs 5 A to run USB peripherals at full current. Set `usb_max_current_enable=1` (see software doc). |
| 7 | Ignition sense + shutdown control | **Option 1 (recommended):** purpose-built car power controller, e.g. a Mausberry Circuits car supply/switch, or similar "Pi car power" boards with ACC sense and timed cut-off. **Option 2 (DIY):** PC817 optocoupler module (12V→GPIO) + 12V automotive relay or high-side MOSFET module for a delayed cut-off. | 1 | 15–35 | See [04-electrical.md](04-electrical.md). |
| 8 | Fuse tap | **Add-a-circuit fuse tap**, matching the car's fuse type (2014 Elantra interior panel: **mini (ATM) / low-profile mini** fuses, verify by pulling one) | 2 | 8 | One for ACC power, one optional for the ACC sense line. |
| 9 | Inline fuse holder | ATC/ATM inline holder, 16 AWG, with a **5A fuse** for the converter feed | 1 | 5 | |
| 10 | USB audio DAC | Any UAC-compliant USB DAC with 3.5 mm out (Apple USB-C→3.5mm dongle + USB-A adapter works well, or a Sabrent-type USB sound card) | 1 | 10 | |
| 11 | Ground-loop isolator | 3.5 mm audio ground-loop noise isolator | 1 | 8 | Fixes alternator whine. |
| 12 | 3.5 mm AUX cable | Male-male, length to reach the console AUX jack | 1 | 6 | |
| 13 | USB microphone | Small clip-on USB lavalier or mini USB mic, mounted near the headliner or A-pillar | 1 | 10–15 | Only needed for Siri / CarPlay calls (A2). Many adapters use the host mic. |
| 14 | Dash mount | Phone/tablet vent or dash mount sized for a 7" screen, **or** a custom 3D-printed bezel + 17 mm ball mount | 1 | 15–30 | See [06-mechanical.md](06-mechanical.md). |
| 15 | Enclosure for the Pi | Vented case with room for the cooler (glovebox or under-dash) | 1 | 10 | |
| 16 | Wiring sundries | 18 AWG primary wire (red/black/yellow), heat-shrink butt connectors, ring terminal for ground, split loom, zip ties, Tesa cloth harness tape | — | 20 | Cloth tape stops rattles behind the dash. |

**Core total: about $230–$330**, depending on display and storage choices.

## 3.2 Optional add-ons (later phases)

| Part | Purpose | Est. $ |
|------|---------|--------|
| Bluetooth ELM327 OBD-II adapter (OBDLink LX/MX+ recommended over cheap clones) | F-09 live gauges and codes | 25–80 |
| USB UVC backup camera, or an analog camera + USB capture dongle | F-10 reverse camera | 25–40 |
| Pi Camera Module 3 (wide) + long ribbon | F-11 dashcam | 35 |
| BH1750 light sensor (I²C) | F-08 auto-brightness | 5 |
| Supercapacitor / small UPS HAT | Rides through cranking dips without rebooting | 25–40 |
| NVMe HAT + SSD | Faster boot and better power-loss tolerance | 35 |
| Steering-wheel-control interface + resistor-ladder ADC (ADS1115) | F-12 (advanced, needs tapping SWC wires) | 10–60 |

## 3.3 Display selection notes

- **Brightness matters most.** Standard 250–300 nit hobby panels are hard to read in daylight. Look for "high brightness" or "sunlight readable" 7" panels, 800–1000 nits.
- **Resolution:** CarPlay renders at whatever resolution the host requests. 1024×600 and 1280×720/800 both look good at 7".
- **Touch:** capacitive only (resistive feels terrible). USB HID touch works with no configuration.
- **Size:** 7" fits the top of the Elantra's dash in front of the center vents without blocking them. Measure first (see the mechanical doc).
- **Aspect:** the Pi Touch Display 2 is natively portrait (720×1280), so rotate it to landscape in software (`display_lcd_rotation` / Wayland transform). Confirm pi-carplay handles the rotated output, or use an HDMI landscape panel.

## 3.4 Tools needed

Multimeter, trim/panel removal tool set, wire stripper/crimper, heat gun or lighter for heat-shrink, 10 mm socket (ground point), flush cutters, and a computer with Raspberry Pi Imager. A soldering iron is optional.
