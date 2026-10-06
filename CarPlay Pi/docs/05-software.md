# 5. Software Specification

## 5.1 Stack

| Layer | Choice |
|-------|--------|
| OS | **Raspberry Pi OS Lite (64-bit, Bookworm or newer)**. No desktop, which boots faster and uses less RAM. Use the OS version the pi-carplay README currently supports. |
| Display server | **cage** (single-app Wayland kiosk compositor). Alternatively, the pi-carplay installer's own kiosk setup, if it provides one. |
| CarPlay host app | **pi-carplay** (`github.com/f-io/pi-carplay`). Fallback: react-carplay. Install from the project's release (AppImage/.deb) per its README. |
| Audio | PipeWire (Bookworm default), with the USB DAC as the default sink |
| Services | systemd units: kiosk, ignition monitor, optional brightness |
| Storage protection | `overlayroot` (Raspberry Pi OS "Overlay File System" in `raspi-config`) after setup is final |

## 5.2 Base OS setup

1. Flash Raspberry Pi OS Lite 64-bit with **Raspberry Pi Imager**. In the advanced options, set the hostname `carplay`, the user `pi`, SSH enabled, and your home Wi-Fi (for setup only).
2. First boot, then update:
   ```bash
   sudo apt update && sudo apt full-upgrade -y
   sudo rpi-eeprom-update -a   # Pi 4/5 bootloader
   ```
3. `/boot/firmware/config.txt` additions:
   ```ini
   # Pi 5 on a non-PD 5V supply: allow full USB current
   usb_max_current_enable=1
   # Faster boot
   disable_splash=1
   boot_delay=0
   # Touch Display 2 in landscape (if used; adjust per display docs)
   # dtoverlay=vc4-kms-dsi-ili9881-7inch,rotation=90
   ```
4. Turn off services you don't need:
   ```bash
   sudo systemctl disable --now cups avahi-daemon triggerhappy ModemManager 2>/dev/null
   sudo systemctl mask systemd-networkd-wait-online NetworkManager-wait-online
   ```
5. Turn on console autologin for `pi`: `sudo raspi-config` → System → Boot / Auto Login → **Console Autologin**.

## 5.3 CarPlay app

1. Install pi-carplay per its README. Typical steps are installing dependencies, adding the **udev rule** that lets a non-root user access the Carlinkit USB device (vendor `1314`, product `1520`/`1521`; check the README), and installing the app.
2. Settings to configure in the app:
   - Resolution = the display's native resolution (e.g. 1024×600)
   - FPS = 30 (try 60 if stable)
   - DPI ≈ 160 for 7"
   - Audio output = the USB DAC
   - Microphone = the USB mic (if used)
   - Wireless CarPlay = on
3. Pair the iPhone once: plug it into the adapter by USB, or use wireless pairing through Settings → General → CarPlay. After that it reconnects wirelessly on its own.

## 5.4 Boot straight into CarPlay (kiosk)

`/etc/systemd/system/carplay-kiosk.service`:

```ini
[Unit]
Description=CarPlay kiosk
After=systemd-user-sessions.service sound.target
Conflicts=getty@tty1.service

[Service]
User=pi
PAMName=login
TTYPath=/dev/tty1
StandardInput=tty
Environment=XDG_RUNTIME_DIR=/run/user/1000
# Replace the path with the installed pi-carplay binary/AppImage
ExecStart=/usr/bin/cage -s -- /home/pi/pi-carplay/pi-carplay.AppImage --no-sandbox
Restart=always
RestartSec=2

[Install]
WantedBy=graphical.target
```

```bash
sudo apt install -y cage
sudo systemctl set-default graphical.target
sudo systemctl enable carplay-kiosk.service
```

`Restart=always` brings the app back automatically if it crashes or the adapter is reset.

## 5.5 Ignition monitor / safe shutdown

For **Design 2 (DIY opto on GPIO17, relay keep-alive on GPIO27)**. If you use a commercial controller, use its script or match its pin roles.

`/usr/local/bin/ignition-monitor.py`:

```python
#!/usr/bin/env python3
"""Shut the Pi down when ACC has been off for OFF_DELAY seconds."""
import subprocess, time
from gpiozero import Button

ACC_PIN = 17        # opto output: pulled LOW while ACC is on
OFF_DELAY = 5.0     # debounce for engine cranking
# The relay keep-alive (GPIO27) is held high by config.txt, not this script,
# so it stays on until the SoC has actually halted.

acc = Button(ACC_PIN, pull_up=True)          # pressed == ACC on

off_since = None
while True:
    if acc.is_pressed:
        off_since = None
    else:
        off_since = off_since or time.monotonic()
        if time.monotonic() - off_since >= OFF_DELAY:
            subprocess.run(["systemctl", "poweroff"])
            time.sleep(60)   # wait for systemd to stop us; don't exit and restart

    time.sleep(0.25)
```

To make the keep-alive line drop at halt (so the relay releases), add to `config.txt`:
```ini
# Drive GPIO27 high from boot; it goes low when the SoC powers down
gpio=27=op,dh
```
and/or use `dtoverlay=gpio-poweroff,gpiopin=27,active_low=1` (check the overlay docs for the Pi 5 behavior).

`/etc/systemd/system/ignition-monitor.service`:
```ini
[Unit]
Description=Ignition (ACC) monitor
After=multi-user.target

[Service]
ExecStart=/usr/bin/python3 /usr/local/bin/ignition-monitor.py
Restart=always

[Install]
WantedBy=multi-user.target
```

## 5.6 Brightness (optional, F-08)

- **Touch Display 2 / DSI:** write to `/sys/class/backlight/*/brightness`.
- **HDMI panels:** many have no software control. Use the panel's own button, or a panel that supports DDC/CI (`ddcutil`).
- Simple rule: 100 % between sunrise and sunset, 30 % at night. Or read a BH1750 light sensor over I²C and scale linearly. Run it as a small systemd timer or service.

## 5.7 Power-loss hardening (E-07)

After everything works:

1. `sudo raspi-config` → Performance → **Overlay File System → enable** (and make the boot partition read-only). The root filesystem now writes to RAM, and a power cut can't corrupt it.
2. To change settings later: turn the overlay off, reboot, make changes, and turn it back on.
3. If pi-carplay saves settings or pairing data to the home directory, change and save them **before** enabling the overlay, or point that config at a small separate read-write partition.
4. Turn off swap: `sudo dphys-swapfile swapoff && sudo systemctl disable dphys-swapfile`.

## 5.8 Boot-time budget (P-01 ≤ 30 s)

| Stage | Target |
|-------|--------|
| Bootloader + kernel | about 5 s |
| systemd to graphical.target | about 6 s |
| cage + app start | about 4 s |
| Adapter ready | about 5–10 s |

Measure with `systemd-analyze` and `systemd-analyze blame`. An NVMe SSD on the Pi 5 cuts several seconds.

## 5.9 Later software modules

- **OBD-II (F-09):** a Python service using `python-OBD` over Bluetooth RFCOMM, shown as a second full-screen page (e.g. a small web UI in a second cage window, switched by a swipe-edge or button). CarPlay stays the default view.
- **Reverse camera (F-10):** opto-sense the reverse-light 12V on another GPIO and bring an `mpv`/`gst-launch` camera window to the front while it's active.
- **Dashcam (F-11):** `rpicam-vid` with segmented loop recording to a separate **read-write** partition or a USB drive (not the overlay).
- **Remote updates:** keep the configs and scripts in this repo, then deploy with a single `install.sh` over SSH while parked on home Wi-Fi.
