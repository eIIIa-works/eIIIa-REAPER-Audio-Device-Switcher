# Changelog

## Universal 0.4.1

- Added a transport guard before all user-facing audio-device actions.
- If REAPER is actively playing or recording, the extension calls REAPER's native `OnStopButton()` before opening/discovering/applying audio-device settings.
- `Show switcher` now stops active transport before discovery begins; direct slot switching does the same before applying the selected profile.
- If REAPER cannot be stopped, the device operation is refused instead of allowing the native Audio Device dialog to enter a failed/glitched state.
- Added `GetPlayState()` / `OnStopButton()` integration and regression tests for stopped, playing, recording, paused, unavailable-API, and stop-failure cases.
- Windows/Linux compile-smoke tests now include the shared transport guard.
- Documentation updated for transport-safe switching.

## Universal 0.4.0

- Removed the redundant `REAPER Input / Output` information row from the switcher menu.
- Removed the redundant bottom status row; operational details remain in the diagnostic log.
- Shortened the popup header so menu width is driven primarily by actual device/profile names.
- Preserved the proven macOS/CoreAudio + REAPER Preferences switching path.
- Added a shared cross-platform audio-profile model.
- Added Windows Win32 REAPER-Preferences discovery/apply path.
- Added Linux REAPER/SWELL discovery/apply path.
- Added native compact popup implementations for Windows/Linux.
- Added platform-native build/install launchers for macOS, Windows and Linux.
- Added Windows/Linux compile-smoke and policy tests.
- Documentation now distinguishes `implemented` from `field-tested`: macOS is tested; Windows/Linux require real-world validation.
