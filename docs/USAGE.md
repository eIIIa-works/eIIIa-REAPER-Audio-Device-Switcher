# Usage

## Open the switcher

Run:

```text
eIIIa Audio Device: Show switcher
```

The menu is intentionally a compact vertical list. It does not repeat REAPER input/output identifiers or display a status sentence underneath the functional controls.

Each available row shows its slot number when assigned, the device/profile name, and channel information when available. The current profile is checked.

## Switch directly

Choose a row. The extension asks REAPER's own Audio Device Preferences handler to apply that configuration.

A short interruption or click in the audio stream is normal while REAPER closes/reopens the audio driver.

## Slots 1–9

Assign shortcuts to:

```text
eIIIa Audio Device: Switch to slot 1
...
eIIIa Audio Device: Switch to slot 9
```

This makes common switches effectively one-key actions.

## Refresh interfaces

Use **Refresh interfaces** after connecting/disconnecting hardware or when the Windows/Linux backend list has changed.

## Rebuild slots 1–9

Use **Rebuild slots 1–9** when you intentionally want the current available profiles to be renumbered.

## macOS behavior

Rows are based on CoreAudio devices and stable device UIDs. Input/output channel counts are shown when CoreAudio reports them.

## Windows/Linux behavior

The universal path treats the REAPER Audio Device page as the source of truth. It enumerates the Audio System choices REAPER exposes and synthesizes selectable profiles from driver/device selectors rather than assuming one fixed OS audio backend.

Windows/Linux are included in 0.4.0 but still need field testing. If discovery is incomplete on your REAPER build, send the runtime log and describe the visible Audio Device page.
