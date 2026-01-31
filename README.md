# VR Stack Control v0.6.21

This release replaces your old `vr-control --gui` YAD UI with a GTK GUI.

## Install

```bash
pkill -9 -f "vr-control --gui" 2>/dev/null || true
pkill -9 -f yad 2>/dev/null || true

cd ~/Downloads
unzip -o vr-stack-control-v0.6.21.zip
cd vr-stack-control-v0.6.21
bash install.sh
```

## Run

```bash
vr-control --gui
```

## What’s new

- `vr-control --gui` now opens the GTK GUI
- Profiles page: create / rename / delete / edit profiles
- Removed the old “Actions” tab (Start/Stop + settings cover most use-cases)
- Apps & Settings page includes inline “why this matters” help text

Note: Tray integration was removed in v0.6.21. Closing the window exits normally.


WiVRn + WayVR + SlimeVR on CachyOS (Quest 3)
FIRST-TIME SETUP — COMPLETE IDIOT-PROOF GUIDE

Follow every step in order.
Do not skip steps.
Do not start apps manually unless told to.

WHAT YOU WILL END WITH (IMPORTANT)

When finished:

Quest 3 connects wirelessly to PC

WayVR runs without errors

SlimeVR body tracking works

One desktop button starts everything

No duplicate launches

Correct OpenXR runtime every time

Final stack:

SlimeVR → SolarXR → WiVRn OpenXR → WiVRn Server → Quest WiVRn APK → WayVR

REQUIREMENTS (READ FIRST)

You need:

PC running CachyOS (Wayland session)

Quest 3 headset

USB cable (one-time setup)

PC + Quest on same Wi-Fi

Internet access

PART 1 — INSTALL PC SOFTWARE

Open a terminal and run exactly:

yay -S wivrn-full-git
sudo pacman -S wayvr slimevr android-tools github-cli


What this installs:

WiVRn server (beta / working build)

WayVR compositor

SlimeVR server

ADB (for Quest)

GitHub CLI (for APK download)

VERIFY COMMANDS EXIST (DO NOT SKIP)
command -v wayvr
command -v wivrn-server
command -v slimevr


If any command is missing, stop here and fix before continuing.

PART 2 — QUEST DEVELOPER MODE + ADB
2.1 Enable Developer Mode (PHONE)

On your phone:

Meta Quest app

Devices → Quest 3

Developer Mode → ON

Reboot headset

2.2 Enable USB Debugging (HEADSET)

Inside headset:

Settings → System → Developer

Enable USB Debugging

2.3 Connect Quest to PC

Plug Quest into PC via USB.

Run:

adb kill-server
adb start-server
adb devices


Inside headset:

Accept USB debugging

Tick Always allow

Verify again:

adb devices


You must see:

XXXXXXXXXXXX    device


If not → stop and fix before continuing.

PART 3 — DOWNLOAD & INSTALL QUEST APK (WORKING METHOD)
3.1 Login to GitHub CLI
gh auth login
gh auth status


You must see:

Logged in to github.com

3.2 Download WiVRn Quest APK (Release)

We use the official GitHub Actions build.

Example run ID you used:

21321590049


Download:

gh run download 21321590049 --repo WiVRn/WiVRn --name apk-Release


Extract:

unzip -o *.zip
ls *.apk


You must see one APK file.

3.3 Install APK to Quest

Inside headset:

Settings → Apps → WiVRn → Uninstall

Reboot headset

Then install:

adb install -r *.apk


Expected output:

Success


Verify:

adb shell pm list packages | grep -i wivrn


Expected:

package:org.wivrn.client

3.4 REQUIRED QUEST SETTING (SLIMEVR USERS)

Inside Quest WiVRn app:

❌ Disable Enable body tracking

✅ Let SlimeVR / SolarXR provide tracking

(This avoids known conflicts.)

PART 4 — FIX OPENXR RUNTIME (CRITICAL STEP)

This is the step that made WayVR work.

Run:

mkdir -p ~/.config/openxr/1
ln -sf /usr/share/openxr/1/openxr_wivrn.json ~/.config/openxr/1/active_runtime.json


Verify:

readlink -f ~/.config/openxr/1/active_runtime.json


It must output:

/usr/share/openxr/1/openxr_wivrn.json


If it does not → STOP and fix before continuing.
