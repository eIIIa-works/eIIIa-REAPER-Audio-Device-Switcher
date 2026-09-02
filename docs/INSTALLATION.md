# Installation

## First: find your REAPER resource path

REAPER loads native extensions from the `UserPlugins` directory inside its **resource path**.

In REAPER use:

**Options → Show REAPER resource path in Explorer/Finder**

Portable REAPER installations can have a different resource path, so this is the authoritative way to find it.

## macOS

### Requirements

- REAPER for macOS
- Apple Command Line Tools
- Git
- Internet access for the first build

Install Apple's tools once if needed:

```bash
xcode-select --install
```

### Automatic build + install

1. Quit REAPER.
2. Extract the source package.
3. Double-click `BUILD_AND_INSTALL.command`.
4. Wait for tests and compilation to finish.
5. Restart REAPER.
6. Open **Actions → Show action list...** and search for `eIIIa Audio Device`.

The script keeps a copy at:

```text
build/reaper_eIIIa_audio_device_switcher.dylib
```

and installs/replaces the standard copy at:

```text
~/Library/Application Support/REAPER/UserPlugins/reaper_eIIIa_audio_device_switcher.dylib
```

For portable REAPER, copy the built `.dylib` manually into that REAPER installation's `UserPlugins` folder.

## Windows

### Requirements

- 64-bit REAPER for Windows
- Visual Studio 2022 or Visual Studio Build Tools with **Desktop development with C++**
- CMake
- Git

The official REAPER SDK uses a C++ interface that expects the MSVC-compatible ABI on Windows, so MSVC is the supported build route.

### Automatic build + install

1. Quit REAPER.
2. Extract the source package.
3. Open **x64 Native Tools Command Prompt for VS 2022**.
4. `cd` to the extracted folder.
5. Run:

```bat
BUILD_AND_INSTALL.bat
```

The script builds:

```text
build\reaper_eIIIa_audio_device_switcher.dll
```

and installs it to:

```text
%APPDATA%\REAPER\UserPlugins\reaper_eIIIa_audio_device_switcher.dll
```

For portable REAPER, copy the built DLL to the portable resource path's `UserPlugins` folder instead.

### Build only

```bat
BUILD.bat
```

## Linux

### Requirements

- REAPER for Linux
- a C++17 compiler (`g++` or compatible)
- CMake
- Git

### Automatic build + install

```bash
chmod +x BUILD_AND_INSTALL.sh
./BUILD_AND_INSTALL.sh
```

The script builds:

```text
build/reaper_eIIIa_audio_device_switcher.so
```

and installs it to:

```text
${XDG_CONFIG_HOME:-$HOME/.config}/REAPER/UserPlugins/reaper_eIIIa_audio_device_switcher.so
```

For a portable/custom REAPER resource path, copy the `.so` manually into its `UserPlugins` directory.

### Build only

```bash
./BUILD.sh
```

## After installation

Restart REAPER. In **Actions**, search for:

```text
eIIIa Audio Device
```

You should see `Show switcher` and slot Actions 1–9.

## Updating

Build/install the newer source over the old plugin and restart REAPER. The installer replaces the binary but does not intentionally clear slot ExtState.

## Uninstalling

Quit REAPER and delete the platform binary from `UserPlugins`:

```text
reaper_eIIIa_audio_device_switcher.dylib   macOS
reaper_eIIIa_audio_device_switcher.dll     Windows
reaper_eIIIa_audio_device_switcher.so      Linux
```

Restart REAPER.
