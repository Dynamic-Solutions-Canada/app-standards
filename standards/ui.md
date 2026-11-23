# UI

What the apps look like and how their common screens behave, per stack. The aim is that a user who knows one app finds the same window, the same Settings and About pages, and the same dialog behaviour in the next one.

Counts are out of the 6 WinUI apps (display-switchr, tray-control-for-wsl, little-launcher, repilot, immich-drive, and persistent's desktop app), the 4 web monorepos (persistent, printstream, game-is-up, store-sign-controller), or the 2 native Android apps (phonelock, sonarbridge) unless a rule says otherwise. store-sign-controller's desktop host and the Capacitor mobile shells are web views around the web app; they follow the Web rules for their content.

Related topics: the tray icon and flyout are in [tray.md](tray.md), the "Our other apps" page in [promo.md](promo.md), interface languages in [locales.md](locales.md), icon artwork in [assets.md](assets.md), and which shared code delivers these screens in [shared-kits.md](shared-kits.md). The conventions rule in the user-level agent rules (kept outside this repo) (find the sibling surface first, extract on the third copy) applies to all UI work and is not restated here.

## Overview

| Stack | Toolkit | Best reference | Project UI guide |
| --- | --- | --- | --- |
| Windows | WinUI 3, Windows App SDK, winui-app-kit | `tray-control-for-wsl/WslMonitor/` (layout and kit use), `persistent/apps/desktop/Persistent.Desktop/Pages/AboutPage.xaml` (About content) | none yet |
| Web | React, Joy UI, TanStack Query | `printstream/apps/web/src/components/` | `printstream/.agents/guides/ui-conventions.md` |
| Android | Material 3; Compose for new apps (D-056) | `phonelock/companion/` (`AppDialog.kt`, `ListRow.kt`, `SettingsScreen.kt`) | none yet |

Every app with a UI keeps a written guide at `.agents/guides/ui-conventions.md` (1 of the 11 end-user apps today: printstream). AI-6 in [ai-docs.md](ai-docs.md) owns that rule, and the location is D-026. The guide holds what is specific to that product; it does not copy these rules.

## Windows

### UI-1: Settings window shell

- **Status:** Existing (6 of 6 for the shell; 2 of 6 take icon, placement and theme tracking from the kit)
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitor/DetailsWindow.xaml` and `.xaml.cs`; `display-switchr/HubDisplaySwitch/SettingsWindow.xaml` for an app whose window is named `SettingsWindow`
- **Why:** The main window is the first thing users compare between apps. All six already share this shape by copy and paste, so writing it down stops it drifting.

Every app's main window has:

- A Mica backdrop with content extended into the title bar, and a transparent `NavigationViewContentBackground` override in `App.xaml`.
- A 48 px custom title bar holding a 16 px app icon and the caption text.
- A left-pane `NavigationView` with the back button hidden and the built-in Settings item switched on. The open pane is 240 wide (2 of 6; little-launcher sizes its pane to the widest label because of its long translated labels, which is an accepted variation).
- One instance of the window per process (`ShowInstance()` or equivalent), with its size and position remembered and ignored when the monitor is gone.

The window class is `SettingsWindow` (5 of 6). Window icon, saved placement and bring-to-front come from winui-app-kit (`WindowIcon`, `WindowChrome`, `WindowPlacement`), not from local Win32 code. Navigation and card icons are `FontIcon` glyphs from the system icon font (6 of 6); no icon library is added. The app icon ships as `Resources/<App>.ico` plus a 256 px `<App>.png` for in-app images (3 of 6), because XAML decodes an `.ico` at its 16 px frame; see [assets.md](assets.md).

### UI-2: Navigation order and footer

- **Status:** Existing (6 of 6)
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitor/DetailsWindow.xaml`
- **Why:** A fixed footer means "About" and the sibling apps are in the same place in every app, whatever the app's own pages are.

The menu items are the app's own pages, starting with its main content (a Home page, or the per-item pages in display-switchr and tray-control-for-wsl). The footer holds exactly two items, in this order: "Our other apps" (glyph E719; see [promo.md](promo.md)), then "About" (glyph E946). The built-in Settings item sits below them as WinUI places it. Nothing else goes in the footer.

### UI-3: Page layout and cards

- **Status:** Decided (D-019). Existing (2 of 6: display-switchr, tray-control-for-wsl)
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitor/App.xaml` (`PageContentStyle`, `PageTitleStyle`, `SectionHeaderStyle`, `CardBorderStyle`) and `tray-control-for-wsl/WslMonitor/Pages/GeneralPage.xaml`
- **Why:** Three generations of page styling coexist today, two of them inside single apps, and the shared promo page matches only the oldest. One named set of styles is the only thing that stops a fourth.

Every Windows app adopts the newest generation (D-019):

| Element | Value |
| --- | --- |
| Page padding | `36,12,36,36` |
| Content max width | 960 |
| Card | `CardBorderStyle`: corner radius 8, padding `16,14` |
| Section header | `BodyStrongTextBlockStyle` |
| Label capitalisation | sentence case ("App theme", "Check for updates") |

Pages use the named styles; they do not write padding, radius or header sizes inline. The styles are defined once and delivered through winui-app-kit's shared XAML (D-005, see [shared-kits.md](shared-kits.md)), so an app is restyled once, during its kit migration, with the promo page first. Shipped apps are restyled too (D-019); the change is visible to users, so each restyled app gets new store screenshots in the same release.

### UI-4: Settings page

- **Status:** Existing (6 of 6 for the title, the theme setting and immediate save). Labels are Decided (D-072) (2 of 6 use the wording)
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitor/Pages/GeneralPage.xaml` and `.xaml.cs`; winui-app-kit `Theming/AppTheme.cs`
- **Why:** These are the settings every app has, so they should read and behave identically.

- The page is titled "Settings" and is reached from the built-in Settings item.
- **Theme:** three choices (system, light, dark), stored as 0, 1, 2, and applied to every open window at once through the kit's `AppTheme` (applied to all windows in 2 of 6 today).
- **Startup:** resident apps have a startup switch; its behaviour is in [tray.md](tray.md) (TRAY-10).
- **Language:** localized apps have a language picker that follows Windows unless the user picks a language (LOC-4 in [locales.md](locales.md)). Its follow-the-system option uses the same wording as the theme's.
- **Saving:** every edit applies and saves immediately. There is no Save button.
- **Wording (D-072):** "Start with Windows" and "Use system setting", in sentence case, shipped as kit strings so they cannot drift.

Reset and export of settings are covered by CONF-4 in [settings-config.md](settings-config.md) (D-104).

### UI-5: About page

- **Status:** Decided (D-005, D-014, D-017, D-086). winui-app-kit ships the page from v1.1.0; 0 of 6 apps use it yet (each app's kit migration adopts it)
- **Applies to:** Windows
- **Reference:** `persistent/apps/desktop/Persistent.Desktop/Pages/AboutPage.xaml` for content; `tray-control-for-wsl/WslMonitor/Pages/AboutPage.xaml` for layout and for taking the version from the kit
- **Why:** About is where a user goes for the version, an update, help and the logs. Today no two apps offer the same set, and none offers a support link.

The items every About surface carries, on any stack, are listed once in BRAND-8 in [branding.md](branding.md). This rule owns how a Windows app lays them out, as these cards, in order:

| Card | Contents |
| --- | --- |
| Identity | App icon (the 256 px image, not a glyph), product name, version, one-line description |
| Updates | A "Check for updates" button. No app checks in the background. What the button does is in [distribution.md](distribution.md) |
| Links | Source code, privacy policy, support, licence and third-party notices; the target of each is in BRAND-8 |
| Diagnostics | "Open logs folder" (LOG-8 in [logging.md](logging.md)) |

- The version is the 4-part package version from the kit's `AppEnvironment.Version`, with "(unpackaged)" appended when the app is not running packaged (2 of 6).
- Links use the repo's current owner and the product's registered policy URL, never a hardcoded pre-move address; the known stale links are FIX-013 to FIX-016.
- Product-specific cards ("How it works", a technology list) may follow the standard cards. They do not replace them.

Windows apps use winui-app-kit's shared `AboutPage`, filled from an `AboutInfo` (see the kit's README, "Shared XAML"); an app never builds its own About page or restyles the standard cards. Product-specific cards go in `AboutInfo.ExtraCards`, after the standard ones.

### UI-6: Dialogs and status messages

- **Status:** Existing (confirmations are code-built `ContentDialog`s in every app that has one; destructive ones default to Close in 3 of the 4 apps checked). A shared helper is Proposed
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitor/DetailsWindow.xaml.cs` (`ConfirmAsync`)
- **Why:** A destructive dialog whose default button is the destructive one turns a stray Enter key into data loss. One helper makes the safe default the only default.

- Confirmations go through one `ConfirmAsync`-style helper per app, not a `ContentDialog` constructed at each call site (little-launcher has 18 constructions across 12 files). Apps on winui-app-kit use the kit's `ConfirmDialog.ConfirmAsync`, which always defaults to Close and never leaves a caller waiting if its window closes (see the kit's README).
- A destructive confirmation names the action on its primary button and sets the default button to Close.
- Standing status that the user must see (not connected, needs setup) is a non-closable `InfoBar` at the top of the Home page, as repilot and immich-drive do. Do not use a dialog for status. Operating-system notifications (toasts) are LOG-11 in [logging.md](logging.md).
- A second click on the same button as confirmation (tray-control-for-wsl's flyout) is allowed only inside a flyout, where a dialog would dismiss it.

## Web

### UI-7: Component library and written guide

- **Status:** Existing (4 of 4 on Joy UI at the same version; 3 of 4 on TanStack Query)
- **Applies to:** Web
- **Reference:** `printstream/apps/web/`; `printstream/.agents/guides/ui-conventions.md`
- **Why:** One component library is the strongest existing web convention, and it is what makes shared primitives possible at all.

Web apps use React with Joy UI as the component library and TanStack Query for server data. MUI Material is added only for components Joy lacks (charts, date pickers). The library versions are part of the web baseline in [platform.md](platform.md) (D-007), and all four apps move together.

Shared primitives are named the same in every app, so the same thing can be found and later extracted: `BackAwareModal`, `EmptyState`, `SectionHeading`, a toast provider with a `useToast` hook. printstream is the source for each; copy from there, keep the name, and see KIT-10 in [shared-kits.md](shared-kits.md) for when the copies become a package.

### UI-8: Theme engine

- **Status:** Existing (2 of 4 use the shared engine: printstream, game-is-up; persistent has its own theme set and store-sign-controller uses the default Joy theme)
- **Applies to:** Web
- **Reference:** `printstream/apps/web/src/theme/buildTheme.ts` with `AppThemeProvider.tsx`; `game-is-up/apps/web/src/theme/buildTheme.ts` for the adapted copy
- **Why:** The theme is where each product's look is defined, and two diverging engines make every shared component need two sets of tokens.

A web app builds its Joy theme through a `theme/buildTheme.ts` module adapted from printstream's, wrapped in an `AppThemeProvider`. Custom CSS variables carry a per-app prefix (`--printstream-*`, `--gu-*`). The user's theme choice offers a system option and is stored as a device preference (see CONF-7 in [settings-config.md](settings-config.md)). An app with no custom look may stay on the default Joy theme until it needs one; it does not write a third engine.

### UI-9: Dialogs and the Back-aware modal

- **Status:** Existing (3 of 4 close dialogs on Back: printstream, persistent, store-sign-controller; the close rule is written down and linted in 1 of 4)
- **Applies to:** Web
- **Reference:** `printstream/apps/web/src/components/BackAwareModal.tsx`; `printstream/eslint.config.mjs` for the lint rules
- **Why:** In an installed PWA or a Capacitor shell, Back is the user's "close this" gesture. A dialog that ignores it navigates the page away underneath and loses the user's place.

- Every dialog is built on `BackAwareModal` (or a wrapper of it), so browser and Android Back dismiss the top dialog instead of navigating. Do not use Joy's `Modal` directly, and do not write another Back handler; store-sign-controller's `AppDialog` is the third copy of this behaviour.
- A dialog closes only when the user aims at something that says it closes it. Scrim clicks do not close a dialog by default.
- Native `alert`, `confirm` and `prompt` are not used; a confirm or prompt goes through the app's dialog provider. This is enforced by lint where the rule exists, and is added to the shared ESLint config in [code-style.md](code-style.md).
- Destructive confirmations name the action on the confirm button and do not make it the default.

### UI-10: Installable web app (PWA)

- **Status:** Existing (3 of 4: printstream, persistent, game-is-up; store-sign-controller has a static manifest and no service worker)
- **Applies to:** Web
- **Reference:** `game-is-up/apps/web/vite.config.ts`; `printstream/apps/web/src/lib/appUpdate.ts`
- **Why:** The same web bundle is what the desktop and mobile shells load, so how it installs and updates decides whether users are ever on a stale version.

- The app is a PWA built with `vite-plugin-pwa` and `registerType: 'autoUpdate'`.
- Update handling lives in one module named `lib/appUpdate.ts` (2 of 3; persistent's is `swUpdate.ts`).
- The manifest provides separate `any` and `maskable` icons; one image declared as both is FIX-027. Icon sources are in [assets.md](assets.md).

## Android

### UI-11: Material 3 with dynamic colour

- **Status:** Existing (2 of 2 for Material 3 and dynamic colour; 2 of 2 keep all text in string resources)
- **Applies to:** Android
- **Reference:** `phonelock/companion/` for Compose; `sonarbridge/android/app/` (`Theme.Material3.DayNight.NoActionBar` plus `DynamicColors`) for Views
- **Why:** These outcomes do not depend on the toolkit, so both native apps can meet them and still look like Android apps first.

- Material 3, light and dark, with dynamic colour where the device supports it, applied through one theme wrapper (phonelock sets it in two places today).
- All user-facing text in string resources (see [locales.md](locales.md)), and an adaptive launcher icon (see [assets.md](assets.md)).
- Settings has an "About" or "This app" section showing the product name and version in every build flavour, with the privacy policy and source links (the full item list is BRAND-8 in [branding.md](branding.md)). Neither app has the full block today, and phonelock's Play flavour shows no version. There is no sibling-apps entry (D-075, see [promo.md](promo.md)).
- One dialog shape and one list-row shape per app, as phonelock's `AppDialog.kt` and `ListRow.kt` require.

### UI-12: Toolkit for new Android work

- **Status:** Decided (D-056). Existing (1 of 2 on Compose: phonelock; sonarbridge builds Views in code by an explicit rule in its `AGENTS.md`)
- **Applies to:** Android
- **Reference:** `phonelock/companion/`
- **Why:** A shared Android UI kit is only practical with one toolkit, and Compose is the platform direction.

New Android apps use Jetpack Compose. sonarbridge stays on programmatic Views and is not converted. There is no shared Android UI kit until a third native app exists; phonelock's primitives are the seed if one is needed.

## All stacks

### UI-13: Empty states

- **Status:** Proposed (a shared empty state exists in 1 of 6 Windows apps, tray-control-for-wsl, and 2 of 4 web apps, printstream and game-is-up)
- **Applies to:** Windows, Web, Android
- **Reference:** `printstream/apps/web/src/components/EmptyState.tsx`; `tray-control-for-wsl/WslMonitor/Controls/DistroList.cs`
- **Why:** An empty list with no explanation looks broken. Grey text in one place and a designed message in the next makes one product look like several.

Any surface that can have nothing to show uses the app's one empty-state component: an icon, a short title saying what is missing, one sentence on why or what to do, and an optional action. The same component is used on every surface that shows the same list (tray-control-for-wsl uses one builder for its flyout and its overview page). Web apps name the component `EmptyState` and keep it in `components/`, not inside a feature folder. An app does not fall back to a different page instead of showing an empty state.

### UI-14: Accessibility baseline

- **Status:** Proposed (no repo has an accessibility lint, an automated check or a written checklist; 1 of 6 Windows apps, little-launcher, handles high contrast)
- **Applies to:** Windows, Web, Android
- **Reference:** `little-launcher/LittleLauncher/` (25 `AutomationProperties` uses and high-contrast handling); `printstream/apps/web/` for `aria-*` and `prefers-reduced-motion` use
- **Why:** These are cheap to do while building a screen and expensive to retrofit, and store reviews increasingly check them.

| Stack | Minimum |
| --- | --- |
| Windows | Every icon-only control has an `AutomationProperties.Name`; every action is reachable by keyboard; the window is usable in high contrast and at 200% text scaling |
| Web | Every icon-only button has an accessible name; dialogs trap and restore focus; motion respects `prefers-reduced-motion`; `eslint-plugin-jsx-a11y` runs in lint |
| Android | Every icon-only control has a `contentDescription` (or is marked decorative); text uses scalable units |

Each stack gets one lint or check for this in [code-style.md](code-style.md) and [testing.md](testing.md) when those tools are chosen (D-016).

### UI-15: First run and change notices

- **Status:** Existing (first-run flag in 3 of 6 Windows apps: display-switchr, tray-control-for-wsl, little-launcher; "existing users only" notices in 2 products: little-launcher, persistent)
- **Applies to:** Windows, Web, Android
- **Reference:** `little-launcher/LittleLauncher/Classes/Settings/SettingsManager.cs` (`RaiseUpgradeNotices`); `display-switchr/HubDisplaySwitch/` for telling first run apart from a settings reset
- **Why:** Windows 11 hides new tray icons, so a resident app that starts silently looks like it did nothing. And a "what changed" notice shown to someone who never saw the old behaviour is noise.

- **First run opens the main surface once.** A Windows app opens its settings window (or its flyout, for an app whose flyout is the product) on first launch and never again automatically. First run is a stored flag (`IsFirstRun`), and "settings were reset" is reported separately from first run (see CONF-2 in [settings-config.md](settings-config.md)).
- **Setup before use.** An app that cannot work until configured opens on the page where that is done.
- **Change notices go only to users who had the previous version.** Fresh installs and new accounts never see them.
- **Permissions are explained before they are requested.** On Android, each permission is described before its system prompt or settings screen opens (`persistent/apps/web/src/native/permissions/PermissionOnboarding.tsx` is the reference).
