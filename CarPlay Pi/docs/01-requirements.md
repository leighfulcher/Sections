# 1. Requirements Specification

Priority key: **M** = must have (v1), **S** = should have (v1 if possible), **C** = could have (later phases).

## 1.1 Functional requirements

| ID | Pri | Requirement |
|----|-----|-------------|
| F-01 | M | Show Apple CarPlay on a touchscreen in the Elantra, with touch input working. |
| F-02 | M | Play CarPlay audio (music, navigation prompts, Siri) through the car's speakers. |
| F-03 | M | Support phone calls (voice in through a mic, voice out through the speakers), either through CarPlay or the car's factory Bluetooth. |
| F-04 | M | Turn on automatically when the ignition is in ACC/ON, and boot straight into the CarPlay UI with no keyboard or mouse. |
| F-05 | M | Shut down safely on its own when the ignition turns off. |
| F-06 | S | Wireless CarPlay: the iPhone connects automatically without being plugged in. Wired CarPlay works as a fallback. |
| F-07 | S | Accept Siri activation from a touch button. Steering-wheel voice button support is C (see F-12). |
| F-08 | S | Adjust screen brightness, either automatically (light sensor or time of day) or with a quick manual toggle for night driving. |
| F-09 | C | OBD-II live data (coolant temperature, RPM, trouble codes) on a secondary screen, using an ELM327 adapter. |
| F-10 | C | Backup camera that switches the display to the camera when reverse is engaged. |
| F-11 | C | Dashcam recording using a Pi camera module. |
| F-12 | C | Steering-wheel media buttons (next, previous, voice) control CarPlay. |
| F-13 | C | Android Auto support for passengers' phones. Most Carlinkit-type adapters support it. |

## 1.2 Performance requirements

| ID | Pri | Requirement | Target |
|----|-----|-------------|--------|
| P-01 | M | Cold boot (ignition on → CarPlay UI ready, waiting for the phone) | ≤ 30 s |
| P-02 | S | Wireless phone connection after the UI is ready | ≤ 20 s |
| P-03 | M | Touch-to-screen response, perceived | ≤ 150 ms |
| P-04 | M | Video smoothness | ≥ 30 fps at the panel's native resolution |
| P-05 | M | Audio has no audible dropouts or crackle over a 30-minute drive | 0 dropouts |
| P-06 | M | Shutdown time after ignition off | ≤ 20 s, then the system draws close to 0 A |
| P-07 | S | Daylight readability | panel ≥ 400 nits, ideally ≥ 800 |

## 1.3 Environmental and electrical requirements

| ID | Pri | Requirement |
|----|-----|-------------|
| E-01 | M | Runs from the car's 12V system, nominally 11.5–14.8 V. Survives cranking dips (down to about 6 V for under 1 s) without corrupting storage. Rebooting during cranking is acceptable. |
| E-02 | M | Survives load-dump and transient spikes. Use an automotive-rated converter with an input range up to at least 24–36 V. |
| E-03 | M | Draws **less than 1 mA** from the battery with the ignition off. The car may sit for weeks. |
| E-04 | M | Survives storage at cabin temperatures from −20 °C to +80 °C (a parked car in summer sun). Operates from 0 °C to 60 °C, with a fan or heatsink keeping the Pi below its throttle point. |
| E-05 | M | Every power connection is fused at the tap point. No permanent cutting of factory wiring (Option A). |
| E-06 | S | No audible alternator whine or ground-loop hum in the audio path. |
| E-07 | M | Storage survives unexpected power loss (read-only root filesystem, or an SSD plus clean shutdown). |

## 1.4 Mechanical and ergonomic requirements

| ID | Pri | Requirement |
|----|-----|-------------|
| M-01 | M | Screen is reachable from the driver's seat without leaning. Top of the screen is no higher than the top of the dashboard. |
| M-02 | M | Doesn't block airbags, the windshield sight line, hazard switch, HVAC controls, or vents (more than partially). |
| M-03 | M | The mount stays rigid. No visible screen shake over normal road bumps. |
| M-04 | S | The Pi and electronics are hidden: in the glovebox, behind the dash, or under a seat. Only the screen is visible. |
| M-05 | S | The whole install can be removed and the car returned to stock in under an hour. |

## 1.5 Constraints and assumptions

- **CarPlay licensing:** Apple's CarPlay protocol requires MFi authentication hardware. This project uses an off-the-shelf CarPlay USB adapter and doesn't try to implement CarPlay itself.
- **Factory head unit stays** in v1 (Option A). See the [Architecture](02-architecture.md) doc for the full-replacement option.
- The owner has a multimeter, basic hand tools, panel/trim removal tools, and a computer for flashing SD cards. Soldering is helpful but optional.
- Budget target: under $350 for v1.
