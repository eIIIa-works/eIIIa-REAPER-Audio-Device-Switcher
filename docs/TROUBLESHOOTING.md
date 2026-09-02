# Troubleshooting

## Actions do not appear

Verify the extension binary is in the `UserPlugins` directory of the REAPER resource path you are actually running. Restart REAPER after copying/replacing a native extension.

## macOS: build script will not run

If Finder blocks execution, right-click the command and choose **Open**, or run it from Terminal. Install Command Line Tools if requested:

```bash
xcode-select --install
```

## Windows: build fails before compilation

Use an **x64 Native Tools Command Prompt for Visual Studio** and verify:

```bat
cl
cmake --version
git --version
```

The Windows target is intended for MSVC, not MinGW.

## Linux: build cannot find compiler or CMake

Verify:

```bash
c++ --version
cmake --version
git --version
```

## Switcher opens but a device/profile is missing

Use **Refresh interfaces**. On macOS, confirm the device is visible to CoreAudio. On Windows/Linux, confirm the backend/device is present in REAPER's own **Preferences → Audio → Device** page.

## A switch fails

Find:

```text
<REAPER resource path>/eIIIa_Audio_Device_Switcher.log
```

The log is especially important for Windows/Linux testing because different REAPER builds/backends can expose different native control layouts.

## macOS Preferences flashes briefly

Known cosmetic limitation. It is not mouse automation and it does not indicate that system audio is being changed.

## Windows/Linux: please report failures

Include:

- OS and version;
- CPU architecture;
- REAPER version;
- Audio System/backend selected in REAPER;
- device/driver name;
- whether `Show switcher` discovers it;
- whether direct selection works;
- whether slot Actions work;
- the runtime log.
