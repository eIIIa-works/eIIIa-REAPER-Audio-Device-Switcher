# Usage Guide

## Open the switcher

In REAPER:

1. open **Actions → Show action list...**;
2. search for `eIIIa Audio Device`;
3. run **eIIIa Audio Device: Show switcher**.

A compact native menu appears at the mouse pointer.

Each device row contains:

```text
slot   device name   input/output channel count
```

A check mark indicates that REAPER is currently using that interface for at least one supported direction.

## Switch directly from the menu

Click a device row.

The extension determines whether that target provides input, output, or both and asks REAPER to apply the corresponding change through REAPER's own Audio Device Preferences controls.

After the native handler runs, the extension verifies the actual open device. A request is not reported as successful merely because a preference control changed.

## Use slots 1–9

The extension exposes nine separate REAPER Actions:

```text
eIIIa Audio Device: Switch to slot 1
...
eIIIa Audio Device: Switch to slot 9
```

Assign shortcuts to the slots you use most often:

1. open the REAPER Action List;
2. select a slot action;
3. use REAPER's normal **Add...** shortcut assignment control;
4. press the key or MIDI/OSC trigger you want to use.

After that, switching does not require opening the menu.

## How slot assignment behaves

Slots store stable CoreAudio UIDs.

- Already assigned devices keep their number.
- Newly discovered devices fill empty slots until slots 1–9 are full.
- If a device is unplugged, its stored slot is not silently reassigned to another device.
- Triggering a slot whose interface is disconnected produces a clear status message.

### Rebuild slots 1–9

Use **Rebuild slots 1–9** when you deliberately want to discard the existing numbering and build a new map from the interfaces currently connected.

The devices returned by the current macOS implementation are sorted by device name and then UID, so rebuilding produces deterministic numbering for the same connected set.

## Refresh interfaces

Use **Refresh interfaces** after connecting/disconnecting hardware if you want to refresh the menu immediately.

The plugin also re-enumerates devices before a slot/device switch, so it does not rely solely on an old menu snapshot.

## Input-only and output-only devices

The menu can contain:

- input-only devices;
- output-only devices;
- devices with both input and output.

The switcher does not force a nonexistent direction. For example, selecting an output-only endpoint changes REAPER's output but leaves the current input alone.

## Brief Preferences window flash

During a switch, REAPER's Audio Device Preferences window may appear for a very short moment.

This is expected with the current architecture: REAPER must construct its native preferences controls before the extension can drive the same internal apply path that REAPER uses itself. The extension hides the window as soon as it can, but macOS may draw a frame before that happens.

No mouse automation or Accessibility permission is involved.

## Status and errors

The switcher reports useful states such as:

- REAPER is already using the selected device;
- slot is empty;
- assigned interface is disconnected;
- selected device disappeared during the operation;
- Preferences controls could not be identified safely;
- REAPER's actual reopened device did not match the requested device.

Detailed diagnostics are also appended to:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```
