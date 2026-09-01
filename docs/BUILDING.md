# Building from Source

## Supported target in this release

Native 0.3.1 currently builds for **macOS/CoreAudio**.

`CMakeLists.txt` deliberately rejects non-Apple builds for this release.

## Recommended build

From Finder, double-click:

```text
BUILD.command
```

or from Terminal:

```bash
./BUILD.command
```

The wrapper keeps its full log at:

```text
build/BUILD.log
```

The resulting extension is:

```text
build/reaper_eIIIa_audio_device_switcher.dylib
```

## Build and install in one operation

```bash
./BUILD_AND_INSTALL.command
```

This calls `BUILD.command` non-interactively and then installs the resulting binary into the standard REAPER `UserPlugins` directory.

Combined log:

```text
build/BUILD_AND_INSTALL.log
```

## What `build_macos.command` does

### Dependencies

It fetches or updates:

```text
_deps/reaper-sdk
_deps/wdl-source
```

from the official repositories:

- https://github.com/justinfrankel/reaper-sdk
- https://github.com/justinfrankel/WDL

The helper `scripts/link_wdl_into_sdk.sh` provides the WDL layout expected by the REAPER SDK includes.

### Tests

Before compiling the plugin, the build runs source-policy tests and C++ logic tests covering areas such as:

- SDK/WDL layout;
- startup safety;
- popup UI safety;
- Preferences bridge policy;
- Preferences hide policy;
- installer behavior;
- configuration parsing/mapping;
- slot assignment;
- Preferences popup matching;
- CoreAudio write-policy investigation helpers;
- INI writing helpers.

### Compilation

The build uses Apple's compiler through `xcrun clang++` with C++17.

C++ files and Objective-C++ files are compiled separately; Objective-C++ uses ARC.

The final extension links:

```text
-framework Cocoa
-framework CoreAudio
-framework CoreFoundation
```

and is written as:

```text
reaper_eIIIa_audio_device_switcher.dylib
```

The build attempts ad-hoc codesigning of the locally generated binary.

## CMake

A CMake configuration is included for development/tooling. It expects the official REAPER SDK under:

```text
_deps/reaper-sdk
```

and expects WDL to be visible as:

```text
_deps/reaper-sdk/WDL/swell/swell.h
```

The easiest way to prepare those dependencies is still to run `build_macos.command` once.

Example:

```bash
./build_macos.command
cmake -S . -B build-cmake
cmake --build build-cmake
ctest --test-dir build-cmake --output-on-failure
```

## Source responsibilities

| File | Responsibility |
| --- | --- |
| `src/plugin_main.cpp` | REAPER extension entry point and Action registration |
| `src/plugin_state.cpp` | high-level state machine and switch verification |
| `src/coreaudio_devices.mm` | CoreAudio enumeration |
| `src/device_window.mm` | native switcher menu |
| `src/reaper_prefs_bridge.mm` | native REAPER Preferences integration |
| `src/prefs_popup_logic.cpp` | safe popup matching |
| `src/slot_assignment.cpp` | persistent UID slot mapping |
| `src/audio_config_bridge.cpp` | REAPER audio-config investigation/helper layer |
| `src/ini_device_prefs.cpp` | REAPER configuration parsing |
| `src/ini_writer.cpp` | narrow INI update helper |

## Testing on a real Mac

Logic/source tests cannot prove that a particular REAPER/macOS version presents the same native Preferences hierarchy. A release should therefore also be manually checked inside REAPER:

1. extension loads at startup without hanging;
2. all Actions appear;
3. switcher menu opens;
4. devices and channel counts are correct;
5. slot Actions work;
6. hardware → headphones switch works;
7. headphones → hardware switch works;
8. virtual CoreAudio device can be selected when available;
9. runtime verification reports the correct target;
10. Preferences flash is limited to the known brief cosmetic behavior;
11. unplugged slot fails safely.
