# Logging, diagnostics and privacy disclosure

Where each app writes its logs, what goes in them, how a user gets them to the maintainer, what happens when an app crashes, and the no-telemetry rule. The privacy policy itself is DOCS-8 in [docs.md](docs.md). Notifications are covered at the end because no other topic owns them; in-app status surfaces such as `InfoBar` and dialogs belong to [ui.md](ui.md).

Counts are against the in-scope repos in [SCOPE.md](../SCOPE.md). "WinUI apps" means the six that share the family shape (hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, and persistent's desktop app). "Web APIs" means the four monorepo APIs (persistent, printstream, game-is-up, store-sign-controller). "End-user apps" means the 11 products listed in [readme.md](readme.md).

## Overview

| Stack | Where logs go | Reference |
| --- | --- | --- |
| Windows | One text file per launch in the app's data folder, written by NLog (LOG-1, LOG-2) | `wsl-monitor/WslMonitor/NLog.config` |
| Web API | Standard output through one `logger` module, captured by Docker with a size cap (LOG-3, LOG-4) | `printstream/apps/api/src/lib/logs.ts` |
| Electron and Linux hosts | One capped `host.log` in the host's data folder (LOG-5) | `persistent/apps/linux-desktop` |
| Android | The platform log only; no log file (LOG-5) | `phonelock` |
| HA | One module-level `_LOGGER` per file, into Home Assistant's own log (LOG-5) | `printstream/integrations/home-assistant/custom_components/printstream/` |

## Rules

### LOG-1: Windows log location and file name

- **Status:** Decided (D-080, D-076). Existing (6 of 6 Windows hosts; store-sign-controller dropped its desktop host when it became web only)
- **Applies to:** Windows
- **Reference:** `wsl-monitor/WslMonitor/NLog.config`
- **Why:** A user asked for "the log from when it broke" can find one file per launch by its timestamp, in the same place in every app.

Logs are written to `%AppData%\<DataFolderName>\logs.<start time>.txt`, one file per launch, named by the process start time. `<DataFolderName>` is the same folder that holds `settings.json`; CONF-1 in [settings-config.md](settings-config.md) defines it once for both the settings store and the logging config. A hand-appended log file, a second folder, or a vendor-named folder is not allowed.

### LOG-2: One shared NLog config, with levels and retention

- **Status:** Decided (D-080). Existing (6 of 6 WinUI apps share the shape, in two variants: hub-display-switch and wsl-monitor set an explicit layout line, the other four do not)
- **Applies to:** Windows, Kit
- **Reference:** `wsl-monitor/WslMonitor/NLog.config` (the hub-display-switch and wsl-monitor variant, with the explicit layout line)
- **Why:** Six copies of one file have already drifted, and nobody has confirmed that old log files are ever removed.

The config comes from winui-app-kit (see [shared-kits.md](shared-kits.md)), not from a per-app copy. It sets:

| Setting | Value |
| --- | --- |
| File target minimum level | Info |
| Debugger target minimum level | Trace |
| Layout | Explicit: timestamp, upper-case level, short logger name, message with exception |
| Retention | The current file plus three older ones |

Retention must be verified on a long-lived install before this rule is marked existing: the current config names three archives but sets no archive trigger, so whether old per-launch files are pruned is unconfirmed. If they are not, the kit prunes them at startup.

Use levels the same way in every stack:

| Level | Use for |
| --- | --- |
| Error | An operation failed and the user or operator is affected |
| Warn | Something unexpected that the app recovered from |
| Info | Lifecycle and user-meaningful events: start, exit, settings changed, update found, connection made or lost |
| Debug and Trace | Developer detail; never written to the file target in a release build |

### LOG-3: Web APIs log through one logger module

- **Status:** Decided (D-082). Existing (1 of 4 web APIs meets the target: printstream; persistent meets the minimum; game-is-up and store-sign-controller call `console.*` directly)
- **Applies to:** Web
- **Reference:** `printstream/apps/api/src/lib/logs.ts` (target); `persistent/apps/api/src/lib/logger.ts` (minimum)
- **Why:** Reading logs and tracing one request should work the same way on every product the maintainer operates.

Route and service code never calls `console.*`. It calls a `logger` module in `apps/api/src/lib/` that writes to standard output. The minimum is persistent's shape: a level and a message followed by structured JSON context. The target is printstream's: every line carries a request correlation id, and the id is returned to the client so a user report can be matched to a log line. The wider service skeleton (health routes, error shape) is PLAT-15 in [platform.md](platform.md).

### LOG-4: Container logs are capped

- **Status:** Existing (1 of 4 web products: printstream)
- **Applies to:** Web
- **Reference:** `printstream/compose.server.example.yml` (the shared logging anchor)
- **Why:** An uncapped container log fills the server's disk, and the failure shows up as an unrelated outage.

Every service in a production compose file sets Docker log options with a maximum file size and file count, declared once as a shared anchor and applied to each service. Compose file naming is in [settings-config.md](settings-config.md).

### LOG-5: Logs on other hosts

- **Status:** Proposed (the pieces exist in one repo each)
- **Applies to:** Web (Electron and Linux hosts), Android, HA
- **Reference:** `persistent/apps/linux-desktop` (`host.log`); `printstream/integrations/home-assistant/custom_components/printstream/`
- **Why:** These hosts are few, so each has invented its own answer; writing the answer down stops a fourth variant.

- **Electron and Linux hosts:** one `host.log` under the host's user-data folder, readable only by the user, rolled to a single previous file when it passes a size cap.
- **Android:** the platform log, with a tag per class. No log file is written and nothing is uploaded. Verbose and debug calls must not carry user content, because other tools on a device can read the platform log.
- **HA:** one module-level `_LOGGER` per Python file, so Home Assistant's own log viewer and per-integration log levels work.
- **Dev tooling:** devkit's own log location is documented in its README and is not restated here.

### LOG-6: What must not be logged

- **Status:** Proposed
- **Applies to:** all
- **Why:** Logs are the one piece of app data users are asked to send to someone else, so anything in them should be safe to attach to a support request.

Never log credentials, tokens, session identifiers, full request or response bodies, or the user's content (file contents, message text, photos, locations). Log the identifier of a thing, not the thing. When a URL or header may carry a credential, log it with the credential removed. Credential storage itself is in [security.md](security.md).

### LOG-7: Unhandled errors are always logged

- **Status:** Existing (6 of 6 WinUI apps; 1 of 4 web APIs: printstream; 2 of 4 web clients: persistent, printstream)
- **Applies to:** Windows, Web
- **Reference:** `wsl-monitor/WslMonitor/App.xaml.cs` (Windows); `printstream/apps/api/src/lib/process-crash-handlers.ts` (API); `persistent/apps/web/src/components/ErrorBoundary.tsx` (web)
- **Why:** A crash that leaves no trace cannot be diagnosed, and with no crash reporting service (LOG-9) the log is the only record.

| Stack | Required |
| --- | --- |
| Windows | Handlers for both the app-domain and the WinUI unhandled-exception events that log the exception and flush the logger. The pair is identical in all six apps and belongs in winui-app-kit. |
| Web API | Process-level handlers for uncaught exceptions and unhandled rejections that log and then exit with a non-zero code, so the container restarts cleanly. |
| Web client | A top-level error boundary that shows a recoverable error screen instead of a blank page. |
| Android | No requirement; the platform records the crash. |

### LOG-8: How a user exports diagnostics

- **Status:** Decided (D-023, D-086). Existing (3 of 6 WinUI apps have "Open logs folder": hub-display-switch, wsl-monitor, persistent desktop; 0 of 6 have a support link)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/apps/desktop/Persistent.Desktop/Pages/AboutPage.xaml` (About page content); wsl-monitor (the one-line kit call); printstream's Logs tab and help and feedback dialog (web)
- **Why:** With no telemetry, the only way the maintainer sees a problem is when the user can find the log and knows where to send it.

- **Windows:** "Open logs folder" opens the log folder in Explorer. It appears on the About page beside the support link, and in the tray menu of a resident app. This rule owns what the action does; where it sits is owned elsewhere and not restated: the About items are BRAND-8 in [branding.md](branding.md), the About page layout is UI-5 in [ui.md](ui.md), and the tray menu position is TRAY-5 in [tray.md](tray.md).
- **Web:** a self-hosted product exposes recent server logs to its administrators inside the app, as printstream does. A product the maintainer hosts needs only a support link, since the maintainer reads the server logs directly.
- **Android:** a support link in Settings, next to the privacy link (BRAND-8). No in-app log export is required.

Where the support link goes is decided once, in BRAND-8 (D-086), and is the same kind of target in every app.

### LOG-9: No telemetry and no hosted crash reporting

- **Status:** Decided (D-023). Existing (15 of 15 repos contain no crash reporting or product telemetry; 1 of 11 end-user apps states it: phonelock)
- **Applies to:** all
- **Reference:** `phonelock/PRIVACY.md` ("The short version")
- **Why:** Every existing privacy statement already promises this, and trust lost by adding tracking later is expensive to win back.

Apps do not include analytics, tracking, or crash-reporting SDKs, and do not send usage data anywhere. Every README and privacy policy says so in one line; where that line is written is DOCS-9 in [docs.md](docs.md). Two narrow exceptions exist today and stay allowed if they are disclosed:

- Page analytics on a product's public marketing site, never inside the app.
- Operational metrics on a self-hosted server that are off by default and enabled by the operator.

Adding any reporting that leaves the user's device or server is a new decision for the maintainer, not an implementation choice.

### LOG-10: Privacy policy and disclosure

- **Status:** Retired. Merged into DOCS-8 in [docs.md](docs.md), which owns the privacy policy: its template, where it is published, what links to it, and the list of network calls it names.

### LOG-11: Notifications

- **Status:** Existing (2 of 2 Android apps follow the channel naming); Windows part Proposed
- **Applies to:** Android, Windows
- **Reference:** `phonelock` and `sonarphone-nmea-bridge` (channels); `persistent/apps/desktop/Persistent.Desktop/Notifications/ToastNotifier.cs` (Windows toasts)
- **Why:** Channel ids are permanent once users have set preferences on them, and toast code is the next thing likely to be copied between Windows apps.

- **Android:** a channel id is one short lower-case word; the channel name is a localized string that is refreshed when the language changes; there is one channel per category a user would want to mute separately. A published channel id is never renamed.
- **Windows:** an app that needs operating-system notifications starts from persistent's `ToastNotifier`; when a second app needs it, it moves into winui-app-kit rather than being copied.
- **Server push:** provider choice (FCM, web push, ntfy) is per product and is not standardised here; its credentials follow [security.md](security.md).
