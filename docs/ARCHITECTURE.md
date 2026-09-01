# Technical Architecture

## Design goal

The extension needs to switch REAPER's active audio interface quickly without changing the macOS system default audio device and without relying on fragile UI automation.

The central difficulty is that REAPER's public C/C++ extension API exposes the current audio-device state but does not provide a documented public call equivalent to:

```text
SetAudioDevice(target)
ApplyAudioDevicePreferences()
```

The working implementation therefore separates **device discovery** from **device application**:

- CoreAudio provides discovery and stable identity.
- REAPER itself remains the authority that applies an audio-device change.

## Components

### `plugin_main.cpp`

Responsibilities:

- REAPER extension entry point (`ReaperPluginEntry`);
- load only the required REAPER API functions;
- register the extension name/vendor;
- register the switcher action;
- register slot Actions 1–9;
- register `hookcommand`;
- register the timer used for the asynchronous Preferences operation.

Startup is intentionally side-effect free. The entry point does not enumerate CoreAudio, create AppKit windows, perform file I/O, or reset the audio engine while REAPER is loading extensions.

### `coreaudio_devices.mm`

Uses Apple's CoreAudio APIs to enumerate `AudioDeviceID` values.

For each live device it obtains:

- human-readable name;
- `kAudioDevicePropertyDeviceUID`;
- input channel count;
- output channel count.

Only devices with a valid UID/name and at least one input or output channel are retained.

The list is sorted by name and then UID.

The CoreAudio UID is the stable identity used by slots. The temporary numeric `AudioDeviceID` is diagnostic only.

### `device_window.mm`

Implements the visible switcher as a transient native `NSMenu`.

The menu contains:

- current REAPER input/output identifiers;
- device rows;
- slot numbers;
- channel counts;
- active-state check marks;
- refresh/rebuild commands.

There is no persistent custom window to manage at REAPER startup.

### `slot_assignment.cpp`

Maintains nine persistent slots.

The slot storage value is the CoreAudio UID. Existing non-empty slot values are preserved, and newly enumerated UIDs fill empty slots.

Persistent storage is provided by REAPER ExtState using section:

```text
eIIIaAudioDeviceSwitcher
```

and keys:

```text
slot1 ... slot9
```

### `plugin_state.cpp`

Owns the high-level switch state machine.

When a device is selected:

1. re-enumerate CoreAudio;
2. resolve REAPER's current input and output with `GetAudioDeviceInfo()`;
3. determine which supported directions actually need to change;
4. capture currently visible application windows;
5. invoke REAPER command `40099` (Audio device configuration);
6. asynchronously wait for the native controls to become available;
7. apply the target through `reaper_prefs_bridge.mm`;
8. verify the actual open device;
9. optionally perform one `Audio_Quit()` / `Audio_Init()` reset if necessary;
10. report success/failure.

### `prefs_popup_logic.cpp`

Contains platform-independent matching rules used by the native Preferences bridge.

A popup can be selected only when:

- it contains the requested target device;
- its currently selected item matches the current input or current output that needs changing.

For one direction, exactly one popup must match. For two directions, one shared popup or two distinct popups are accepted. Other cases are treated as ambiguous.

This rule is important: the bridge does not select a popup merely because it happens to contain the target text.

### `reaper_prefs_bridge.mm`

This is the key integration layer.

It inspects REAPER/AppKit windows after REAPER's Audio Device action has been invoked, gathers native popup controls and buttons, and identifies one window whose control structure safely matches the requested change.

Once resolved:

1. the Preferences window is hidden again (`orderOut:`);
2. the exact device popup item is selected;
3. the popup's native target/action is dispatched through `NSApp`;
4. the Preferences **OK** button is found and clicked programmatically.

This deliberately executes REAPER's own preference-change handler instead of trying to reproduce REAPER's internal CoreAudio configuration logic.

### Verification

`GetAudioDeviceInfo()` is used after the native Preferences handler runs:

```text
IDENT_IN
IDENT_OUT
```

The returned REAPER identifier is mapped back to the enumerated CoreAudio device. Success requires the resulting UID to equal the target UID for every direction that was supposed to change.

## Why direct configuration writes are not the primary switching path

The source tree still contains configuration-discovery/write helpers because they were developed while investigating REAPER's audio state and remain covered by tests. They document useful behavior of REAPER's configuration surface.

They are **not** the primary 0.3.1 live-switch mechanism.

During development it was established that, on the tested macOS REAPER build:

- the CoreAudio device preference names can be present in REAPER's configuration;
- `set_config_var_string()` does not provide a reliable live setter for these device fields;
- writing `reaper.ini` and calling only `Audio_Quit()` / `Audio_Init()` does not force REAPER to reload the newly written CoreAudio preference as a live device change.

The native Preferences handler is therefore the proven apply boundary.

## Why not change the macOS default device?

CoreAudio can change system-level defaults, but that would affect every application on the Mac. This plugin is specifically intended to change **REAPER only**.

System-default switching would also produce surprising interactions with browsers, communication software, media players, and other audio applications.

## Why not use Accessibility/UI scripting?

The extension intentionally avoids:

- System Events;
- AppleScript/`osascript`;
- Accessibility permissions;
- mouse coordinates;
- synthetic pointer movement/clicks.

Those approaches are sensitive to window position, monitor layout, UI scaling, localization, focus, and unrelated user input.

The implemented bridge talks directly to the native REAPER/AppKit controls inside the REAPER process.

## Preferences flash

The bridge captures a baseline of currently visible windows before command `40099`, then hides newly shown windows and hides the resolved Preferences window again before applying the controls.

This reduces presentation time, but it cannot guarantee that AppKit/macOS never composites a frame before the extension receives the window object. Completely intercepting presentation would require invasive modification of AppKit/REAPER behavior and is intentionally outside the design.

## Failure policy

Safety is preferred over clever guesses.

The operation fails if, for example:

- the requested device disappears;
- the target is not present in the expected popup;
- control matching is ambiguous;
- more than one REAPER window matches;
- the OK button is not found;
- the native control action fails;
- post-switch verification does not match the target UID.

Detailed structure is written to the diagnostic log where possible.

## Runtime dependencies

The compiled extension links the macOS system frameworks:

- Cocoa;
- CoreAudio;
- CoreFoundation.

It uses the official REAPER extension SDK and WDL/SWELL at build time.
