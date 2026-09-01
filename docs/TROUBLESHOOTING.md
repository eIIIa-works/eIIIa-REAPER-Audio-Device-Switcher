# Troubleshooting

## The eIIIa actions do not appear in REAPER

Check the installation location first.

1. In REAPER choose **Options → Show REAPER resource path in Finder**.
2. Quit REAPER.
3. Confirm this file exists:

```text
<UserPlugins>/reaper_eIIIa_audio_device_switcher.dylib
```

4. Restart REAPER.

Do not use **Actions → Load ReaScript**. This is a native REAPER extension, not a Lua/Python/EEL script.

If you use more than one REAPER installation, make sure the extension is in the resource path belonging to the REAPER instance you are actually launching.

## `BUILD_AND_INSTALL.command` fails immediately

Open:

```text
build/BUILD_AND_INSTALL.log
```

and also:

```text
build/BUILD.log
```

Common first-build issue: Apple's Command Line Tools are missing.

Install them with:

```bash
xcode-select --install
```

Then run the installer again.

## The build says the REAPER SDK or WDL is missing

Normally `build_macos.command` downloads both automatically.

Make sure:

- `git` works;
- the Mac has internet access;
- the project directory is writable;
- `_deps/reaper-sdk` and `_deps/wdl-source` are not damaged partial checkouts.

The expected header after setup is:

```text
_deps/reaper-sdk/WDL/swell/swell.h
```

## No CoreAudio interfaces are shown

The extension lists live CoreAudio devices that have at least one input or output channel and a valid name/UID.

Check whether the device is visible to macOS and to REAPER itself. Then use **Refresh interfaces**.

If REAPER can see the interface but the switcher cannot, inspect the runtime log and report the macOS/REAPER versions plus the device name.

## A slot says the interface is not connected

This is intentional. Slots are persistent by CoreAudio UID, so unplugging a device does not allow another device to silently steal that slot number.

Reconnect the device, or use **Rebuild slots 1–9** if you intentionally want new numbering.

## I select a device but REAPER does not switch

Open the runtime log:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```

The extension verifies the real open input/output after REAPER's native Preferences handler runs. If verification fails, the log usually explains one of these situations:

- the expected Audio Device popup could not be resolved;
- more than one REAPER window matched and the extension refused to guess;
- the target disappeared from the popup;
- REAPER's native popup action rejected the selection;
- the Preferences OK button could not be found;
- REAPER reopened a different device than requested.

The plugin is intentionally fail-safe: ambiguous controls are an error, not permission to click something approximately correct.

## The Preferences window flashes briefly

This is a known cosmetic limitation.

The reliable live-switch path is REAPER's own Audio Device Preferences handler. REAPER creates that window before its controls are available to the extension. The plugin immediately tries to hide newly shown windows and hides the resolved Preferences window again before selecting the popup and invoking OK, but the macOS compositor can still display a brief frame.

The plugin intentionally does **not** use invasive AppKit method swizzling, Accessibility automation, fake clicks, or screen-coordinate tricks just to hide this flash.

## There is an audible click or short pause when switching

Changing an audio interface requires REAPER to reopen/reconfigure its audio engine. A short interruption is therefore normal and depends on the hardware/driver.

The extension may perform one explicit `Audio_Quit()` / `Audio_Init()` reset if REAPER's Preferences handler updates its state but does not reopen the requested device immediately.

## REAPER Console opens unexpectedly

Runtime diagnostics are written through the REAPER console API and to the log file. If a build/version causes console visibility behavior that is undesirable, the file log remains the primary diagnostic source:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```

## Where is the REAPER resource path?

Use:

**Options → Show REAPER resource path in Finder**

This is safer than assuming a hard-coded location, especially for portable/custom installations.

The standard macOS installation normally uses:

```text
~/Library/Application Support/REAPER/
```
