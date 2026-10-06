# 6. Mechanical & Mounting Specification

## 6.1 Dash layout (2014 Elantra sedan)

The MD Elantra's center stack is a tall, sculpted "hourglass". The factory radio sits under the two center vents, with HVAC controls below it and the hazard switch between the vents. The dash top is curved and has a soft finish.

## 6.2 Candidate screen locations

| Location | Pros | Cons | Verdict |
|----------|------|------|---------|
| **A. Dash-top, centered above the center vents** (adhesive or screw base + ball mount) | Close to the driver's line of sight, so navigation is easy to glance at. Doesn't block vents. | Visible from outside (theft attraction). Glare. Curved surface needs a good base. | **Recommended** |
| B. Clamped to the center vent slats | No adhesive. Easy to remove. | Blocks a vent. Wobbles. Hot or cold air hits the screen. | OK for prototyping |
| C. Over the factory radio face (custom bezel) | Cleanest look | Covers radio controls (needed in Option A) | Option B only |
| D. Cupholder / console pedestal mount | No dash contact | Too low, requires looking far down | Not recommended |

### Location A requirements

- Measure **the depth of flat area** behind the windshield base above the vents, and the **screen's height** with its mount. The top edge of the screen must stay **below the top of the dash cowl, as seen from the driver's eye point** (M-01). That's typically ≤ 12–13 cm from the mount surface.
- **Avoid the passenger airbag area.** In the MD Elantra it's on the passenger side of the dash top. Keep the mount on the **driver's side of center**, clear of the airbag seam.
- Use a **low-profile 17 mm ball mount** with a flexible-adhesive or screw-down base. For the curved dash, a 3D-printed adapter shaped to the dash contour (printed in **PETG or ASA**, not PLA) gives a solid fit.

## 6.3 Electronics placement

| Item | Location | Notes |
|------|----------|-------|
| Raspberry Pi + CarPlay adapter | **Glovebox** (rear wall, vented case) or behind the lower center console side panel | The glovebox makes it easy to reach for SD swaps and debugging. Run cables through a small grommet hole in the back of the glovebox liner, or around its edge. |
| Buck converter / power controller | Under the dash near the fuse panel, zip-tied to an existing harness | Keep 12V runs short, with the fuse at the tap. |
| USB DAC + isolator | Center console bin, near the AUX jack | |
| Microphone | Driver's side A-pillar / headliner edge near the visor | Route the cable behind the A-pillar trim. **Don't route it across the curtain airbag**; tuck it along the existing headliner edge. |

## 6.4 Cable routing

```
Fuse panel (driver kick) ──► under the steering column ──► behind the center stack ──► glovebox
Glovebox ──► behind the dash / up the right side of the center stack ──► dash-top screen
           (exit through the gap between the dash top and the windshield-side trim, or the vent bezel edge)
Glovebox ──► along the console side under the carpet edge ──► center bin (AUX)
```

- Wrap all runs in **Tesa cloth tape or split loom**. Secure them every 15–20 cm.
- Keep clear of the **steering column, pedals, and HVAC blend-door actuators**.
- Leave service loops (about 10 cm) at each end.

## 6.5 Thermal design (E-04)

- A parked car's dash can reach **80–90 °C** in the sun. The dash top is the hottest spot in the car.
- The **screen is the most heat-exposed part.** Check its storage temperature rating (look for at least 80 °C). Use a sunshade when parked, or make the screen quick-release so it can come off and live in the glovebox.
- The **glovebox runs cooler** than the dash top. Use the Pi 5 Active Cooler and a vented case. Target an SoC temperature **under 75 °C** while driving: `vcgencmd measure_temp`.
- The microSD and SSD are at risk above 85 °C. Industrial-temperature SD cards are worth it in hot climates.

## 6.6 Enclosure / bezel (optional 3D print)

- **Screen bezel:** wraps the 7" panel and hides the PCB and connectors. Has a rear boss for the 17 mm ball, a strain-relief clip for the cables, and a sun-hood lip of about 10 mm to cut glare.
- **Material:** ASA or PETG (PLA warps in a hot car).
- **Pi case:** vented top over the cooler, mounting tabs for the glovebox wall, and a cutout for the adapter.

## 6.7 Mechanical acceptance checks

- [ ] From the driver's normal seated position, the screen doesn't block any part of the road view.
- [ ] Not in or over the passenger airbag door.
- [ ] No shake visible at the screen corners over rough pavement.
- [ ] Hazard switch, vents, and HVAC controls fully usable.
- [ ] Glovebox still closes and latches.
- [ ] No new rattles at highway speed.
