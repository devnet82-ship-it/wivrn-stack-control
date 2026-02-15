# VR Stack Control (v0.6.85)

Linux GUI + systemd user service for running a modular VR stack:

**tracking → server → headset app → VR app**

Built to keep OpenXR and OpenVR switching reliable (especially when SteamVR likes to stick).

## Credits
- David Coates — idea, testing, real-world setup notes
- ChatGPT — implementation help + iteration

## Why this exists
VR on Linux can break when OpenXR and OpenVR state gets stuck:
- SteamVR leaves an OpenXR override behind
- OpenVR paths go missing or duplicate
- switching between WiVRn and SteamVR does not snap back

VR Stack Control makes switching predictable:
- WiVRn mode clears the user OpenXR override
- WiVRn mode closes Steam and SteamVR on switch
- SteamVR mode sets the user OpenXR override to SteamVR
- OpenVR paths are repaired and normalized when needed
- One button start and stop with correct ordering

---

## Install VR Stack Control (Release zip)

Download the v0.6.85 zip from GitHub Releases, then:

```bash
cd ~/Downloads
unzip -o vr-stack-control-v0.6.85*.zip -d vr-stack-control-v0.6.85
cd vr-stack-control-v0.6.85/*
chmod +x install.sh uninstall.sh bin/*
./install.sh
systemctl --user daemon-reload
```

## Run

```bash
vr-control --gui
```

## Tray (optional)
- Start tray once: `vr-control tray`
- Autostart tray: `vr-control tray-enable`
- Disable tray: `vr-control tray-disable`

## Doctor (sanity check)

```bash
vr-control doctor
```

---

# Quest 3 on CachyOS
## WiVRn (BETA) + SlimeVR (BETA) + WayVR using VR Stack Control

This is the setup that worked on CachyOS + Quest 3.
Use beta or dev builds for WiVRn and SlimeVR for this stack.

Final stack order (managed by VR Stack Control):
SlimeVR (beta) → WiVRn server → Quest WiVRn app → WayVR → game

## Requirements
- CachyOS (Wayland session)
- Meta Quest 3 (Developer Mode enabled)
- USB cable for one time ADB setup
- PC and headset on the same network

---

## Part 1 — Install PC software (beta and dev builds)

### 1) Install an AUR helper (if you do not have one)
If `yay` is missing:

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### 2) Remove stable packages (if installed)

```bash
sudo pacman -Rns wivrn-dashboard wivrn-server slimevr 2>/dev/null || true
```

### 3) Install WiVRn dev build + SlimeVR beta build

```bash
yay -S --needed wivrn-full-git slimevr-beta-bin
```

### 4) Install the rest

```bash
sudo pacman -S --needed wayvr android-tools github-cli
```

Verify:

```bash
command -v wivrn-server
command -v wayvr
command -v slimevr
```

---

## Part 2 — OpenVR compatibility for SteamVR games (XRizer)

WiVRn will warn if OpenVR compatibility is missing. Install XRizer:

```bash
yay -S --needed xrizer-git xrizer-common-git lib32-xrizer-git
```

Set VR_PATHREG_OVERRIDE in fish (universal):

```fish
set -Ux VR_PATHREG_OVERRIDE $HOME/.local/share/openvr/openvrpaths.vrpath
```

---

## Part 3 — Set system OpenXR runtime to WiVRn (recommended)

VR Stack Control clears the user override when you select WiVRn.
To make WiVRn the default fallback, set the system runtime to WiVRn:

```bash
sudo mkdir -p /etc/openxr/1
sudo ln -sf /usr/share/openxr/1/openxr_wivrn.json /etc/openxr/1/active_runtime.json
```

---

## Part 4 — Quest Developer Mode + ADB

Enable Developer Mode in the Meta Quest phone app:
Devices → Quest 3 → Developer Mode → ON
Reboot the headset.

Enable USB Debugging in the headset:
Settings → System → Developer → USB Debugging

Connect and verify:

```bash
adb kill-server
adb start-server
adb devices
```

Accept the prompt in the headset and tick Always allow, then run again:

```bash
adb devices
```

You must see: `XXXXXXXXXXXX    device`

---

## Part 5 — Install the WiVRn Quest APK (beta) from GitHub Actions

List recent runs:

```bash
gh run list --repo WiVRn/WiVRn --limit 10
```

Pick the newest successful run ID and download the APK artifact (usually named `apk-Release`):

```bash
gh run download RUN_ID_HERE --repo WiVRn/WiVRn --name apk-Release
unzip -o *.zip
ls *.apk
```

Install to Quest:

```bash
adb install -r *.apk
adb shell pm list packages | grep -i wivrn
```

If you use SlimeVR for body tracking, set WiVRn app body tracking to off to avoid conflicts.

---

## Part 6 — Run everything using VR Stack Control

Open the GUI:

```bash
vr-control --gui
```

In the GUI:
- Select **WiVRn (Native / Quest streaming)**
- Enable SlimeVR autostart in your profile if you want tracking every time
- Start the stack

Service commands (optional):

```bash
systemctl --user start vr-stack-control.service
systemctl --user stop vr-stack-control.service
journalctl --user -fu vr-stack-control.service
```

Quick sanity check:

```bash
vr-control doctor
```

Expected in WiVRn mode:
- OpenXR runtime name: Monado
- OpenXR library includes libopenxr_wivrn
- SteamVR processes: not running
