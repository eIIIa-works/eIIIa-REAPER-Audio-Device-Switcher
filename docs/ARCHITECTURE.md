# Architecture

## Design principle

REAPER's own Audio Device Preferences page is the authoritative application mechanism. The extension does not attempt to reproduce backend-specific driver opening logic itself.

## Transport guard

Audio-device operations are gated by a shared transport guard before any platform-specific Preferences work begins. The guard uses REAPER's own APIs:

```text
GetPlayState() -> if playing/recording -> OnStopButton() -> verify transport stopped
```

The guard is called both before `Show switcher` discovery and before direct profile/device application. This keeps the behavior identical across macOS, Windows and Linux and prevents a backend-specific Preferences switch from starting while REAPER is actively playing or recording.

Paused state is not treated as active playback. If Stop is unavailable or REAPER remains active after Stop, the audio-device operation is aborted.

## Common core

Shared C++ code owns:

- REAPER Action registration;
- slots 1–9;
- compact popup rows;
- stable profile identity helpers;
- profile synthesis from REAPER selector snapshots;
- diagnostics;
- transport-safe Stop-before-switch policy;
- startup safety.

## macOS adapter

The tested macOS implementation combines:

- CoreAudio device enumeration for names, UIDs and channel counts;
- Cocoa traversal of REAPER's native Preferences controls;
- native `NSPopUpButton` selection actions;
- REAPER's own Preferences `OK` handler;
- `GetAudioDeviceInfo(IDENT_IN/IDENT_OUT)` verification.

The macOS path deliberately preserves the mechanism proven in the 0.3.x development series.

## Windows adapter

The Windows path uses Win32 HWND/combo-box messages inside REAPER's Audio Device Preferences page.

The source of truth for backend names is REAPER's **Audio System** combo. The controller iterates the choices exposed by REAPER and inspects the backend-specific selectors. It does not call a Windows system-default audio setter.

Selection uses `CB_SETCURSEL` plus REAPER's native selection-change notification and finally REAPER's `OK` path.

## Linux adapter

REAPER for Linux uses Cockos WDL/SWELL, which exposes Win32-like control APIs. The Linux path reuses the same semantic selector/profile logic through SWELL and compiles with `SWELL_PROVIDED_BY_APP`.

The extension does not link directly against ALSA/JACK/PipeWire as its discovery authority; the intent is to discover what the installed REAPER build exposes.

## Profile synthesis

A portable profile contains:

```cpp
struct AudioProfile {
  std::string stable_id;
  std::string display_name;
  std::string audio_system;
  std::vector<SelectorValue> selectors;
  std::string platform_uid;
  bool active;
};
```

Driver selectors generate one profile per driver. Generic device selectors generate one profile per device. Separate input/output selectors are merged by matching device title rather than creating an uncontrolled input × output Cartesian product.

## Safety

The entrypoint does not enumerate devices or open UI while REAPER is loading the extension. Device/profile work begins only after an Action is invoked.

Ambiguous control matching is treated as failure, not permission to guess.

## Known UI compromise

The Preferences dialog is hidden as soon as possible. On macOS a brief compositor-visible flash can remain because REAPER presents the window before the extension can reliably identify it. The project intentionally avoids global method-swizzling or comparable invasive interception.
