# Advanced Power Menu

Advanced Power Menu is an LSPosed module that provides a highly compatible extended advanced power menu.

Inspired by the stock Android power menu, it offers various reboot and lock actions with internal Android APIs whenever possible, while providing robust fallback mechanisms for ROM-specific behavior.

Support: [t.me/SuiAndroidMods](https://t.me/SuiAndroidMods)

## Features

- Reboot
- Recovery
- Bootloader
- Safe Mode
- Restart SystemUI
- Soft Reboot
    - Restart Zygote
    - Restart System Services
- Power Off
- Lockdown

> [!WARNING]
> Be aware that restarting Zygote or System services (equivalent soft reboot) can be unstable depending on the ROM.

## Rootless and root fallback

Advanced Power Menu does not rely on shell commands or `su` for normal operation. (Since the module's scope includes the system, root privileges are required to use LSPosed.)

Whenever possible, actions are executed directly from `system_server` using Android's internal APIs and Binder interfaces.

```
User Action
    ↓
SystemUI
    ↓
system_server
    ↓
Internal Android APIs
    ↓
Success
```

If an action fails due to ROM-specific implementations, Advanced Power Menu automatically falls back to a secondary implementation.

```
system_server
    ↓
Failed
    ↓
Broadcast result
    ↓
Fallback implementation
    ↓
Success
```
May need to manually grant root privileges to SystemUI.

Can also configure to always use the fallback.

## Compatibility

### Power menu

Advanced Power Menu uses multiple interception points to support differences between Android versions and custom ROMs.

The default implementation detects the standard **Power + Volume Up** power-menu request. When **Compatibility mode** is enabled, the module also uses additional framework-level fallback points, including:

* `PhoneWindowManager.showGlobalActionsInternal()`
* Policy `GlobalActions.showDialog()`
* `PhoneWindowManager.powerLongPress()`

Both modern and legacy `powerLongPress()` implementations are supported. Hook installation is isolated, so an unsupported method on one ROM does not prevent the remaining compatibility hooks from being installed.

Because power-menu behavior is frequently modified by device manufacturers and custom ROMs, compatibility mode may replace the configured long-press power action instead of the standard Power + Volume Up action.

### Power Action

The module is designed around Android's internal architecture rather than only shell commands.

Examples:

| Action | Primary Implementation | Fallback |
|-------|-------|-------|
| Reboots | PowerManagerService | su -c reboot [reason] |
| Safe Mode | PowerManagerService | persist.sys.safemode + su -c reboot |
| Soft Reboot | ctl.restart or ActivityManager.restart | ctl.restart or su -c am restart |
| Lockdown | LockPatternUtils + WindowManager | Binder Root Fallback |
| SystemUI Restart | killProcess() | - |

## Experimental Quick Settings support

- Added an optional experimental setting to replace the stock power menu opened from the Quick Settings power button.
- Uses the SystemUI Global Actions dialog entry point and keeps regular power-button invocations separate.
- The lock-screen exclusion setting also applies to Quick Settings invocations.
- Tested on AOSP-based ROMs.
- Compatibility may vary on ROMs with heavily customized SystemUI implementations.

## Lockdown

Lockdown mode is implemented using Android's native lockdown APIs.

The module performs:

- Strong authentication requirement
- Immediate device lock
- Work profile locking (best effort)

This behavior closely matches AOSP's stock implementation.

## Settings

- Enable / Disable
- Exclude lock screen
- Compatibility mode (Long-press power replacement)
- Replace QS Power Menu (Experimental)
- Soft reboot mode selection
- Long-press confirmation mode
- Force always use fallback

All settings are cached and applied immediately. No reboot or SystemUI restart is required.

## Required

- LSPosed

## Tested

- Nothing Phone 1 - Nothing OS 3.0 (A15)
- Nothing Phone 2a - Nothing OS 4.0 (A16)
- Poco X6 Pro - HyperOS 3.︎0.2.0 (A16)
- Xperia XZ1 Compact - LineageOS 21.0 UNOFFICIAL (A14)
- Xperia XZ1 Compact - LineageOS 22.2 UNOFFICIAL (A15)
- Xperia XZ Premium - crDroid 9.13 UNOFFICIAL (A13)

## Notes

Some actions may behave differently depending on your ROM implementation.

The module automatically prefers Android's internal APIs over shell commands whenever possible.

## License

CC-BY-NC-ND-4.0

See [`LICENSE`](https://github.com/Xposed-Modules-Repo/com.sui.advancedpowermenu/blob/main/LICENSE.md) for full details.
