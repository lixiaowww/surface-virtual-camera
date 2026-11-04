# Surface Virtual Camera

One-click installer that exposes a Surface (IPU3 / libcamera) front camera as a normal V4L2 webcam for Zoom, Teams, WeChat, WhatsApp (ZapZap), and other Electron apps.

## Install

```bash
chmod +x install.sh
./install.sh
```

This installs:

- `v4l2loopback` virtual device named **Surface Front Camera**
- a systemd relay service (`surface-camera.service`)
- on-demand desktop launchers that start the camera only while the app is open
- `update-initramfs -u` so `softdep` / module options apply on the next boot

### Options

```bash
./install.sh --always-on     # enable relay at boot (camera LED stays on)
./install.sh --no-hide-raw   # do not lock down ipu3-* nodes (keep user libcamera/PipeWire)
./install.sh --uninstall     # remove everything this script installed
```

## Requirements

- Ubuntu / Linux Mint / Debian
- Working libcamera stack ([linux-surface](https://github.com/linux-surface/linux-surface) kernel recommended)
- `cam -l` should list your front camera **before** install (or when using `--no-hide-raw`)
- sudo access (installs DKMS module + system files)

## After install

Open Zoom / Teams / WeChat / WhatsApp from the menu or Desktop **（摄像头）** launchers.
Stock Flatpak icons are also wrapped so they start the camera automatically.

Then in each app’s video settings, select **Surface Front Camera**.

Quitting the last wrapped app stops the camera automatically.

If `/dev/video0` is not the virtual device yet, reboot once so module load order applies.

## Manual control

```bash
sudo systemctl start surface-camera
sudo systemctl stop surface-camera

# Or wrap any command / Flatpak:
app-with-camera.sh flatpak com.example.App
app-with-camera.sh run firefox
```

## Uninstall

```bash
./install.sh --uninstall
```

## Notes

### Raw `ipu3-*` node permissions (default)

By default the installer writes `99-hide-ipu3-v4l.rules` which sets `ipu3-cio2*` / `ipu3-imgu*` to `MODE=0600`, `GROUP=root`, and strips `uaccess`. The root `surface-camera` relay can still open them via libcamera; **unprivileged** clients cannot:

- `cam -l` (without sudo) → `Permission denied` / empty camera list
- WirePlumber `monitor.libcamera` (user session)
- Browsers using the PipeWire Camera portal against the raw sensors

This is intentional for Zoom/Electron: those apps otherwise enumerate unusable raw V4L2 nodes. Prefer the wrapped apps + **Surface Front Camera**.

If you want native libcamera / PipeWire camera access when the relay is idle, reinstall with:

```bash
./install.sh --no-hide-raw
```

(On IPU6 machines that fill `/dev/video0`–`63`, a per-process `LD_PRELOAD` remapper such as [surface-zoom-bridge](https://github.com/luqqas96/surface-zoom-bridge) may be a better fit than hiding nodes globally.)

### Other

- Cheese / other libcamera apps may conflict while the relay is running.
- Camera ACPI id is auto-detected via `cam -l` into `/etc/default/surface-camera` (detection runs as root during install).
- Electron/Chromium apps only list the loopback while a producer is active (`exclusive_caps`).
- `softdep ipu3_cio2 pre: v4l2loopback` only affects early boot if `/etc/modprobe.d` is packed into the initramfs; install/uninstall run `update-initramfs -u` for that reason.
