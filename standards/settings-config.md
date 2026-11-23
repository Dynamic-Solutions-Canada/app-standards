# Settings and configuration

Where each app keeps what the user chose (user settings) and what the developer or operator chose (app and build configuration), and how both are stored safely.

Counts are out of the 6 WinUI apps (displayswitchr, tray-control-for-wsl, little-launcher, repilot, immich-drive, and persistent's desktop app) or the 4 web monorepos (persistent, printstream, game-is-up, store-sign-controller) unless a rule says otherwise.

Related topics: what the Settings page looks like is UI-4 in [ui.md](ui.md); where credentials and the `.secrets/` folder live is in [security.md](security.md) (D-038, D-081); log files that share the data folder are in [logging.md](logging.md); runtime and SDK pins are in [platform.md](platform.md). The Windows settings store is provided by winui-app-kit and documented in [its README](https://github.com/Dynamic-Solutions-Canada/winui-app-kit); it is not re-described here.

## Overview

| Stack | User settings live in | Mechanism | Rules |
| --- | --- | --- | --- |
| Windows | `%AppData%\<App>\settings.json` | winui-app-kit `JsonSettingsStore<T>` | CONF-1 to CONF-5 |
| Electron and Linux hosts | `<userData>/settings.json` | one atomic state store per host | CONF-6 |
| Web | browser storage for device preferences; the database for account settings | one storage-keys module per app | CONF-7 |
| Android | `SharedPreferences` or DataStore | one typed wrapper per app | CONF-8 |
| HA | the Home Assistant config entry and options flow | Home Assistant | CONF-8 |
| Server | environment variables | one validated `lib/env.ts` | CONF-9 to CONF-11 |

## User settings

### CONF-1: Windows settings location and store

- **Status:** Decided (D-076) for the location (6 of 6 already use it). Existing (2 of 6: displayswitchr, tray-control-for-wsl) for the kit store
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitor/Classes/SettingsManager.cs` (40 lines over the kit); winui-app-kit `Settings/JsonSettingsStore.cs` and `AppEnvironment.cs`
- **Why:** One location and format means support instructions, backup advice and "open the data folder" work the same in every app. Four apps still carry a copied settings manager with a known data-loss bug (CONF-3).

- Settings are one file, `%AppData%\<DataFolderName>\settings.json`, in indented PascalCase JSON, holding one `UserSettings` class. Logs and caches live in the same folder (D-076: everything roaming, no local split). store-sign-controller's desktop host follows when it moves to the kit.
- The file is read and written only through the kit's `JsonSettingsStore<T>`; the app keeps a thin `SettingsManager` for its own defaults and migrations. The kit resolves the packaged-app redirect, so apps do not compute paths themselves.
- `DataFolderName` is defined once in code and is the name the log configuration uses too; it is not typed a second time in `NLog.config` (see [logging.md](logging.md)).
- The data folder name is permanent once users have data, and is recorded in the identifier registry (BRAND-10 in [branding.md](branding.md)).

### CONF-2: Atomic writes and unreadable files

- **Status:** Existing (2 of 6 Windows apps, through the kit; 2 of 3 Electron and Linux hosts write atomically)
- **Applies to:** Windows, Web (Electron and Linux hosts)
- **Reference:** winui-app-kit `Settings/JsonSettingsStore.cs`; `displayswitchr/HubDisplaySwitch/Classes/Settings/SettingsManager.cs` (`WasReset`)
- **Why:** A crash or power loss during a plain file write leaves a truncated settings file, and silently replacing it with defaults loses the user's setup without telling them.

- Every save writes a temporary file and renames it over the original. No app uses a direct `File.WriteAllText` on its settings file.
- A file that cannot be read is kept beside the original as `.bad`, defaults are loaded, the event is logged, and the user is told once that settings were reset. Because the store writes defaults straight away, the next launch looks normal, so the notice belongs to that same launch: a resident app opens its settings window on that launch (even when started silently at sign-in) and shows the notice at the top, using winui-app-kit's `SettingsResetNotice` and the `NeedsResetNotice` flag from v1.7.0. The notice names only the backup file's name, never a full path. This notice is separate from first run (UI-15 in [ui.md](ui.md)).
- Settings save as they change (UI-4); there is no deferred "save on exit".

### CONF-3: Schema version and written defaults

- **Status:** Decided (D-077). Existing (0 of 6 have an integer schema version; 2 of 6 write default values: displayswitchr, tray-control-for-wsl)
- **Applies to:** Windows, Kit
- **Reference:** `displayswitchr/HubDisplaySwitch/Classes/Settings/SettingsManager.cs` (`MigrateV1`, the `.v1.bak` copy) as the nearest example of one migration step
- **Why:** With defaults omitted from the file, a `false` saved for a setting that defaults to `true` loads back as `true`. little-launcher works around this per property; repilot, immich-drive and persistent's desktop app are exposed. And without a version, a migration has to guess the file's age from its shape.

- Every settings file carries an integer `SchemaVersion`. Migrations are an ordered list of functions, one per version step, run by the kit's store, and the old file is backed up before the first one runs. A file without a version is version 0, and the current version is the number of steps.
- The store reports how the file was loaded, and each case is handled the same way in every app:

| Case | Handling |
| --- | --- |
| First run | Defaults are written at once, so the next launch is an ordinary load |
| Migrated | Backed up as `settings.json.v<old>.bak` before the first step |
| Written by a newer app version (a downgrade) | Backed up as `settings.json.v<n>.bak`, loaded as far as this version understands, and not rewritten until the next save |
| Unreadable, or a migration step throws | Kept as `.bad` (and `.v<n>.bak` for a failed migration), defaults written, and the one-time "settings were reset" notice shown (CONF-2) |
| Reset | See CONF-4 |

The mechanics are the kit's `JsonSettingsStore` and are documented in its README; an existing app keeps its current behaviour until it opts into the versioned store.
- Default values are written to the file; only nulls are omitted. An app switches to written defaults before it next changes any default, because the switch is only safe while every omitted value still means the current default.
- An app version stamp such as little-launcher's `LastRunVersion` may be kept as well, for upgrade notices only (UI-15). It is not used to choose migrations.

### CONF-4: Reset and export

- **Status:** Decided (D-104). Proposed (reset: 0 of 6; export and import: 1 of 6, little-launcher)
- **Applies to:** Windows, Web
- **Reference:** `little-launcher/LittleLauncher/Pages/SystemPage.xaml` for export and import
- **Why:** "Delete the settings file" is the support answer today, and it requires the user to find a folder that packaged apps redirect.

- **Reset:** the Settings page offers "Reset settings". It renames the current file to `settings.json.<UTC timestamp>.reset` (so it can be recovered), writes defaults, and reports it as a reset, not a first run. It does not touch state Windows owns, such as the startup task (TRAY-10 in [tray.md](tray.md)), or stored credentials unless the user is told. Web apps offer the equivalent for device preferences ("reset this device").
- **Export and import** are offered where settings are content the user authored (launchers, mappings, layouts), not for apps with a handful of switches. An export never contains credentials.
- Account-level data export and deletion for web products is a separate question (D-025, see [security.md](security.md)).

### CONF-5: Credentials are never stored in the settings file

- **Status:** Existing (1 of the 2 Windows apps that hold a credential; the other is FIX-001). The kit helper is Proposed
- **Applies to:** Windows, Web, Android
- **Reference:** `little-launcher/LittleLauncher/Services/ProtectedStore.cs` (DPAPI, with the reasoning in its comments)
- **Why:** Settings files get attached to support requests, synced and exported. A token inside one leaks with it, and that cannot be patched afterwards.

A settings file, a preferences file or a settings export never holds an API key, token or password. Where each stack keeps a credential instead is owned by SEC-1 to SEC-3 in [security.md](security.md) (D-081) and is not restated here. The Windows protected-store class moves into winui-app-kit so the next app does not copy it (KIT-9 in [shared-kits.md](shared-kits.md)).

### CONF-6: Electron and Linux hosts

- **Status:** Existing (1 of 3 hosts has the full shape: printstream's desktop app; persistent's Linux desktop app writes atomically; little-launcher's Linux app was not examined)
- **Applies to:** Web (Electron and Linux hosts)
- **Reference:** `printstream/apps/desktop/electron/src/state-store.ts` and `app-profile.ts`; `persistent/apps/linux-desktop/src/settings.ts` for validating loaded settings
- **Why:** These hosts were written separately and each re-decided where state goes; printstream's is the only one that keeps a development build from writing into a production profile.

A host keeps its settings in one state store under the platform's user-data folder (XDG paths on Linux): written atomically (CONF-2), with file mode `0600`, validated against the current shape on load, and with a separate data folder per build channel (development, staging, production). Host logs sit beside it. The contract between a host and the web app it loads is PLAT-13 in [platform.md](platform.md) (D-057).

### CONF-7: Browser storage keys

- **Status:** Decided (D-105). Proposed (0 of the 3 web apps with device preferences follow one grammar; printstream alone uses four)
- **Applies to:** Web
- **Reference:** nearest: printstream's `DEVICE_*_OVERRIDE_KEY` constants in `printstream/apps/web/src`; `persistent/apps/web/src/settings/useSettings.tsx` for one settings module with a `defaults` object
- **Why:** Keys scattered through components cannot be listed, reset, migrated or checked for collisions, and a renamed product leaves its old prefix behind (printstream still reads `bambu.` keys).

- New keys are named `<app>.<area>.<name>`, with a `.v<N>` suffix added only when the stored shape changes.
- Every key is declared in one module per app, with its default, so "reset this device" (CONF-4) can enumerate them.
- Each setting is marked device-scoped (browser storage) or account-scoped (the database). An account setting with a per-device override, as printstream has, is declared as such in the same module.
- Existing keys are not renamed without a read-old, write-new migration; they hold users' current preferences.

### CONF-8: Android and Home Assistant settings

- **Status:** Proposed for Android (0 of 2 have a typed wrapper; 1 of 2 sets `allowBackup` explicitly: phonelock). Existing (1 of 1) for the versioned Home Assistant config entry
- **Applies to:** Android, HA
- **Reference:** `phonelock/companion/src/main/AndroidManifest.xml` for explicit backup; `printstream/integrations/home-assistant/custom_components/printstream/config_flow.py`
- **Why:** Preference keys typed at each call site drift the same way browser keys do, and an unset `allowBackup` means the platform default decides whether settings leave the device.

- **Android:** preferences are read and written through one typed wrapper per app that owns every key name and default (DataStore for new code; existing `SharedPreferences` files stay). The backup setting is stated explicitly in every manifest, as SEC-2 in [security.md](security.md) requires.
- **Home Assistant:** settings live in the config entry and options flow, with a `VERSION` and a migration when it changes. A diagnostics download with secrets redacted and a re-authentication step are part of the standard and are missing today; both are fixed in the printstream source.

## App and build configuration

### CONF-9: Server environment is read in one validated module

- **Status:** Existing (4 of 4)
- **Applies to:** Web
- **Reference:** `persistent/apps/api/src/lib/env.ts` (helpers such as `blankToUndefined`); `store-sign-controller/apps/api/src/lib/env.ts` (production guards)
- **Why:** A missing or malformed variable should stop the server at start with a clear message, not surface as a wrong behaviour in production.

- `apps/api/src/lib/env.ts` parses `process.env` once with a zod schema and exports a typed object. No other non-test file reads `process.env` (printstream has 8 that do).
- The schema refuses unsafe production combinations. Which ones is SEC-6 in [security.md](security.md), and the demo and reviewer-access pattern is SEC-14 there (D-083).
- Feature flags are environment booleans. There is no runtime flag service.
- A product with more than a handful of variables keeps the full list in `docs/configuration.md` (see [docs.md](docs.md)); printstream's is the reference.

### CONF-10: Env example files and shared variable names

- **Status:** Decided (D-078). Existing (1 of 4 names examples by role: printstream; 2 of 4 use a single `.env.example`: persistent, store-sign-controller)
- **Applies to:** Web, Android (for a server component)
- **Reference:** `printstream/.env.server.example` (by role); `persistent/.env.example` (single file with a "which context needs what" header)
- **Why:** The example file is the only documentation most variables get, and three names for the same Android origin setting means shared code and shared docs cannot refer to it.

| Case | Files |
| --- | --- |
| One audience (a single container the maintainer deploys) | `.env.example` |
| More than one audience (server operator, bridge, developer) | `.env.<role>.example`, one per audience, with the role matching the compose file name (`dev`, `server`, `bridge`) |

- Every variable in an example has a comment, placeholder values are never real hostnames or keys (see FIX-007), and a required secret has no working default (see FIX-006).
- Names for the same thing are the same in every app: `DATABASE_URL`, `CLIENT_ORIGIN`, `API_PORT`, `CLOUDFLARE_EMAIL_*`, `FCM_*`, `DEPLOY_*`, and `ANDROID_APP_ORIGINS` for the Android origin setting. Booleans end in `_ENABLED`. A setting with more than two states is one `_MODE` variable with named values (`BRIDGE_UPDATE_MODE=automatic|manual|pinned`), not several booleans that can contradict each other. The shared list is kept in this file (D-078).
- Renaming a variable that deployed servers already set is done with a fallback to the old name for one release, because the rename is cheap in the repo and costly on the server. Software that customers run themselves keeps reading the old name indefinitely and logs a deprecation notice when it is used, because their configuration is never upgraded for them; where the software writes its own configuration file, it rewrites the old name to the new one on its next start (decided 2026-11-23, PrintStream's bridge).
- Real values live in `.env` and `.secrets/`, both ignored; see [security.md](security.md) and [repo-hygiene.md](repo-hygiene.md).

### CONF-11: Compose file names and production topology

- **Status:** Decided (D-079) for production (2 of 4 use the name: persistent, store-sign-controller; 3 of 4 ship a single image). Existing (4 of 4) for `compose.dev.yml`
- **Applies to:** Web
- **Reference:** `persistent/compose.server.yml` and its nginx site file; `game-is-up/compose.dev.yml` (no project-specific additions); `printstream/compose.server.example.yml` for a self-hosted product
- **Why:** A tracked production file is reviewable and is what is actually running. An example file that is copied and edited on the server drifts from the repo by design, which is right only when someone else operates it.

| File | Use |
| --- | --- |
| `compose.dev.yml` | Development: one `node` service built from `docker/dev/Dockerfile`; devkit adds the database (see [dev-scripts.md](dev-scripts.md) and devkit's README) |
| `compose.server.yml` | Production for products the maintainer hosts: tracked and used directly (D-079) |
| `compose.server.example.yml` | Production for self-hosted products only (printstream): copied to an ignored `compose.yml` by the operator |
| `nginx.server.conf` | The site file for products the maintainer hosts: tracked with its real public host names and installed as is by the deploy (CTR-8, CTR-11). Public host names are not secret; server addresses and paths still never appear (SEC-5) |
| `nginx.server.example.conf` | Reverse-proxy example for self-hosted products only (printstream), with placeholder hostnames |

- The root `compose.yml` name is reserved for an ignored deployment-local copy and is never tracked.
- That production is one image serving the API and the web app is DIST-12 in [distribution.md](distribution.md).
- What a production compose file must contain is owned by other rules and not restated here: required secrets with no default are SEC-5 and the non-root user, health check and named volumes are SEC-7 (both in [security.md](security.md)), and log size caps are LOG-4 in [logging.md](logging.md). The known gaps are FIX-006 and FIX-008.

### CONF-12: Build and project configuration files

- **Status:** Existing (6 of 6 Windows apps have `Directory.Build.props` and `windows-build.json`; 5 of 6 have `global.json`; 3 of 4 web monorepos have `devkit.config.mjs`)
- **Applies to:** Windows, Web, Android, Kit
- **Reference:** `tray-control-for-wsl/windows-build.json`, `tray-control-for-wsl/Directory.Build.props`; `persistent/devkit.config.mjs`
- **Why:** Build inputs in the same named files mean a script, an agent or a shared build tool can find them without knowing the repo.

| Stack | File | Holds |
| --- | --- | --- |
| Windows | `Directory.Build.props` | The version (the only place it is written; see [releases.md](releases.md)) and shared build properties |
| Windows | `global.json` | The .NET SDK pin (D-053, see [platform.md](platform.md)) |
| Windows | `windows-build.json` | Per-app values for the shared build tooling and the kit's scripts: project, identities, signing mode (D-065, see [distribution.md](distribution.md)), the packaging settings read by the kit's `build-msix.ps1`, and the `localization` block read by the shared `check:l10n` (see winui-app-kit's README) |
| Web | `devkit.config.mjs` | Ports, database, worktree files and env mapping for devkit, in the house shape |
| Android | `gradle.properties`, the version catalog | Build settings and dependency versions (D-055) |

- Configuration that is the same in every app belongs in the shared kit, not in six copies (KIT-9 and KIT-10 in [shared-kits.md](shared-kits.md)).
- Machine-specific values stay out of tracked files: no absolute host paths (FIX-049), and machine state lives in the tool's own config folder, as devkit's `host.json` does.
- Build-time secrets come from an ignored file or CI secrets, with a tracked `.example` showing the shape; one Windows app's build-time secrets file has no example today.
