# Installation and Updates

This page covers the standard REAPER installation, portable/custom REAPER resource paths, updating, manual installation, and removal.

## Standard installation — easiest method

The project includes a one-click build-and-install script for the normal macOS REAPER resource location.

### 1. Quit REAPER

Do not replace a native extension while REAPER is using it.

### 2. Run the installer

Double-click:

```text
BUILD_AND_INSTALL.command
```

The script performs the following sequence:

1. downloads or updates the official REAPER SDK;
2. downloads or updates Cockos WDL;
3. verifies the expected SDK/WDL layout;
4. runs the source and C++ tests;
5. compiles the extension with Apple's `clang++`;
6. links `reaper_eIIIa_audio_device_switcher.dylib`;
7. keeps a build copy in the project `build/` directory;
8. replaces the standard installed copy in REAPER's `UserPlugins` directory.

Standard destination:

```text
~/Library/Application Support/REAPER/UserPlugins/reaper_eIIIa_audio_device_switcher.dylib
```

### 3. Restart REAPER

Native extensions are loaded when REAPER starts. They are **not** installed through **Actions → Load ReaScript**.

### 4. Confirm installation

Open:

**Actions → Show action list...**

Search for:

```text
eIIIa Audio Device
```

You should see the switcher action and slot actions 1–9.

---

## First-build developer tools

The build requires Apple's Command Line Tools. If they are not installed, run:

```bash
xcode-select --install
```

After installation finishes, run `BUILD_AND_INSTALL.command` again.

The script also needs internet access the first time because it clones:

- the official REAPER C/C++ extension SDK;
- Cockos WDL/SWELL.

They are kept under the project's `_deps/` directory and updated on subsequent builds.

---

## Portable REAPER or a custom resource path

`BUILD_AND_INSTALL.command` intentionally installs to the normal macOS REAPER resource location. If the REAPER instance you want to use has a different resource path, build first and copy the result manually.

### 1. Build without installing

Double-click:

```text
BUILD.command
```

The resulting extension is kept at:

```text
build/reaper_eIIIa_audio_device_switcher.dylib
```

### 2. Find the correct REAPER resource directory

In the target REAPER installation choose:

**Options → Show REAPER resource path in Finder**

### 3. Quit REAPER

### 4. Copy the extension

Copy:

```text
build/reaper_eIIIa_audio_device_switcher.dylib
```

to:

```text
<that REAPER resource path>/UserPlugins/
```

Create `UserPlugins` if it does not already exist.

### 5. Restart that REAPER installation

This method is also useful when maintaining several separate REAPER installations.

---

## Manual installation of a prebuilt `.dylib`

If a trusted prebuilt binary is provided:

1. In REAPER choose **Options → Show REAPER resource path in Finder**.
2. Quit REAPER.
3. Open `UserPlugins` inside that resource directory.
4. Copy `reaper_eIIIa_audio_device_switcher.dylib` into it.
5. Replace an older copy if updating.
6. Start REAPER.

A locally built binary is ad-hoc codesigned by the build script. Distribution/notarization of downloaded public binaries is a separate release concern and is not implied by the source build workflow.

---

## Updating

For the standard installation:

1. quit REAPER;
2. replace/update the project files;
3. run `BUILD_AND_INSTALL.command` again;
4. restart REAPER.

The installer replaces the existing `.dylib` atomically via a temporary sibling file and rename.

Slot assignments are stored separately in REAPER ExtState and are not erased by replacing the binary.

---

## Uninstalling

1. Locate the correct REAPER resource path.
2. Quit REAPER.
3. Delete:

```text
<UserPlugins>/reaper_eIIIa_audio_device_switcher.dylib
```

4. Restart REAPER.

If you installed the plugin into more than one portable/custom REAPER resource directory, remove the file from each one separately.

The extension stores only a very small slot-to-device-UID mapping in persistent REAPER ExtState. Leaving it behind has no effect after the plugin is removed.
