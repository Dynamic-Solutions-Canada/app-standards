# Tray

How a resident Windows app lives in the notification area: its icon, clicks, menu, flyout, host window, and how it starts and exits.

These rules apply to the 5 resident WinUI apps (hub-display-switch, wsl-status, little-launcher, immich-drive, and persistent's desktop app), and counts are out of those 5 unless a rule says otherwise. repilot is deliberately not resident (its key press is handled by a separate short-lived process), so it has no tray icon and only TRAY-9 applies to it. Android's nearest equivalent, the foreground-service notification, is not covered here.

The mechanics are provided by winui-app-kit and documented in [its README](https://github.com/Dynamic-Solutions-Canada/winui-app-kit) (local copy: `wsl-status/external/winui-app-kit/README.md`). This file says what every app must do with them; it does not describe the kit's API. Which apps must be on the kit is KIT-2 in [shared-kits.md](shared-kits.md).

## Overview

| Concern | Kit type | Rule |
| --- | --- | --- |
| Icon, GUID, Explorer-restart recovery, click events | `TrayIcon` | TRAY-1 to TRAY-4 |
| Right-click menu | `TrayMenu` | TRAY-5 |
| Flyout window | `FlyoutController` | TRAY-6 |
| Host window, exit | none yet (see KIT-9) | TRAY-7, TRAY-8 |
| One running copy | `SingleInstance` | TRAY-9 |
| Start with Windows | `StartupRegistration`, `StartupStatus` | TRAY-10 |

## Rules

### TRAY-1: Use the kit's tray types

- **Status:** Existing (2 of 5: hub-display-switch, wsl-status; the other three call `Shell_NotifyIcon` directly)
- **Applies to:** Windows
- **Reference:** `wsl-status/WslMonitor/MainWindow.xaml.cs`; hub-display-switch commit `af41968` for migrating an existing app
- **Why:** Three menu implementations and three icon implementations are in use for the same behaviour, and the hand-written ones are the ones missing a GUID, dark-mode menus and light-taskbar support.

A resident app creates its icon with the kit's `TrayIcon` and its menu with `TrayMenu`. It does not call `Shell_NotifyIcon` itself and does not add a tray library (H.NotifyIcon is being removed). little-launcher is the one exception until the kit supports several icons per host window, because it shows one icon per launcher. An app that needs behaviour the kit lacks adds a general option to the kit (KIT-8).

### TRAY-2: Icon source and states

- **Status:** Decided (D-071). Existing (3 of 5 draw the icon in code and follow the taskbar's light or dark colour: hub-display-switch, wsl-status, little-launcher; 2 of 5 load `.ico` variants: immich-drive, persistent)
- **Applies to:** Windows
- **Reference:** `wsl-status/WslMonitor/Classes/TrayIconRenderer.cs`
- **Why:** A dark glyph on a dark taskbar, or a 32 px frame scaled down, is the most visible quality difference between the apps.

Either source is allowed (D-071):

- **Drawn in code** at the exact tray pixel size for the current scaling (sharpest; the shared rasteriser is a kit candidate, KIT-9), or
- **`.ico` files** with one variant per state, where brand artwork matters more.

Either way the icon must be legible on both light and dark taskbars and switch when the taskbar colour changes. If the app has a state the user cares about (active or paused, running, offline), the icon shows it with a small, consistent change such as a dot or a muted variant, and the tooltip is the app name followed by the current state (3 of 5; persistent shows the name only). The form is `<App> – <state>` (D-071), with a spaced en dash, as hub-display-switch and wsl-status write it. Tray artwork comes from the same master as the app icon; see [assets.md](assets.md).

### TRAY-3: Stable GUID, and the icon always comes back

- **Status:** Decided (D-071) for the GUID (3 of 5: hub-display-switch, wsl-status, little-launcher). Existing (5 of 5) for re-adding after Explorer restarts
- **Applies to:** Windows
- **Reference:** winui-app-kit `Tray/TrayIcon.cs`
- **Why:** Windows remembers the user's "show this icon" choice by GUID. Without one, the choice is lost whenever the executable path changes, which for a packaged app is every update.

- Each app has one fixed tray GUID, generated once and never changed. It is a permanent identifier: it survives a move to the kit unchanged (as hub-display-switch's did) and is recorded in the identifier registry (BRAND-10 in [branding.md](branding.md)). little-launcher uses one stable GUID per launcher.
- The icon is re-added when Explorer restarts.
- If the icon cannot be added, the app logs a warning and opens its main surface, so it is never running with no visible way in (persistent does this today).

### TRAY-4: Clicks

- **Status:** Existing (5 of 5) for left click. Decided (D-071) for double click (3 of 5 open the flyout: hub-display-switch, wsl-status, persistent)
- **Applies to:** Windows
- **Reference:** `wsl-status/WslMonitor/MainWindow.xaml.cs`
- **Why:** Users click tray icons without reading; the same gesture must do the same thing in every app.

| Gesture | Action |
| --- | --- |
| Left click | Toggle the flyout (open it, or close it if it is open) |
| Double click | The same as a single click (D-071). It does not open Settings and is not left unhandled |
| Right click | Open the menu (TRAY-5) |

### TRAY-5: Menu items and order

- **Status:** Decided (D-071). Existing (2 of 5 match the order exactly: hub-display-switch, wsl-status; 5 of 5 end with a separator and "Exit")
- **Applies to:** Windows
- **Reference:** `wsl-status/WslMonitor/MainWindow.xaml.cs`
- **Why:** The bottom of the menu is muscle memory. Settings and Exit must be where the user's hand already goes.

The order, top to bottom:

1. **App actions:** whatever the app does from the tray (an "Open <App>" item, a status line, per-item choices, pause or resume). Separators may group them.
2. **Settings:** labelled "Settings", not "App Settings". It opens the settings window on its Settings page.
3. **Open logs folder:** the same action as on the About page (LOG-8 in [logging.md](logging.md)).
4. A separator.
5. **Exit:** always last. It is the only way to quit the app (TRAY-8).

Menu text is localized with the rest of the app, uses sentence case, and the menu follows dark mode (the kit's `TrayMenu` does; the hand-built menus in 3 of 5 do not).

### TRAY-6: Flyout

- **Status:** Existing (2 of 5 use the kit's `FlyoutController`: hub-display-switch, wsl-status; immich-drive has its own equivalent)
- **Applies to:** Windows
- **Reference:** `hub-display-switch/HubDisplaySwitch/Windows/SwitcherWindow.xaml.cs` (builds content only; the kit does the rest)
- **Why:** The flyout is the surface users see most. Placement, dismissal and how fast it opens should not differ between apps.

A tray flyout is a window handed to the kit's `FlyoutController`: desktop acrylic, rounded corners, placed by the taskbar, dismissed on focus loss or Esc, and parked off-screen so it reopens instantly. The app's flyout class only builds content. Two flyouts are outside this rule as they stand: persistent's resizable web view and little-launcher's launcher panel, which is its product surface. Confirmation inside a flyout follows UI-6 in [ui.md](ui.md).

### TRAY-7: Hidden host window and exit

- **Status:** Existing (5 of 5) for the host window. Decided (D-074) for how it is hidden and how the app exits (3 of 5 use the decided form: hub-display-switch, wsl-status, little-launcher)
- **Applies to:** Windows
- **Reference:** `wsl-status/WslMonitor/MainWindow.xaml.cs`
- **Why:** The tray icon needs a window that outlives every visible one, and exit must not leave the icon, a connection or a notification registration behind.

- `MainWindow` is a hidden host titled "<App> Host". It owns the tray icon, is styled as a tool window so it never appears in the taskbar or Alt+Tab, and cancels its own close.
- **Host and exit (D-074):** the host is a 1x1 window moved off-screen, and Exit calls `Environment.Exit(0)` after an orderly-exit hook has run. An app with teardown to do (persistent closes a connection and unregisters notifications) does it in that hook; no app skips teardown to exit faster.
- The host setup is about 10 lines retyped in each app and is a kit candidate (KIT-9).

### TRAY-8: Closing a window never exits

- **Status:** Existing (5 of 5)
- **Applies to:** Windows
- **Reference:** any resident app
- **Why:** A resident app that quits when its settings window closes stops doing its job without telling the user.

Closing the settings window or the flyout closes only that window; the host keeps the app running. The only way out is "Exit" in the tray menu. There is no "minimise to tray" or "exit on close" setting, because there is nothing to choose between.

### TRAY-9: One running copy

- **Status:** Existing (6 of 6 WinUI apps use a named mutex, repilot included; 3 of 5 resident apps exit quietly on a silent second launch; 2 of 6 use the kit's `SingleInstance`)
- **Applies to:** Windows
- **Reference:** winui-app-kit `SingleInstance.cs`; `hub-display-switch/HubDisplaySwitch/App.xaml.cs` (checks before settings load)
- **Why:** Two copies means two tray icons and two writers to one settings file.

- The app takes a named mutex at startup, before it loads settings, through the kit's `SingleInstance`. The mutex name is fixed for the life of the product and is kept when an app moves to the kit.
- A second launch by the user asks the first copy to show its main surface (settings window or flyout), then exits.
- A second launch with `--silent` (startup, or a restart after an update) exits without showing anything.

### TRAY-10: Start with Windows

- **Status:** Existing (3 of 5 have the full mechanics: hub-display-switch, wsl-status, little-launcher). Where the state lives is Decided (D-073) (2 of 5 follow it). The label is Decided (D-072)
- **Applies to:** Windows
- **Reference:** winui-app-kit `Startup/StartupRegistration.cs`; `wsl-status/WslMonitor/Pages/GeneralPage.xaml.cs`
- **Why:** Windows, the user (in Task Manager or Settings) and group policy can all change startup behind the app's back. A switch that shows a stored copy can disagree with what will actually happen at sign-in.

Mechanics, all provided by the kit's `StartupRegistration`:

- A manifest startup task, enabled by default, launching with `--silent`. The task id is fixed for the life of the product.
- A Run-key fallback with `--silent` when the app runs unpackaged, and removal of stale Run entries when it runs packaged.
- The app never re-enables startup that the user disabled in Windows.
- `RegisterApplicationRestart("--silent")` at startup, so a restart after an update comes back quietly.

State (D-073): the switch shows the live `StartupStatus` read from Windows and is disabled, with the reason, when Windows Settings or a policy decides. Any copy kept in `settings.json` is overwritten with the live state and never competes with it, and "reset settings" leaves the startup task alone (see CONF-4 in [settings-config.md](settings-config.md)).

The switch is labelled "Start with Windows" (D-072) and sits on the Settings page (UI-4 in [ui.md](ui.md)). A non-resident app has no startup switch and no startup setting.
