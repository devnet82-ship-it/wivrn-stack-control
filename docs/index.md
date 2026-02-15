---
layout: default
title: WiVRn Stack Control
---

# WiVRn Stack Control

A lightweight Linux GUI for managing a modular VR stack (tracking → server → VR app) with reliable OpenXR/OpenVR switching.

**Made by:** David Coates (idea/testing) + ChatGPT (implementation help)  
**GitHub:** https://github.com/devnet82-ship-it

## Why this exists
VR on Linux can break when OpenXR/OpenVR state gets “stuck” (SteamVR overrides, missing OpenVR paths, duplicates, etc.).
This app keeps it sane by:
- selecting the correct OpenXR runtime for WiVRn vs SteamVR
- repairing OpenVR paths when needed
- starting/stopping everything in the right order
