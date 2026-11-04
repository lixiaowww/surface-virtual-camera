# Surface Virtual Camera

One-click installer that exposes a Surface (IPU3 / libcamera) front camera as a normal V4L2 webcam for Zoom and other Electron apps.

## Install

```bash
chmod +x install.sh
./install.sh
```

This installs:

- `v4l2loopback` virtual device named **Surface Front Camera**
- a systemd relay service (`surface-camera.service`)
- a desktop launcher **Zoom (Camera)** that starts the camera only while Zoom is open

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

1. Open **Zoom (Camera)** from the desktop (or run `/usr/local/bin/zoom-with-camera.sh`)
2. In Zoom → **Settings → Video**, select **Surface Front Camera**
3. Quitting Zoom stops the camera automatically

If `/dev/video0` is not the virtual device yet, reboot once so module load order applies.

## Manual control

```bash
sudo systemctl start surface-camera
sudo systemctl stop surface-camera
```

## Uninstall

```bash
./install.sh --uninstall
```

## Notes

- Raw `ipu3-*` nodes are hidden from apps so Zoom only sees the virtual webcam.
- Cheese / other libcamera apps may conflict while the relay is running.
- Camera ACPI id is auto-detected via `cam -l` into `/etc/default/surface-camera`.
