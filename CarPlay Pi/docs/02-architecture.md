# 2. System Architecture

## 2.1 How CarPlay on a Pi actually works

Apple doesn't license CarPlay to hobbyist hardware. Every head unit has to prove it's genuine to the iPhone using an Apple MFi authentication coprocessor. The practical workaround is a **CarPlay USB adapter** (Carlinkit CPC200-CCPA, also sold as "Autokit" or under other brands). The adapter:

- holds the MFi chip and handles the CarPlay session, including the Wi-Fi and Bluetooth link for **wireless CarPlay**;
- sends **H.264 video** and **PCM audio** to the host over USB;
- takes **touch events** and **microphone audio** back from the host.

Open-source host software decodes that stream and displays it full-screen:

- **pi-carplay** (`f-io/pi-carplay`), a Pi-focused, actively maintained fork. **This is the recommended starting point.**
- **react-carplay** (`rhysmorgan134/react-carplay`), the original Electron/React app.
- **node-carplay** (`rhysmorgan134/node-carplay`), the underlying library, in case a custom UI is built later.

> Check each repo's README for the current supported adapter firmware and Pi OS version before buying. Adapter firmware updates have broken compatibility before.

## 2.2 Integration options

### Option A — Supplemental screen, factory radio kept (RECOMMENDED for v1)

```
                         2014 Elantra
 ┌──────────────────────────────────────────────────────────────┐
 │                                                              │
 │  Fuse panel ──ACC fuse tap──► [10A inline fuse]              │
 │  (driver kick panel)              │                          │
 │                                   ▼                          │
 │                    [12V→5V 5A automotive buck converter]     │
 │                                   │ 5V                       │
 │        ┌──────────────────────────┼──────────────┐           │
 │        ▼                          ▼              ▼           │
 │  [Ignition sense            [Raspberry Pi 5]  [7" touch      │
 │   opto → GPIO]◄─ACC 12V        │   │   │       display]      │
 │                                │   │   └─DSI/HDMI + USB touch┘
 │                    USB ────────┘   │                         │
 │                    ▼               └── USB ──► [USB mic]     │
 │        [Carlinkit CarPlay adapter]                           │
 │             ((Wi-Fi/BT))  ◄──── iPhone                       │
 │                                                              │
 │  Pi audio ─► [USB DAC] ─► [ground-loop isolator] ─► 3.5mm ─► │
 │                                         Factory radio AUX in │
 │                                         (center console)     │
 └──────────────────────────────────────────────────────────────┘
```

**Pros:** fully reversible. Keeps the steering-wheel audio controls, factory Bluetooth calling, the clock, and the radio. No harness adapters or dash kit needed. Cheapest option.
**Cons:** the radio has to be set to AUX for CarPlay audio. Two screens and two volume controls (use the factory knob as master volume, with the Pi output at a fixed 90 %). The screen is mounted on the dash rather than built in.

**Phone calls in Option A:** pick one approach.
- *A1 (simplest):* pair the iPhone to the **factory Bluetooth** for calls, and use CarPlay for everything else. iOS sends call audio to the car's Bluetooth automatically. The radio switches to the BT phone source by itself during a call.
- *A2:* calls go through CarPlay → Pi → AUX, using the USB mic. This only works when the radio is on AUX, and it has more potential for echo.

The **recommendation is A1.**

### Option B — Full head-unit replacement (FUTURE / v2)

Pull the factory radio. Install a double-DIN dash kit for the 2014 Elantra sedan, plus a Hyundai harness adapter, an antenna adapter, a steering-wheel-control interface (Axxess ASWC-1 or PAC SWI-RC), and a **4-channel amplifier** (the Pi has no speaker-level outputs). The Pi becomes the only head unit. FM radio would need a USB SDR dongle or would be dropped.

**Pros:** clean factory-style look and one audio path.
**Cons:** 3–4× the cost and effort. You'd be giving up the factory radio, CD, and possibly the Bluetooth module. Amp wiring is more complicated. Harder to undo.

> Honestly, if Option B is the end goal, a **$200–300 aftermarket CarPlay head unit** (Sony, Pioneer, Kenwood, or JVC) does it better than a Pi. The Pi build earns its keep through customizability (OBD dashboards, a custom UI, dashcam, tinkering), which Option A gives you.

## 2.3 Software architecture (Option A)

```
┌──────────────────────── Raspberry Pi OS Lite (64-bit, Bookworm) ───────┐
│                                                                        │
│  systemd                                                               │
│   ├─ carplay-kiosk.service ── cage (Wayland kiosk) ── pi-carplay app   │
│   │                                           │  ▲                     │
│   │                       USB (libusb) ◄──────┘  │ touch (evdev)       │
│   │                       Carlinkit adapter      │                     │
│   │                                       PipeWire ─► USB DAC          │
│   ├─ ignition-monitor.service  (GPIO ACC sense → delayed poweroff)     │
│   ├─ brightness.service        (optional: time/LDR → backlight)        │
│   └─ overlayroot / read-only rootfs  (power-loss protection)           │
└────────────────────────────────────────────────────────────────────────┘
```

Details are in [05-software.md](05-software.md).

## 2.4 Key design decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Compute | Raspberry Pi 5 (4 GB) | Hardware H.264 decode is better on Pi 4, but Pi 5's CPU handles 1080p H.264 decode easily. The extra headroom helps with OBD and dashcam later. **Pi 4 (4 GB) is a fine cheaper option.** |
| CarPlay source | Carlinkit CPC200-CCPA adapter | The only realistic path to CarPlay. Supported by pi-carplay. Wireless built in. |
| Display | 7" capacitive, 1024×600 or 1280×720/800, ≥ 400 nits | Fits the dash. CarPlay scales well to it. |
| Audio out | USB DAC → factory AUX | The Pi 5 has no analog jack. A USB DAC is cleaner than HDMI audio extraction. |
| Power | Automotive buck converter + GPIO ignition sense + delayed cut-off | Clean shutdown, zero draw when off. |
| Storage | High-endurance microSD + read-only overlay, or an NVMe/USB SSD | Survives power loss. |
