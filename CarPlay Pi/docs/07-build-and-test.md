# 7. Build Plan & Test Plan

## 7.1 Phased plan

Each phase ends with a **gate**: don't start the next phase until the current gate passes.

### Phase 0 — Confirm and order (about 1 evening)
- [ ] Answer the open questions in the [README](../README.md) (radio model, AUX jack, mount location).
- [ ] Measure the dash-top mount area and the glovebox interior.
- [ ] Find the ACC and constant fuses with a multimeter. Note the fuse type and slot.
- [ ] Check the pi-carplay README for the supported adapter model, firmware, and OS version.
- [ ] Order the core BOM.

**Gate 0:** parts on hand. ACC and constant fuse slots identified and written down here:

| Circuit | Fuse slot / label | Rating | Verified V (key OFF / ACC) |
|---------|-------------------|--------|-----------------------------|
| ACC | | | / |
| Constant | | | / |

### Phase 1 — Bench prototype (1–2 evenings)
- [ ] Flash the OS, set up the base system (sections 5.2–5.3).
- [ ] Power from a **wall 5V/5A supply** (official Pi 5 PSU).
- [ ] Connect the display, adapter, DAC, and mic. Pair the iPhone (wired first, then wireless).
- [ ] Set up the kiosk service (5.4). Confirm the Pi boots straight into CarPlay.

**Gate 1:** bench tests T-01 through T-06 pass.

### Phase 2 — Power & shutdown on the bench (1 evening)
- [ ] Build the power chain (Design 1 controller, or Design 2 DIY) using a **12V bench supply** or a spare 12V battery. Use a toggle switch to stand in for ACC.
- [ ] Install the ignition-monitor service (5.5).

**Gate 2:** T-07 through T-10 pass on the bench.

### Phase 3 — In-car temporary install (1 afternoon)
- [ ] Disconnect the battery. Install the fuse taps, ground, and converter/controller.
- [ ] Put the Pi loose in the glovebox. Clip the screen to a vent (location B, temporary).
- [ ] Run the AUX audio with the isolator.

**Gate 3:** in-car tests T-11 through T-16 pass across **at least 1 week of normal driving**.

### Phase 4 — Final install & hardening (1 weekend)
- [ ] Final screen mount (location A), bezel, cable routing and taping.
- [ ] Mount the Pi in its glovebox case. Install the mic on the A-pillar.
- [ ] Enable the overlay filesystem (5.7). Back up the SD card image.

**Gate 4:** all tests pass again, plus the mechanical checks in 6.7.

### Phase 5 — Extras (optional, any order)
- OBD-II gauges (F-09) → reverse camera (F-10) → auto brightness (F-08) → dashcam (F-11) → steering-wheel controls (F-12) → consider Option B.

## 7.2 Test cases

| ID | Phase | Test | Pass criteria | Req |
|----|-------|------|---------------|-----|
| T-01 | 1 | Cold boot to the CarPlay UI | ≤ 30 s, no keyboard needed | F-04, P-01 |
| T-02 | 1 | Wired iPhone connection | CarPlay home screen shows | F-01 |
| T-03 | 1 | Wireless reconnection after reboot | Connects automatically ≤ 20 s after the UI appears | F-06, P-02 |
| T-04 | 1 | Touch accuracy: tap all 4 corners and the center | Each tap hits its target | F-01, P-03 |
| T-05 | 1 | Play music and navigation for 30 minutes | No audio dropouts, video ≥ 30 fps, no thermal throttling | P-04, P-05 |
| T-06 | 1 | Siri via the on-screen button and the mic | Siri hears and responds correctly | F-07 |
| T-07 | 2 | ACC off → shutdown | Clean poweroff ≤ 20 s, then power is cut | F-05, P-06 |
| T-08 | 2 | ACC off for 2 s then back on (crank simulation) | No shutdown starts | E-01 |
| T-09 | 2 | Off-state current | < 1 mA | E-03 |
| T-10 | 2 | Pull power mid-run 20 times (after the overlay is enabled) | Boots normally every time | E-07 |
| T-11 | 3 | Engine start from key off | System comes up and stays up through cranking | E-01 |
| T-12 | 3 | Audio noise at idle and 3000 RPM, radio volume at normal level | No audible whine or hum | E-06 |
| T-13 | 3 | Incoming phone call (A1: factory BT) | Call audio clear both ways, CarPlay resumes after | F-03 |
| T-14 | 3 | Navigation prompt while music plays | Prompt audible, music ducks | F-02 |
| T-15 | 3 | `vcgencmd get_throttled` after a 30-minute drive | `0x0` | E-04 |
| T-16 | 3 | Car parked 7 days | Starts normally, battery voltage drop is no worse than baseline | E-03 |
| T-17 | 4 | Daylight readability, sun behind the car | Map readable at a glance | P-07 |
| T-18 | 4 | Rough-road and highway drive | No visible shake, no rattles | M-03 |
| T-19 | 4 | Full removal dry-run (time it) | Back to stock in < 1 hour | M-05 |

## 7.3 Risks and mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Adapter firmware update breaks pi-carplay compatibility | Med | High | Turn off the adapter's auto-update (if offered). Note the working firmware version. Keep react-carplay as a fallback. |
| SD card corruption | Med | Med | Endurance card, overlay FS, clean shutdown, keep a backup image. |
| Screen too dim in daylight | Med | Med | Buy a high-brightness panel. Add a sun hood to the bezel. |
| Summer heat damages the screen | Low–Med | Med | Sunshade, quick-release mount, check the storage temperature rating. |
| Ground-loop noise | High | Low | Isolator, single ground point. |
| Battery drain | Low | High | Full power cut-off design. T-09 and T-16. |
| Airbag interference | Low | Very high | Mount location rules in 6.2. Never mount on the airbag door. |
