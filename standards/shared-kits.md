# Shared kits

Which shared projects exist, what each owns, when an app must use them, and what copied code is due to move into one.

The three kits own their own usage docs. These standards say when to use a kit and link to its README; they do not repeat its API or setup steps.

| Kit | What it is | Canonical docs | Local copy |
| --- | --- | --- | --- |
| winui-app-kit | Shared plumbing for WinUI tray and desktop apps | [README](https://github.com/Dynamic-Solutions-Canada/winui-app-kit) | `wsl-monitor/external/winui-app-kit/README.md` |
| apps-promo | The "Our other apps" catalog and page | [README](https://github.com/Dynamic-Solutions-Canada/apps-promo) | `wsl-monitor/external/promo/README.md` |
| devkit | Per-checkout dev hostnames, ports and databases for web apps | README in the devkit repo, on the maintainer's personal account until it moves to the `Dynamic-Solutions-Canada` organisation under D-009 | `devkit/README.md` |

Counts are out of the 6 WinUI apps (hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, and persistent's desktop app) or the 4 web monorepos (persistent, printstream, game-is-up, store-sign-controller) unless a rule says otherwise. The kits are themselves held to these standards (READMEs, docs, AI docs, licence).

## Rules

### KIT-1: What each kit owns

- **Status:** Existing (3 of 3 kits have a defined scope in their README)
- **Applies to:** Kit, Windows, Web
- **Reference:** each kit's README, linked above
- **Why:** A kit with a clear boundary is one an app can adopt without reading its source, and a clear boundary is what keeps app behaviour out of it.

| Kit | Owns | Does not own |
| --- | --- | --- |
| winui-app-kit | Tray icon and menu, tray flyout, window chrome and placement, the JSON settings store, start with Windows, theme switching, localization mechanism, app environment (packaged or not, version, data folders), single instance | Any app's pages, settings fields, strings, or feature behaviour |
| apps-promo | The catalog data (`ids.json`, the generated manifest and icons), `refresh.ps1`, the WinUI page and its localization, `catalog.ts` for web consumers | Where the page sits in an app (see [promo.md](promo.md)); other apps' listings |
| devkit | A hostname, port and database per checkout and worktree, the routing proxy, `devkit.config.mjs`, the `devkit` commands, development data snapshots | Production deployment (D-045 lets it supply the deploy script as a template, nothing more), CI, the app's own dev processes |

What the kits should additionally own is KIT-9 and KIT-10. The rules that depend on each kit are in [tray.md](tray.md), [ui.md](ui.md), [settings-config.md](settings-config.md), [promo.md](promo.md) and [dev-scripts.md](dev-scripts.md).

### KIT-2: WinUI apps are built on winui-app-kit

- **Status:** Existing (2 of 6: hub-display-switch, wsl-monitor)
- **Applies to:** Windows
- **Reference:** `wsl-monitor/` for a new app; hub-display-switch commit `af41968` for migrating an existing one
- **Why:** About 2,500 lines across the other four apps duplicate what the kit already owns, and the copies are where the known defects are (non-atomic settings writes, no tray GUID, startup that can override the user).

- A new WinUI app starts on the kit and uses every kit type that applies to it. It does not copy the equivalent class from a sibling app.
- An existing app migrates as a planned piece of work, one app at a time. immich-drive is the most direct next migration. It keeps Windows 10 22H2 as its floor (D-018, PLAT-6 in [platform.md](platform.md)), and the kit's own floor is Windows 10 22H2 for that reason, so it migrates fully; a kit feature that needs Windows 11 checks the OS at runtime, and the kit's README names it.
- A migration keeps the app's permanent identifiers unchanged: tray GUID, mutex name, startup task id, data folder name (see TRAY-3, TRAY-9, TRAY-10 and CONF-1).
- An app may adopt the kit in parts when one part does not fit yet. little-launcher can take theme, window chrome, settings store, startup and single instance now; its tray waits for multi-icon support in the kit.
- Web-view hosts of web products follow when they are next reworked; the host contract is D-057.

### KIT-3: WinUI apps include apps-promo

- **Status:** Existing (6 of 6)
- **Applies to:** Windows
- **Reference:** [promo.md](promo.md)
- **Why:** See PROMO-1.

Every WinUI app with a settings window includes the promo page. Path, URL, pin, placement and catalog rules are PROMO-1 to PROMO-9 and are not repeated here.

### KIT-4: Web monorepos use devkit for development

- **Status:** Decided (D-007). Existing (3 of 4: persistent, printstream, game-is-up, all on the same tag)
- **Applies to:** Web
- **Reference:** `persistent/devkit.config.mjs` for a typical app; `printstream/devkit.config.mjs` for checks, cleanup and baselines
- **Why:** Without it each checkout needs fixed ports and its own database setup, so two worktrees of one app cannot run at once. store-sign-controller shows the cost: its own Postgres service, fixed ports, and an install on every container start.

A web monorepo gets its development hostname, ports and database from devkit, configured in `devkit.config.mjs` in the house shape. store-sign-controller adopts devkit (D-007). Setup and remote-development instructions are devkit's README; apps link to it and do not keep their own copy (README-4 in [readme.md](readme.md)).

### KIT-5: How shared code reaches an app

- **Status:** Decided (D-005). Existing (submodule under `external/`: winui-app-kit 2 of 6, apps-promo 6 of 6; tag-pinned package: devkit 3 of 4)
- **Applies to:** Windows, Web, Kit
- **Reference:** `wsl-monitor/.gitmodules`; `external/promo/PromotedApps.projitems` for shared XAML; `persistent/package.json` for the devkit pin
- **Why:** One mechanism per stack means an agent or a CI job knows where shared code is and how to update it. Copying is the mechanism that produced six diverged build scripts.

| Stack | Mechanism |
| --- | --- |
| Windows code | A git submodule under `external/<kit>` with a project reference |
| Windows XAML, build props and packaging scripts | The same submodule: XAML through a shared-items import (as the promo page already ships to all six apps), `.props` and `.targets` by import, and one packaging script driven by `windows-build.json` |
| Web tooling | devkit as a git-tag-pinned npm dependency |
| Web UI components | A git-tag-pinned npm package for primitives that have reached a third copy (D-005: a new `web-app-kit` repo); the rest stay copied from printstream under the same names (KIT-10) |

- Shared build configuration arrives through three imports: the app's `Directory.Build.props`, `Directory.Build.targets` and `Directory.Packages.props` each import the matching file from the kit's `build/` folder. The kit's README documents the files and their switches; apps do not restate them.
- Shared code is never vendored by hand into an app's own source folders.
- The build tooling the Windows apps call (windows-build-tools) has no remote today and is referenced by an absolute host path (FIX-049). It gets a GitHub repo of its own (D-005).
- The kit README's "No XAML (yet)" rule is replaced when this decision lands.
- Shared agent skills and shared CI workflows are delivered separately; see [ai-docs.md](ai-docs.md) (D-029) and [ci.md](ci.md) (D-047). This repo itself reaches apps by link (D-002).

### KIT-6: Versioning and pinning

- **Status:** Decided (D-005). Existing (1 of 3 kits is tagged: devkit; winui-app-kit and apps-promo are pinned by bare commit)
- **Applies to:** Kit, Windows, Web
- **Reference:** devkit's `vX.Y.Z` tags and the `#v0.16.2` pins in its three consumers
- **Why:** A bare commit says nothing about what changed or whether it is safe to take, and without tags the six promo consumers ended up on four different commits.

- Every kit is tagged `vX.Y.Z` with SemVer meaning: a breaking change is a major version. Tag and release-note rules are in [releases.md](releases.md).
- Consumers pin a tag. All consumers of a kit are on the same tag; a consumer that falls behind catches up in its next release at the latest. devkit's consumers agree with each other today but trail devkit's newest tag.
- A breaking kit change is made together with the consumer updates, in the same pass, as the winui-app-kit README already requires.
- The commit that bumps a pin has the subject `Update <Kit name>: <what it changes for this app>` ("Update WinUI App Kit: ..."), which is the existing practice. This rule owns that wording; HYG-5 in [repo-hygiene.md](repo-hygiene.md) links here.
- Kit dependency versions (Windows App SDK, NLog, .NET SDK, Node) follow the family pins in [platform.md](platform.md), so a kit never forces an app onto a different version.

### KIT-7: The third copy moves into a kit

- **Status:** Existing as the conventions rule in the user-level agent rules; applied across apps in 2 of 6 WinUI apps
- **Applies to:** all
- **Reference:** winui-app-kit's localization classes, extracted from hub-display-switch's copy
- **Why:** Each copy is small and each drifts. Three hand-written versions of the same packaging fix appeared in three apps in one day.

The rule "when the same shape appears a third time, extract it" from the user-level agent rules (kept outside this repo) applies across repos, not only inside one:

- Before writing plumbing in an app, check the kit and then the sibling apps. If the kit has it, use it. If the kit nearly fits, improve the kit (KIT-8).
- When the same code exists in two apps, the second copy keeps the same names and shape as the first, so it can be extracted cleanly.
- When a third app needs it, it goes into the kit that owns that stack, and all three apps take it from there. The third copy is not written.
- An extraction is named for what it does, not for the app it came from.

KIT-9 and KIT-10 list what is already at or past the third copy.

### KIT-8: Kits stay domain-neutral, and fixes land in the kit

- **Status:** Existing (3 of 3 kits; winui-app-kit states it in its README, and its code no longer names any app)
- **Applies to:** Kit
- **Reference:** "Rules for changes" in the winui-app-kit README
- **Why:** A kit that knows about one app's settings or wording cannot be adopted by the next app without a fork.

- A kit holds no app names, settings fields, strings or feature behaviour. When an app needs different behaviour, the kit gains a general option, event or hook and the policy stays in the app.
- A bug found in kit code is fixed in the kit and taken by a pin bump. It is not patched in the app, and the submodule's working tree is not edited from inside a consuming repo without the change being committed to the kit.
- A kit's README is the canonical usage doc (README-8 in [readme.md](readme.md)). App docs and these standards link to it. Kit `AGENTS.md` files hold no host paths and no list of consumers that can go stale (FIX-035).

### KIT-9: Windows code due to move into a kit

- **Status:** Decided (D-005, and the decisions named per row)
- **Applies to:** Windows, Kit
- **Reference:** the "best copy" column
- **Why:** These are the items already copied three or more times, or copied twice with a defect in one copy.

| Code | Copies today | Destination | Best copy | Depends on |
| --- | --- | --- | --- | --- |
| Tray, startup, theme, settings store, window chrome, single instance, localization classes | 4 apps still carry their own (about 600 to 800 lines each) | winui-app-kit (exists; apps migrate, KIT-2) | the kit | none |
| Update service (Store when packaged, GitHub releases when not) | 4: little-launcher, repilot, immich-drive, persistent | winui-app-kit | `repilot/Repilot/Services/UpdateService.cs` | D-014 |
| Settings window shell, About page, card and page styles | 6 | winui-app-kit shared XAML | `wsl-monitor/WslMonitor/` | D-019 |
| Confirm dialog helper (in the kit from v1.0.0: `ConfirmDialog`) | 5 apps | winui-app-kit | `wsl-monitor/WslMonitor/DetailsWindow.xaml.cs` | none |
| App bootstrap (two unhandled-exception handlers, restore settings, create host) and the hidden host window setup (in the kit from v1.0.0: `AppBootstrap`, `HostWindow`) | 6 and 5 | winui-app-kit | `wsl-monitor/WslMonitor/App.xaml.cs` | D-074 |
| Tray icon rasteriser (supersample and shrink) (in the kit from v1.0.0: `TrayIconRasterizer`) | 2: hub-display-switch, wsl-monitor | winui-app-kit | `wsl-monitor/WslMonitor/Classes/TrayIconRenderer.cs` | D-071 |
| DPAPI protected store | 2: little-launcher, immich-drive (FIX-001) | winui-app-kit, accepting an app-supplied folder, file name and entropy (both apps' values are permanent once shipped), with no logging dependency so a second process such as a shell extension can link it | `immich-drive/ImmichDrive/Services/ProtectedStore.cs` (adds `Exists`, an injectable protector, folder and error hooks, atomic save, and no delete on a failed load because two processes read the file) | D-081 |
| Settings schema versioning, backup and reset | 0 (new) | winui-app-kit | none | D-077, D-104 |
| `NLog.config` | 6, in two variants | winui-app-kit | `wsl-monitor/WslMonitor/NLog.config` | D-080 |
| Runtime CVE guard (`Directory.Build.targets`) and `global.json` | 3 guards, in three implementations | winui-app-kit | repilot's | D-053 |
| `Directory.Build.props` shared block, `.gitignore` tail, `scripts/windows-build` wrapper | 6 | winui-app-kit or the build tooling | wsl-monitor's | none |
| `build-msix.ps1` | 6 (1,716 lines in total) | one script driven by `windows-build.json` | `wsl-monitor/WslMonitorMSIX/build-msix.ps1` | D-065 |
| MSIX image generator | 5 | devkit's icon generator | none | D-069 |
| Language list (the nine locales and their aliases) and the catalog completeness check | 5 and 4 | one shared data file and one check | wsl-monitor's | D-067 |
| Promo page localization adapter | 3: wsl-monitor, repilot, persistent | already in apps-promo; delete the copies (PROMO-7) | apps-promo | none |

Kit features that block a migration are added first: several tray icons per host window (little-launcher), re-pointing the Run key for portable builds and an orderly-exit hook (persistent).

### KIT-10: Web code due to move into a kit

- **Status:** Decided (D-045, D-050, D-005)
- **Applies to:** Web, Kit
- **Reference:** the "best copy" column
- **Why:** Each devkit consumer hand-copies about 100 lines of glue that devkit could generate, and the web UI primitives have reached their third copy with no kit to receive them.

| Code | Copies today | Destination | Best copy | Depends on |
| --- | --- | --- | --- | --- |
| `scripts/dev/run-dev.mjs`, the `compose.dev.yml` `node` service, `paseo.json`, the `dev:*` scripts | 3 | devkit, as generated templates, not a runtime dependency; printstream's runner may stay custom | `game-is-up/` | D-045 |
| `wait-for-db.mjs`, the Vite `server` block | 3 | devkit helpers | `persistent/` | D-045 |
| Test discovery script for `node --test` | 2 scripts and 2 hand-written globs | devkit | `persistent/scripts/dev/run-tests.mjs` | D-050 |
| "Remote development with Devkit" README section and `docs/devkit-remote.md` | 3, drifted | deleted; link to devkit's README | devkit's README | none |
| `deploy-over-ssh.mjs` | 3 implementations | devkit, as a generated template written into `scripts/deploy/` | `persistent/scripts/deploy/deploy-over-ssh.mjs` | D-045 |
| Back-aware modal | 3: printstream, persistent, store-sign-controller (`AppDialog`); game-is-up has a narrower handler in one modal | a shared web UI package (D-005: a new `web-app-kit` repo, pinned by tag) | `printstream/apps/web/src/components/BackAwareModal.tsx` | D-005, D-007 |
| Theme engine, `EmptyState`, PWA update helper | 2 each | stay copied under the same names (UI-7 to UI-10 in [ui.md](ui.md)) | printstream's | none |
| SVG-to-icon generator | 1 (game-is-up), plus 5 PowerShell drawing scripts | devkit | game-is-up's | D-069 |

The Back-aware modal is at three copies, so D-005 gives shared web UI a package of its own, `web-app-kit`. It does not go into devkit, which is development tooling and must not become a runtime dependency. Until the package exists, copies keep printstream's names and behaviour, and the primitives still at two copies stay copied until each reaches a third.

### KIT-11: What is deliberately not a kit

- **Status:** Decided (D-056, D-057). Existing (no shared Android or host-shell code in any repo)
- **Applies to:** Android, Web
- **Reference:** none
- **Why:** A kit has a running cost: every consumer must be updated together. It is only worth it once the third consumer exists and the consumers share a toolkit.

- **Android:** no shared UI kit until a third native app exists (D-056); the two native apps use different toolkits by design. Play release tooling in sonarphone-nmea-bridge is the first candidate when a second app needs it.
- **Native host shells** (WebView2, Electron, Capacitor) are not merged into one codebase; the contract between host and web app is written down instead (PLAT-13 in [platform.md](platform.md), D-057).
- **Backend service skeleton:** written as a standard first and extracted when a third copy is confirmed (PLAT-15 in [platform.md](platform.md), D-082).
- **One-app code stays in the app.** Nothing moves to a kit because it might be reused.
