# eIIIa Audio Device Switcher for REAPER

A native REAPER extension for switching REAPER audio interfaces from Actions, toolbar buttons, MIDI/OSC commands, or keyboard shortcuts — without changing the operating system's global audio device.

**Universal 0.4.0** is designed for **macOS, Windows, and Linux** from one source tree.

> **Test status**
>
> - **macOS / CoreAudio:** tested in real REAPER and confirmed working.
> - **Windows:** Win32 implementation and build target are included; real-world REAPER testing is still needed.
> - **Linux:** REAPER/SWELL implementation and build target are included; real-world REAPER testing is still needed.
>
> If you test Windows or Linux, please report the REAPER version, audio backend, device, and result.

## What it solves

REAPER's normal way to change audio hardware is **Preferences → Audio → Device**. That is fine occasionally, but slow if you switch repeatedly between an interface, headphones, monitor/display audio, virtual devices, ASIO drivers, WASAPI endpoints, JACK/ALSA configurations, and similar setups.

This extension turns audio-device switching into normal REAPER Actions.

## Interface

The popup is intentionally minimal. It contains only:

```text
1  Device / profile
2  Device / profile
3  Device / profile
...
9  Device / profile

Refresh interfaces
Rebuild slots 1–9
```

There is no redundant `REAPER Input / Output` line and no bottom status sentence. The menu width is therefore driven mainly by the actual device/profile names.

The active configuration has a check mark.

## Actions

- `eIIIa Audio Device: Show switcher`
- `eIIIa Audio Device: Switch to slot 1`
- `eIIIa Audio Device: Switch to slot 2`
- ...
- `eIIIa Audio Device: Switch to slot 9`

Assign these like any other REAPER Action: keyboard shortcuts, toolbar buttons, MIDI/OSC, custom actions, etc.

## Why it is different from changing the system default

The extension changes **REAPER's own audio configuration only**.

It does **not**:

- change the macOS default input/output device;
- call Windows `SetDefaultAudioEndpoint` or equivalent system-wide switching;
- change the Linux desktop's global PipeWire/PulseAudio default;
- use Accessibility/UI Automation;
- use mouse coordinates or recorded clicks;
- use AppleScript/System Events, AutoHotkey, xdotool, or similar automation;
- require ReaPack, ReaImGui, Lua, Python, or SWS.

## How switching works

REAPER exposes read access to the currently opened audio device through `GetAudioDeviceInfo()`, but does not expose a public cross-platform `SetAudioDevice()` API.

The reliable solution is to let **REAPER itself apply the configuration**:

1. the extension opens REAPER's own **Audio device configuration** page internally (command `40099`);
2. it finds REAPER's native device/system selectors;
3. it selects the target using the control's native selection-change mechanism;
4. it invokes REAPER's own **OK** handler;
5. REAPER performs backend-specific validation, persistence, audio-engine restart, and device opening;
6. the extension checks the resulting audio state instead of assuming success.

No physical mouse event is simulated.

### macOS

CoreAudio is used for fast device enumeration and stable UIDs. Applying the change still goes through REAPER's own Preferences handler.

### Windows

The extension does not hard-code one backend. It inspects the **Audio System** selector that REAPER actually exposes and is designed to enumerate the available variants (for example ASIO, WASAPI, DirectSound, WaveOut, or other systems present in that REAPER build). Backend-specific driver/device selectors are then treated as REAPER audio profiles.

### Linux

Linux is a separate target from macOS. The implementation uses REAPER/WDL **SWELL** controls and follows the same Preferences-driven strategy. It is designed to consume the audio systems exposed by the user's REAPER build rather than depending directly on libasound/JACK/PipeWire libraries.

## Slots 1–9

Slots provide instant switching.

On macOS the proven path persists devices by stable CoreAudio UID. On Windows/Linux the universal path uses a stable profile identity derived from the REAPER audio system plus selector values.

If a saved slot is unavailable, the extension refuses to guess.

## Installation

See [Installation](docs/INSTALLATION.md) for step-by-step instructions on all platforms.

The short version:

### macOS

Double-click:

```text
BUILD_AND_INSTALL.command
```

Output:

```text
build/reaper_eIIIa_audio_device_switcher.dylib
```

Installed to the standard REAPER resource path:

```text
~/Library/Application Support/REAPER/UserPlugins/
```

### Windows

Open a **x64 Native Tools Command Prompt for Visual Studio** and run:

```text
BUILD_AND_INSTALL.bat
```

Output:

```text
build\reaper_eIIIa_audio_device_switcher.dll
```

Default install location:

```text
%APPDATA%\REAPER\UserPlugins\
```

### Linux

Run:

```bash
chmod +x BUILD_AND_INSTALL.sh
./BUILD_AND_INSTALL.sh
```

Output:

```text
build/reaper_eIIIa_audio_device_switcher.so
```

Default install location:

```text
${XDG_CONFIG_HOME:-~/.config}/REAPER/UserPlugins/
```

## Do I build separately for each OS?

Yes. It is **one source codebase**, but a native REAPER extension must be compiled for the operating system and CPU ABI on which it will run:

- macOS → `.dylib`
- Windows → `.dll`
- Linux → `.so`

A Mac build cannot simply be copied to Windows, and a Windows DLL cannot run on Linux. Cross-compilation is theoretically possible, but it does not remove the need to test the extension inside REAPER on the target OS. The supplied build scripts intentionally build natively on each platform.

## Known limitation: brief Preferences flash

On macOS, REAPER may briefly show its Audio Device Preferences window while the extension is switching. The extension hides the dialog as soon as it can identify it, but REAPER creates/presents that native window before the extension can reliably manipulate its controls, so a short flash can still occur.

This is cosmetic. The project deliberately avoids invasive AppKit swizzling or global window-hook tricks just to eliminate a frame of UI.

Windows/Linux also hide the internal Preferences window where the platform allows it, but those paths still need real-world testing.

## Diagnostics

Runtime log:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```

On macOS, use **Options → Show REAPER resource path in Finder** to locate it. Equivalent resource-path locations are documented for Windows/Linux in the installation guide.

## Documentation

- [Installation](docs/INSTALLATION.md)
- [Usage](docs/USAGE.md)
- [FAQ](docs/FAQ.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Building from source](docs/BUILDING.md)
- [Testing Windows and Linux](docs/TESTING.md)

## Development status

Universal 0.4.0 intentionally distinguishes **implemented** from **field-tested**:

- macOS is the reference implementation and has been tested in real REAPER;
- Windows and Linux code paths, native menu implementations, profile model, platform build scripts, and compile/policy smoke tests are included;
- Windows/Linux still need confirmation on actual REAPER installations and actual audio hardware/backends.

That is why feedback from Windows/Linux users is particularly useful.

## Author

**eIIIa**

Independent software for REAPER. Not affiliated with Cockos, Inc.
