# eIIIa Audio Device Switcher for REAPER

A native REAPER extension for fast, reliable switching between CoreAudio interfaces on macOS — directly from REAPER Actions and keyboard shortcuts.

> **Current release documented here:** Native 0.3.1  
> **Platform:** macOS / CoreAudio  
> **Runtime dependencies:** none beyond REAPER itself  
> **No ReaPack, ReaImGui, Lua, SWS, Accessibility automation, mouse macros, or macOS system-audio switching.**

## Why this exists

REAPER can switch audio devices from **Preferences → Audio → Device**, but doing that repeatedly is slow when you move between an audio interface, headphones, display audio, virtual devices, and other CoreAudio endpoints.

This extension turns that workflow into REAPER Actions:

- open a compact native device menu;
- see the CoreAudio interfaces currently available;
- see the interface name and input/output channel count;
- switch with one click;
- assign REAPER keyboard shortcuts to persistent device slots 1–9.

The extension changes **REAPER's own audio configuration only**. It does not change the macOS system default input or output device.

## Quick start

### Recommended: build and install automatically

1. Download `eIIIa_REAPER_Audio_Device_Switcher_Native_0.3.1_Documented.zip` from this repository and extract it.
2. Quit REAPER.
3. Double-click **`BUILD_AND_INSTALL.command`** inside the extracted folder.
4. If macOS asks for developer tools, install Apple's Command Line Tools and run the script again.
5. Restart REAPER.
6. Open **Actions → Show action list...**.
7. Search for **`eIIIa Audio Device`**.
8. Run **`eIIIa Audio Device: Show switcher`**.

The build script downloads the official REAPER SDK and WDL, runs the tests, builds the native extension, and installs it to the standard REAPER `UserPlugins` folder.

For portable/custom REAPER resource paths, see [Installation](docs/INSTALLATION.md).

## Actions

The extension registers these actions in REAPER:

- `eIIIa Audio Device: Show switcher`
- `eIIIa Audio Device: Switch to slot 1`
- `eIIIa Audio Device: Switch to slot 2`
- `eIIIa Audio Device: Switch to slot 3`
- `eIIIa Audio Device: Switch to slot 4`
- `eIIIa Audio Device: Switch to slot 5`
- `eIIIa Audio Device: Switch to slot 6`
- `eIIIa Audio Device: Switch to slot 7`
- `eIIIa Audio Device: Switch to slot 8`
- `eIIIa Audio Device: Switch to slot 9`

Any of them can be assigned to a keyboard shortcut, toolbar button, MIDI/OSC action, custom action, or other REAPER action trigger.

## The switcher menu

The popup displays:

- slot number;
- CoreAudio device name;
- input/output channel count;
- a check mark when the device is currently used by REAPER;
- **Refresh interfaces**;
- **Rebuild slots 1–9**.

The top of the menu also shows REAPER's current input and output identifiers.

## Slots 1–9

Slots make one-key switching possible.

- Slots are stored persistently by **CoreAudio device UID**, not by temporary numeric device ID.
- Existing assignments are preserved across REAPER restarts.
- A disconnected device keeps its slot; triggering that slot reports that the interface is not connected.
- Empty slots are filled from currently connected interfaces.
- **Rebuild slots 1–9** clears the slot map and rebuilds it from the currently connected interfaces.

The current implementation automatically maintains the slot map; it does not provide a separate manual drag/drop slot editor.

## How switching works

REAPER's public extension API exposes functions for reading the currently opened audio device, but it does not expose a public `SetAudioDevice()` / `ApplyAudioDevicePreferences()` API.

Earlier approaches that attempted to change REAPER configuration variables directly were not reliable on macOS. The working design intentionally routes the change through **REAPER's own Audio Device Preferences handler**:

1. CoreAudio is queried for available device names, UIDs, and channel counts.
2. REAPER's current input/output identifiers are read with `GetAudioDeviceInfo()`.
3. The extension invokes REAPER's native **Audio device configuration** action.
4. It identifies the exact Audio Device popup controls by their current value and available target value.
5. It selects the target through the native control action and invokes REAPER's own **OK** handler.
6. The result is verified again with `GetAudioDeviceInfo(IDENT_IN/IDENT_OUT)`.
7. If REAPER does not open the requested device, the extension reports failure instead of pretending the switch succeeded.

The bridge deliberately refuses ambiguous UI matches rather than guessing.

For the detailed implementation, see [Architecture](docs/ARCHITECTURE.md).

## Important known limitation: brief Preferences flash

REAPER must create its Audio Device Preferences window before the extension can access REAPER's native controls. The extension tries to hide that window immediately and repeatedly while the switch is being applied, but macOS may still draw a brief frame of the Preferences window.

This is cosmetic. It is a consequence of using REAPER's reliable native apply path rather than fragile mouse/Accessibility automation. Eliminating the flash completely would require invasive interception of REAPER/AppKit window presentation, which is intentionally not used.

## Safety and scope

The extension is intentionally conservative:

- **does not change the macOS system default audio device**;
- does not redirect system audio;
- does not use Accessibility permissions;
- does not use `osascript` or System Events;
- does not move or click the mouse;
- does not depend on screen coordinates;
- does not open UI or enumerate CoreAudio while REAPER is loading the extension;
- verifies the actual device after every requested switch;
- refuses to guess when the REAPER Preferences controls cannot be identified safely.

## Requirements

For normal runtime:

- macOS;
- REAPER;
- one or more CoreAudio devices.

For building from source:

- Apple Command Line Tools (`xcode-select --install`);
- `git` (included with the Command Line Tools);
- internet access during the first build so the official REAPER SDK and WDL can be downloaded.

No ReaPack, SWS, ReaImGui, Lua environment, Python environment, Homebrew package, or third-party runtime framework is required.

## Installation, troubleshooting and development

- [Installation and update guide](docs/INSTALLATION.md)
- [Using the switcher](docs/USAGE.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Technical architecture](docs/ARCHITECTURE.md)
- [Building from source](docs/BUILDING.md)

## Diagnostic files

Runtime log:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```

Build logs:

```text
build/BUILD.log
build/BUILD_AND_INSTALL.log
```

To locate the REAPER resource path, use **Options → Show REAPER resource path in Finder**.

## Uninstall

1. Quit REAPER.
2. Open the REAPER resource path and then `UserPlugins`.
3. Delete:

```text
reaper_eIIIa_audio_device_switcher.dylib
```

4. Start REAPER again.

The small persistent slot assignments stored in REAPER ExtState are harmless if left behind.

## Project structure

The downloadable source package contains:

```text
src/plugin_main.cpp           REAPER extension entry point and Actions
src/plugin_state.cpp          switch orchestration, slots and verification
src/coreaudio_devices.mm      CoreAudio enumeration and stable UIDs
src/device_window.mm          native transient switcher menu
src/reaper_prefs_bridge.mm    bridge to REAPER's native Audio Device controls
src/prefs_popup_logic.cpp     safe popup matching logic
src/slot_assignment.cpp       persistent slot assignment logic
BUILD_AND_INSTALL.command     one-click build + standard REAPER install
BUILD.command                 build-only wrapper with persistent log
build_macos.command           tests, compilation and linking
```

## Status and platform roadmap

Native 0.3.1 is the working macOS/CoreAudio implementation documented by this repository. A cross-platform architecture for Windows and Linux is being explored separately, but Windows/Linux support should not be assumed for this release.

## Author

**eIIIa**

This project is independent software for REAPER and is not affiliated with Cockos, Inc.
