# Building from Source

## One source tree, three native outputs

```text
macOS    reaper_eIIIa_audio_device_switcher.dylib
Windows  reaper_eIIIa_audio_device_switcher.dll
Linux    reaper_eIIIa_audio_device_switcher.so
```

The codebase is shared, but each platform must be compiled for its own native ABI.

## Dependencies fetched automatically

The platform build scripts fetch:

- official REAPER SDK: `https://github.com/justinfrankel/reaper-sdk`
- official Cockos WDL: `https://github.com/justinfrankel/WDL`

into `_deps/`.

## macOS

Build only:

```text
BUILD.command
```

Build + install:

```text
BUILD_AND_INSTALL.command
```

The macOS script uses Apple's `xcrun clang++` directly and links Cocoa, CoreAudio and CoreFoundation.

## Windows

Use x64 Visual Studio developer tools.

Build only:

```bat
BUILD.bat
```

Build + install:

```bat
BUILD_AND_INSTALL.bat
```

CMake generates a Visual Studio/MSVC build because the official REAPER SDK expects the MSVC-compatible ABI on Windows.

## Linux

Build only:

```bash
./BUILD.sh
```

Build + install:

```bash
./BUILD_AND_INSTALL.sh
```

The Linux target uses REAPER/WDL SWELL and builds `swell-modstub-generic.cpp` with `SWELL_PROVIDED_BY_APP`.

## Tests included

The source package includes pure C++ tests and policy/compile-smoke tests for:

- stable profile identity;
- profile synthesis;
- slot assignment;
- conservative Preferences matching;
- compact menu policy;
- prohibition of system-default/mouse automation mechanisms;
- Linux/SWELL source compilation through a test shim;
- Windows/Win32 source compilation through a test shim;
- macOS 0.3.x Preferences-handler regression rules.

These tests do not replace real REAPER testing on the target OS.
