# Windows and Linux Testing

macOS is already field-tested. Windows and Linux need community verification.

## Acceptance checklist

1. Build succeeds using the platform launcher.
2. REAPER starts normally with the extension installed.
3. Actions named `eIIIa Audio Device` appear.
4. `Show switcher` opens a compact device/profile list.
5. All expected REAPER Audio System variants are represented.
6. Selecting a profile causes REAPER to switch audio successfully.
7. Slots 1–9 persist across REAPER restart.
8. Disconnected/unavailable profiles fail safely.
9. REAPER's system-wide OS audio default is not changed.
10. Send the runtime log for any failure or incomplete discovery.

## Report template

```text
OS:
CPU:
REAPER version:
Audio System/backend:
Device/driver:
Build succeeded: yes/no
Show switcher discovery: complete/incomplete
Direct switch: works/fails
Slot switch: works/fails
Notes:
```

Attach `eIIIa_Audio_Device_Switcher.log` when possible.
