# CarPlay Pi — Raspberry Pi Apple CarPlay Smart Screen

**Vehicle:** 2014 Hyundai Elantra SE (sedan, MD generation, 2011–2016 platform)
**Goal:** Add a touchscreen to the car that runs Apple CarPlay (wireless preferred), powered by a Raspberry Pi, without breaking any of the car's factory features.

## Documents

| # | Document | What it covers |
|---|----------|----------------|
| 1 | [Requirements](docs/01-requirements.md) | What the system must and should do, and the constraints it works within |
| 2 | [Architecture](docs/02-architecture.md) | The two integration options, the recommended one, and the system block diagram |
| 3 | [Hardware & BOM](docs/03-hardware-bom.md) | Every part, why it was picked, and the estimated cost |
| 4 | [Electrical](docs/04-electrical.md) | 12V tap, 5V supply, ignition sensing, safe shutdown, audio wiring, fusing |
| 5 | [Software](docs/05-software.md) | OS image, CarPlay stack, boot-to-CarPlay, shutdown daemon, hardening |
| 6 | [Mechanical & Mounting](docs/06-mechanical.md) | Where the screen and Pi go, enclosure, heat, cable routing |
| 7 | [Build Plan & Test Plan](docs/07-build-and-test.md) | Phased build steps, acceptance tests, and the rollout order |

## The short version

1. **CarPlay requires Apple-certified hardware.** A Raspberry Pi cannot run CarPlay alone because Apple's protocol needs an MFi authentication chip. The usual approach is a **CarPlay USB adapter** (Carlinkit CPC200-CCPA or a similar "Autokit" dongle). It handles authentication and wireless CarPlay. The Pi runs open-source software (**pi-carplay** or **react-carplay**) that shows the adapter's video stream on the touchscreen and sends touches back to it.
2. **Recommended integration (Option A):** keep the factory radio. Add a 7" dash-mounted touchscreen. Send CarPlay audio into the factory **AUX input** in the center console. This approach is low-risk, fully reversible, and keeps the factory steering-wheel controls, Bluetooth calling, and clock working.
3. **Power:** a fused **add-a-circuit tap** on an ignition-switched (ACC) fuse feeds a **12V→5V 5A buck converter**. The Pi detects when the ignition goes off and **shuts itself down cleanly**, so the SD card doesn't get corrupted and the battery doesn't drain.
4. **Budget:** about **$230–$330** for the core build (see the BOM).
5. **Phased build:** bench prototype → in-car power and shutdown → in-car CarPlay and audio → final mounting → optional extras (OBD-II gauges, backup camera, full head-unit replacement).

## Open questions to confirm before buying parts

- [ ] **Which factory radio does the car have?** The 2014 Elantra SE came with the standard AM/FM/CD/MP3 unit, some with a small 4.3" display and rear camera, and a 7" navigation unit was optional. A picture of the dash settles this. It affects Option B, mounting, and whether a factory backup camera feed exists.
- [ ] **Is there a working AUX jack** in the center-console bin (next to the USB port)? Option A's audio path needs it.
- [ ] **iPhone model / iOS version.** Wireless CarPlay needs an iPhone 5 or later, and a current iOS is assumed.
- [ ] **Where should the screen go:** the top of the dash center, or the left/right of the center vents? See the mechanical doc.
- [ ] **Comfort level with car wiring.** Option A only needs a fuse tap. Option B means pulling the radio and working with harness adapters.

## Safety and legal notes

- Don't mount anything in an airbag deployment zone or where it blocks the driver's view of the road. Many US states limit what can be mounted on the windshield or dash.
- Every 12V connection must be fused at the source.
- Set it up so the driver doesn't need to look at or touch the screen much while driving. Use Siri and the steering-wheel controls wherever possible.
