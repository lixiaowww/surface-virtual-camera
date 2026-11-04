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

### Options

```bash
./install.sh --always-on   # enable relay at boot (camera LED stays on)
./install.sh --uninstall   # remove everything this script installed
```

## Requirements

- Ubuntu / Linux Mint / Debian
- Working libcamera stack ([linux-surface](https://github.com/linux-surface/linux-surface) kernel recommended)
- `cam -l` should list your front camera
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

- Raw `ipu3-*` nodes are hidden from apps so only the virtual webcam is visible.
- Cheese / other libcamera apps may conflict while the relay is running.
- Camera ACPI id is auto-detected via `cam -l` into `/etc/default/surface-camera`.
- Electron/Chromium apps only list the device while the relay is running (`exclusive_caps`).
