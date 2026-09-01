# FAQ

## Does this change my Mac's default audio device?

No. The extension changes REAPER's audio configuration only. Other applications keep using whatever macOS is configured to use.

## Does it need ReaPack?

No.

## Does it need SWS, ReaImGui or Lua?

No. This is a native REAPER C++/Objective-C++ extension loaded from `UserPlugins`.

## Does it need Accessibility permission?

No. It does not drive the mouse, use screen coordinates, or use AppleScript/System Events.

## Why does REAPER Preferences briefly flash during switching?

The current reliable live-switch mechanism uses REAPER's own Audio Device Preferences handler. REAPER must create that native window before its controls exist. The extension hides it as quickly as possible, but macOS can still composite a frame before the hide request takes effect.

## Why not just write the device name into `reaper.ini`?

That was tested during development. Updating the CoreAudio preference in the file and calling only `Audio_Quit()` / `Audio_Init()` did not make the tested REAPER build reload the new device as a live switch. REAPER's own Preferences Apply/OK path does.

## Why not change CoreAudio's system default instead?

Because that would change audio routing for every application, not just REAPER. The goal of this extension is specifically a REAPER-local switch.

## What are slots 1–9?

They are persistent REAPER Actions pointing to stable CoreAudio device UIDs. They are useful for keyboard shortcuts, toolbar buttons, MIDI/OSC triggers and custom actions.

## What happens when a slotted interface is unplugged?

The slot remains assigned to that UID and reports that the interface is not connected. Another device is not silently substituted.

## Can I have an output-only device in a slot?

Yes. The current macOS implementation enumerates devices with input, output, or both. A direction the target device does not provide is left unchanged.

## Does the extension switch sample rate or buffer size?

The current user-facing switch operation targets the device selection. Other Audio Device Preferences remain under REAPER's control and are not presented as separate switcher settings.

## Is there a prebuilt binary?

The documented 0.3.1 package is designed to build locally with `BUILD_AND_INSTALL.command`. A public prebuilt/notarized distribution workflow is separate from this source-build workflow.

## Where are diagnostics stored?

Runtime log:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```

Build logs:

```text
build/BUILD.log
build/BUILD_AND_INSTALL.log
```

Use **Options → Show REAPER resource path in Finder** to locate the active REAPER resource directory.

## Does it support Windows or Linux?

Not in Native 0.3.1. This release is explicitly macOS/CoreAudio. A cross-platform design can reuse the same principle—letting REAPER apply its own audio settings—but Windows and Linux require separate platform adapters and should not be advertised as supported until they are tested in real REAPER builds.
