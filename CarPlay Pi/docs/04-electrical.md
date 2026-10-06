# 4. Electrical Specification

> ⚠️ Disconnect the negative battery terminal before tapping fuses or running power wires. Fuse every new 12V connection as close to the source as possible.

## 4.1 Power sources in the 2014 Elantra

- **Interior fuse panel:** driver's side, behind the lower dash/kick panel cover, left of the steering column. The fuse-box lid diagram and the owner's manual list each circuit.
- **What you need:**
  - **ACC (switched) circuit:** live with the key in ACC or ON, dead with the key off. Common choices are the circuit feeding the **power outlet / cigar lighter** or the **audio** fuse. **Verify with a multimeter.** Probe both sides of the fuse with the key OFF (should read 0 V) and the key in ACC (should read about 12 V).
  - **Ground:** a bare-metal chassis bolt under the dash, using a ring terminal. Don't use painted surfaces or plastic.
- **Load budget:** the Pi 5 plus display, adapter, and DAC peak around 15–20 W at 5 V, which is about **1.5–2 A at 12 V**. That fits easily on a 15–20A accessory circuit. **Fuse the new branch at 5A** on the tap.

## 4.2 Power architecture

Two valid designs. **Design 1 is recommended** because it's simple and needs no custom electronics.

### Design 1 — Purpose-built car power controller (recommended)

```
Battery ─(constant 12V fuse)─► [Car power controller] ─5V─► Pi 5V/GND
ACC fuse tap ─────────────────► [  ACC/IGN sense in   ]
                                [  GPIO handshake     ]◄──► Pi GPIO
```

These boards (e.g. Mausberry-style "car switch" supplies) take **constant 12V + ACC sense**. When ACC drops, they signal the Pi over GPIO, wait for the Pi to report it's halted (or a timeout), and then **cut power completely**. Follow the board's own wiring guide and use its shutdown script, or the one in [05-software.md](05-software.md).

*Note:* this design uses a **constant** 12V feed. The board's off-state draw must be under 1 mA (check its datasheet) to meet requirement E-03.

### Design 2 — DIY: ACC-powered + GPIO sense + hold-up (no constant feed)

```
ACC fuse tap ─[5A fuse]─► [12V→5V 5A buck] ─5V─► Pi 5 (USB-C or GPIO 5V pins)
                │
                └──[1kΩ]──► PC817 opto LED ──► GND
                            PC817 transistor: collector → GPIO17 (internal pull-up), emitter → GND
```

- When ACC turns off, the converter loses input, so the **Pi has to shut down within the hold-up time** of whatever stores energy. Without storage that time is zero, so **Design 2 needs one of these:**
  - **(a)** a **supercap/UPS HAT** that gives about 20–30 s of runtime after input loss, or
  - **(b)** feed the converter from **constant 12V through a relay**. The relay coil is energized by ACC **OR** by a "keep-alive" GPIO (through a transistor). The Pi holds the relay on, sees ACC drop, shuts down, and the GPIO releases at halt, which cuts power. (This is what Design 1 boards do internally.)
- Use an **optocoupler** for ACC sense so 12V spikes never reach a 3.3 V GPIO.

### Recommended Design 1 wiring summary

| From | To | Wire | Fuse |
|------|----|------|------|
| Constant 12V fuse tap (e.g. dome/memory circuit, verify) | Controller 12V IN | 18 AWG red | 5A |
| ACC fuse tap | Controller ACC/IGN sense | 20–22 AWG yellow | 1–2A |
| Chassis ground bolt | Controller GND | 18 AWG black | — |
| Controller 5V OUT | Pi 5V + GND (GPIO pins 2/6 or USB-C) | 18 AWG, as short as possible | — |
| Controller signal lines | Pi GPIO (per board docs, e.g. GPIO17 in / GPIO27 out) | jumper | — |

## 4.3 Pi 5 power notes

- The Pi 5 asks USB-C PD for 5 A. A buck converter can't negotiate PD, so add `usb_max_current_enable=1` to `/boot/firmware/config.txt`. Otherwise the USB ports are limited to 600 mA total, and the CarPlay adapter, display touch, and DAC may brown out.
- Keep the 5V run to the Pi **under about 50 cm with 18 AWG** so the voltage drop stays below 0.1 V. Watch for the undervoltage warning (`vcgencmd get_throttled` should return `0x0`).
- Power the display from the Pi's USB, or with a short Y-split from the same 5V rail. **Don't** use a separate converter on a different ground (ground loops).

## 4.4 Cranking behavior

- In ACC → START, most cars **drop ACC** while cranking. The Pi will see an ACC "off" pulse lasting 1–3 s.
- **Shutdown logic must debounce:** only start shutdown if ACC has been off for **at least 5 s** continuously (see the software doc). The controller or UPS rides through the pulse.

## 4.5 Audio wiring

```
Pi USB ─► USB DAC ─3.5mm─► ground-loop isolator ─3.5mm─► Console AUX jack
```

- Set the Pi's software volume to a fixed about 90 % and use the **factory volume knob** as master volume.
- If you hear whine that changes with engine RPM: (1) the isolator should fix it; (2) make sure the Pi and display share **one** ground point; (3) as a last resort, use an isolated DC-DC converter.

## 4.6 Signal and data connections

| Link | Interface | Cable |
|------|-----------|-------|
| Pi ↔ CarPlay adapter | USB 2.0/3.0 (USB-A) | Short; the adapter can sit next to the Pi |
| Pi ↔ display video | DSI (Touch Display 2) or micro-HDMI → HDMI | DSI flex ≤ about 30 cm is reliable. HDMI up to 2–3 m. |
| Pi ↔ display touch | Over DSI, or a USB cable | — |
| Pi ↔ microphone | USB | Route up the A-pillar trim, clipped near the headliner/visor |
| Pi ↔ OBD (later) | Bluetooth (ELM327) | — |

## 4.7 Electrical acceptance checks

- [ ] Key OFF: current draw through the new circuit **< 1 mA**. Measure with the meter in series on the constant feed once everything is fully shut down.
- [ ] Key ACC: system boots. `vcgencmd get_throttled` → `0x0` after 10 minutes of CarPlay video.
- [ ] Crank the engine: the system doesn't start a shutdown.
- [ ] Key OFF: clean shutdown in ≤ 20 s, then power is fully cut.
- [ ] No hum or whine on AUX with the engine at idle and at 3000 RPM.
