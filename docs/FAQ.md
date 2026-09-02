# FAQ

## Is this macOS-only?

No. Universal 0.4.0 contains build targets and native REAPER-control paths for macOS, Windows, and Linux.

macOS is currently the **tested** platform. Windows and Linux are **implemented but not yet field-tested** in enough real REAPER installations to claim the same confidence level.

## Why are there three binaries if it is universal?

Because it is native C++ code. The source is shared, but each OS loads its own binary format and ABI:

- macOS `.dylib`
- Windows `.dll`
- Linux `.so`

## Can I compile the Windows DLL on my Mac?

Cross-compilation is possible in principle, but it is not the recommended release workflow here. The REAPER Windows SDK expects a MSVC-compatible C++ ABI, and a binary still needs to be tested inside Windows REAPER. Build Windows releases with MSVC on Windows.

## Does it change my system-wide audio device?

No. It controls REAPER's own Audio Device configuration.

## Does it require ReaPack, SWS, Lua, ReaImGui or Python?

No.

## Why does REAPER Preferences briefly flash on macOS?

The only reliable application path found is REAPER's own Audio Device Preferences handler. REAPER creates the window before the extension gets reliable access to its controls. The extension hides it immediately, but a compositor frame may already have been drawn. Avoiding that completely would require invasive global AppKit interception, which this project deliberately does not do.

## Why not edit `reaper.ini` and restart audio?

That was tested. `Audio_Quit()` / `Audio_Init()` reopens the configuration REAPER already has in memory rather than reliably applying a newly written device preference. The working approach is to invoke REAPER's own Preferences handler.

## Why not call a `SetAudioDevice()` function?

REAPER's public extension API exposes `GetAudioDeviceInfo()` but no public cross-platform audio-device setter/apply function.

## What happens if device detection is ambiguous?

The extension refuses to guess and writes diagnostics to the log.
