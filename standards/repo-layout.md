# Repo layout

Where things live at the top of each repo, where build output goes, and what a deliverable file is called. What git ignores is HYG-6 in [repo-hygiene.md](repo-hygiene.md). Each stack is already close to one shape internally; these rules name that shape so a new repo starts from it and an old one knows what is out of place.

Counts are against the in-scope repos a rule applies to: the 5 standalone WinUI apps (display-switchr, wsl-status, little-launcher, repilot, immich-drive), the 6 Windows-building repos (those five plus persistent), the 4 web monorepos (persistent, printstream, game-is-up, store-sign-controller), the 2 Android repos (phonelock, sonarbridge), and the 3 kits (devkit, winui-app-kit, apps-promo).

The trees show the canonical set. An entry marked "if any" is omitted when the repo has nothing to put there. Several entries rest on a decision owned by another topic; those are noted beside the entry.

## Rules

### LAYOUT-1: WinUI app layout

- **Status:** Existing (5 of 5 for `<App>/`, `<App>MSIX/`, `external/`, `scripts/windows-build`, `windows-build.json`, `Directory.Build.props`, `package.json`; `tests/` in 3 of 5)
- **Applies to:** Windows
- **Reference:** `immich-drive/` for the file set; `display-switchr/store-assets/` for listings
- **Why:** The shared build tooling, the kit and every agent guide assume these paths. A repo that matches needs no per-repo explanation.

```text
<repo>/                         kebab-case
├── <App>/                      app project (PascalCase); see LAYOUT-2
├── <App>MSIX/                  Package.appxmanifest, Images/, packaging script
├── <Helper>/                   extra shipped projects, if any (for example RepilotKey/, ChatMixr.GameBar/, D-120)
├── tests/
│   └── <Name>/                 one folder per test harness (see testing.md)
├── external/
│   ├── winui-app-kit/          submodule
│   └── promo/                  submodule
├── scripts/
│   └── windows-build           wrapper for the shared build tooling, plus repo helpers
├── docs/                       developer docs (D-035); documents only, no images
├── assets/                     icon masters and cross-surface exports (D-102; see assets.md)
├── store-assets/
│   ├── microsoft/              listing text and screenshots (D-066)
│   └── web/                    README and website screenshots
├── release-notes/
│   └── <version>.md            (D-058); a localized app adds <version>.<identity>.md
├── .agents/                    agent docs (see ai-docs.md)
├── .github/
│   ├── workflows/
│   └── scripts/                helpers called only by workflows
├── AGENTS.md
├── README.md
├── LICENSE                     (D-008)
├── NOTICE                      where the licence requires one (HYG-2)
├── <App>.slnx                  (D-054)
├── Directory.Build.props       version and shared build settings
├── Directory.Build.targets     runtime guard (D-053)
├── global.json
├── windows-build.json
├── package.json                script entry points only; no dependencies
├── .editorconfig
├── .gitattributes
├── .gitignore
└── .gitmodules
```

Local only, never tracked: `.artifacts/`, `.windows-build/`, `bin/` and `obj/` under each project.

A WinUI app inside a web monorepo (persistent) keeps this same set under `apps/desktop/`, with `windows-build.json` and `scripts/windows-build` at the monorepo root.

### LAYOUT-2: Folders inside the WinUI app project

- **Status:** Existing (5 of 5 have `Classes`, `Models`, `Pages`, `Services`, `Resources`; 4 of 5 have `ViewModels` and `Windows`)
- **Applies to:** Windows
- **Reference:** `little-launcher/LittleLauncher/`
- **Why:** The same class is in the same folder in every app, so a fix found in one app can be found in the next.

| Folder | Holds |
| --- | --- |
| `Classes/` | Plain helper and policy classes with no UI |
| `Models/` | Data shapes |
| `ViewModels/` | View models, including `UserSettings` |
| `Pages/` | Settings and content pages |
| `Windows/` | Secondary windows; `MainWindow` and `SettingsWindow` stay at the project root |
| `Controls/` | Reusable controls, if any |
| `Services/` | Long-lived services (update, tray, startup) |
| `Resources/` | Styles, icons and other embedded resources |
| `Localization/` | String catalogs; the folder and file form are set by D-067 (see [locales.md](locales.md)) |

`NLog.config` and `app.manifest` sit at the project root (5 of 5). Do not add a `Helpers/` folder; that code belongs in `Classes/`. Code that winui-app-kit already provides is not copied into these folders (see [shared-kits.md](shared-kits.md)).

### LAYOUT-3: Node monorepo layout

- **Status:** Existing (4 of 4 for `apps/api`, `apps/web`, `packages/shared`; 3 of 4 for `scripts/dev/` and `scripts/deploy/`; store-sign-controller has no `scripts/dev/`)
- **Applies to:** Web
- **Reference:** `persistent/` for the layout; `printstream/scripts/ci/` for CI helpers
- **Why:** Four products share one skeleton. Keeping it identical is what makes devkit, the compose files and any future shared web package fit all of them.

```text
<repo>/
├── apps/
│   ├── api/                    Express API; src/server.ts (entry), src/app.ts, src/lib/, src/routes/
│   ├── web/                    Vite and React; src/pages/, src/components/, src/lib/
│   ├── mobile/                 Capacitor shell, if any
│   ├── desktop/                Windows host, if any (LAYOUT-1 shape inside)
│   └── linux-desktop/          Electron host, if any
├── packages/
│   └── shared/                 contracts shared by api and web
├── integrations/               add-ons published elsewhere, if any (see LAYOUT-5)
├── scripts/
│   ├── dev/                    local development and test runners
│   ├── deploy/                 deployment
│   └── ci/                     helpers called by workflows and the gate
├── docker/                     Dockerfiles other than the production one
├── docs/
├── assets/                     icon masters and cross-surface exports (D-102; see assets.md)
├── store-assets/
│   └── <store>/                one listing home per product (D-066)
├── release-notes/              or the changelog JSON the app displays (D-058)
├── .agents/
├── .github/workflows/
├── AGENTS.md
├── README.md
├── LICENSE                     (D-008)
├── ARCHITECTURE.md, GLOSSARY.md   where they exist (D-035)
├── package.json, package-lock.json
├── .nvmrc, .npmrc
├── tsconfig.base.json
├── eslint.config.mjs, .prettierrc.json
├── devkit.config.mjs, paseo.json
├── Dockerfile
├── compose.dev.yml             and the production compose file (D-079)
├── .env.example                (D-078), plus `.env.<role>.example` per role (server, dev, bridge)
├── *.example.yml, *.conf.example   self-hosted products' documented example configs (compose, nginx, prometheus)
├── tsconfig.test.json          where tests need their own compiler settings
├── .vscode/extensions.json, settings.json   shared editor recommendations, if any
├── .dockerignore
├── .editorconfig, .gitattributes, .gitignore
```

Local only, never tracked: `node_modules/`, `dist/`, `.artifacts/`, `.secrets/`, `.backups/`, `tmp/`, `.env`.

A product that ships demo content its app serves (printstream's `data/demo-library/`) may keep it tracked under `data/`; a demo file over the 5 MB pre-commit limit is listed with `--allow-large` in the hook, with a comment saying why it is tracked.

Tests sit beside the code they test as `*.test.ts` (4 of 4). The npm workspace list names `apps/api`, `apps/web` and `packages/*` explicitly, so native shells are not installed as workspaces by accident (persistent and store-sign-controller do this; the other two use the `apps/*` glob). Compose and env file names are in [settings-config.md](settings-config.md).

### LAYOUT-4: Android repo layout

- **Status:** Existing (1 of 2: sonarbridge; phonelock keeps Gradle at the repo root and calls its scripts folder `tools/`). phonelock's move is Decided (D-094)
- **Applies to:** Android (native apps; Capacitor shells follow LAYOUT-3)
- **Reference:** `sonarbridge/`
- **Why:** Nesting the Gradle project leaves the repo root for the things every repo has (docs, scripts, listings), and keeps room for a companion such as firmware or a server.

```text
<repo>/
├── android/                    Gradle project root
│   ├── app/                    or one folder per app module
│   ├── gradle/                 wrapper and version catalog (D-055)
│   ├── gradlew
│   ├── settings.gradle.kts
│   ├── build.gradle.kts
│   └── gradle.properties
├── <companion>/                non-Android parts, if any (firmware, server)
├── scripts/                    build, release and device helpers
├── docs/
├── assets/                     icon masters and cross-surface exports (D-102; see assets.md)
├── store-assets/
│   └── play/                   listing source (D-066); a fastlane tree is generated from it
├── release-notes/              (D-058)
├── .agents/
├── .github/workflows/
├── AGENTS.md
├── README.md
├── LICENSE                     (D-008)
├── package.json                script entry points only (D-044)
├── .editorconfig, .gitattributes, .gitignore
```

Local only, never tracked: Gradle `build/` and `.gradle/` folders, `.kotlin/`, `local.properties`, `.artifacts/`, `.secrets/`, release keystores.

The rule applies to new Android repos now. Under D-094, phonelock moves its Gradle root under `android/` and renames `tools/` to `scripts/`, in the same change as its Gradle work under D-055.

### LAYOUT-5: Home Assistant integration layout

- **Status:** Existing (1 of 1). What the public repo contains: Decided (D-015)
- **Applies to:** HA
- **Reference:** `printstream-home-assistant/`; source at `printstream/integrations/home-assistant/`
- **Why:** HACS and hassfest require this shape, and the public repo is generated, so the layout is fixed in the source folder.

```text
<repo>/                         generated; do not edit here
├── custom_components/
│   └── <domain>/               manifest.json, the integration code, translations/, brand/
├── .github/workflows/          validate.yml, release.yml
├── hacs.json
├── README.md
├── LICENSE
└── NOTICE
```

The source of truth is a folder in the parent product (`integrations/home-assistant/` in printstream), which also holds what is not exported today: `card-source/` and `tests/`. Changes are made there and reach the public repo through the export script.

### LAYOUT-6: Shared kit layout

- **Status:** Existing (1 of 3: winui-app-kit; devkit is close; apps-promo keeps source, script and test at the root)
- **Applies to:** Kit
- **Reference:** `wsl-status/external/winui-app-kit/`
- **Why:** A kit is read from inside other repos as a submodule or package, so it must be small, flat and free of anything a consumer has to skip over.

```text
<repo>/
├── <Project>/  or  src/        one source folder
├── Xaml/                       shared-items XAML, if the kit ships any (beside, not inside, the library)
├── build/                      imported .props, .targets and shared files, if any
├── tests/                      harnesses, if any
├── scripts/                    if any (winui-app-kit: dotnet, run-tests, windows-build, check-gitignore, check-localization.py)
├── release-notes/              one file per version (REL-6)
├── .agents/, .claude/skills    agent guides and skills (AI-6 to AI-8)
├── .github/                    workflows, SECURITY.md, issue template
├── README.md                   the kit's usage doc (see readme.md)
├── AGENTS.md
├── LICENSE                     (D-008)
├── .editorconfig, .gitattributes, .gitignore
```

A kit has no committed build output (LAYOUT-13) and no consumer-specific paths. XAML that apps compile through a shared-items import sits in its own folder outside the library project, because the library would otherwise compile it too and produce conflicting resources (winui-app-kit's `Xaml/`, the same mechanism apps-promo uses).

### LAYOUT-7: Submodules live under `external/`

- **Status:** Existing (6 of 6 consumers; persistent nests it at `apps/desktop/external/`)
- **Applies to:** Windows, Web
- **Reference:** `wsl-status/.gitmodules`
- **Why:** One folder name tells people and tools which code is not owned by this repo.

Submodules are checked out at `external/<name>` beside the project that uses them: `external/promo`, `external/winui-app-kit`, and optionally `external/standards`. The folder name is the short name, not the repo name. `.gitmodules` uses the current organisation URL; the three stale ones are FIX-046. How kits are versioned and pinned is in [shared-kits.md](shared-kits.md).

### LAYOUT-8: `scripts/` holds the repo's scripts, and only that

- **Status:** Existing (11 of 15 have `scripts/`; 3 of 4 monorepos split it into `dev/` and `deploy/`; phonelock uses `tools/` and a root `build.sh`)
- **Applies to:** all
- **Reference:** `printstream/scripts/` (split by purpose); `display-switchr/scripts/` (small repo)
- **Why:** The same word means one wrapper in the Windows apps, a tree in the monorepos and release helpers on Android; a fixed meaning tells an agent where to look and where to add.

- Scripts that people or `package.json` run live under `scripts/`. A repo with more than a handful splits them into `scripts/dev/`, `scripts/deploy/` and `scripts/ci/`.
- Helpers called only by a workflow live in `.github/scripts/`.
- The MSIX packaging script and image generator stay in `<App>MSIX/` until they move to winui-app-kit under D-005.
- No script sits at the repo root, and the folder is not called `tools/`.
- A script's tests sit beside it (`x.test.mjs`).

Script names and what each must do are in [dev-scripts.md](dev-scripts.md).

### LAYOUT-9: The repo root is a closed set

- **Status:** Proposed (1 of 15 tracks a leftover folder: `legacy/` in display-switchr; five more working trees hold untracked strays at the root)
- **Applies to:** all
- **Reference:** `wsl-status/` (the smallest root of the Windows apps)
- **Why:** The root is the first thing read. Leftovers there are copied into the next repo.

The root holds only what its stack's tree lists. Retired code is deleted, not kept in a `legacy/` folder; git history keeps it. Scratch output, logs and one-off captures go under `.artifacts/` or `tmp/`, never in a named root file. Documents other than `README.md`, `AGENTS.md`, the licence and the files D-035 allows live under `docs/` (see [docs.md](docs.md)).

### LAYOUT-10: Build output goes under a git-ignored root `.artifacts/`

- **Status:** Decided (D-040). Existing (6 of 6 Windows-building repos; 0 of the 5 other repos that produce deliverables)
- **Applies to:** all repos that build something a person picks up
- **Reference:** `display-switchr/.gitignore` and its `.artifacts/windows-build/`; `little-launcher/linux/packaging/build.py` (Linux output under `.artifacts/`)
- **Why:** One folder means one ignore rule, one place to look for the last build, and one place to clean. The same idea has three names today (`.artifacts/`, `artifacts/`, `release/`).

Build output goes to `.artifacts/` at the repo root everywhere, including monorepos, with fixed subfolders:

| Path | Holds |
| --- | --- |
| `.artifacts/windows-build/<utc>-<id>/` | One folder per run of the Windows build tooling: `artifacts/`, `build.log`, `result.json` |
| `.artifacts/packages/<version>/` | Final, named deliverables for any platform (LAYOUT-12) |
| `.artifacts/packages/<product>/` | Deliverables that are content-addressed (a source fingerprint) or that a tool keeps as a multi-version tree (electron-builder), instead of per version |
| `.artifacts/work/<product>/` | Scratch a packaging step needs outside the tool's own build folder, deleted freely |
| `.artifacts/evidence/<topic>/` | Screenshots, verification output and other proof kept for review |

Intermediate output stays where the tool puts it and stays ignored: `bin/` and `obj/`, `dist/`, Gradle `build/`. A packaging step copies the finished, named file into `.artifacts/packages/`; nothing is written beside the sources.

### LAYOUT-11: Old build output is pruned automatically

- **Status:** Decided (D-040). Proposed (nothing prunes today; about 25 GB is retained across the six repos)
- **Applies to:** Windows; any other repo that writes per-run folders
- **Reference:** none
- **Why:** Runs are kept on purpose for review and reinstalling, but without a limit the folder only grows.

The build tooling keeps the last N runs per repo (10 is suggested) plus any run that was installed or released, deletes the rest at the start of the next run, and has a flag to keep a run. `.artifacts/packages/` keeps released versions; `.artifacts/evidence/` is cleared by hand. Pruning lives in the shared tooling, not in each repo.

### LAYOUT-12: Deliverable file names and formats

- **Status:** Decided (D-041). Proposed (closest today: `LittleLauncher-<version>-linux-x64.AppImage`)
- **Applies to:** all repos that publish a file
- **Reference:** `little-launcher/linux/packaging/build.py`
- **Why:** No two families name files the same way today, and three leave the version out, so a downloaded file cannot be identified.

The pattern is `<Product>-<version>-<platform>-<arch>[-<channel>].<ext>`:

| Part | Values |
| --- | --- |
| Product | PascalCase, no dots or spaces (`LittleLauncher`, `Persistent`) |
| Version | Three-part for releases; four-part only for dev MSIX builds |
| Platform | `windows`, `linux`, `android` |
| Arch | `x64`, `arm64`; omitted where the file is not architecture-specific |
| Channel | Only where one platform has two builds: `store`, `portable`, `play` |

Examples: `LittleLauncher-1.43.0-windows-x64-portable.zip`, `Persistent-0.30.1-android-play.apk`.

| Platform | Format |
| --- | --- |
| Windows | `.msix` per architecture; `.msixbundle` inside `.msixupload` for the Store; `.zip` for a portable build |
| Linux | `.AppImage` with an update manifest |
| Android | `.aab` to Play; `.apk` for direct download |
| Any release with downloadable files | One `SHA256SUMS` file |

The pattern applies to new products now, and to an existing asset only where no installed updater fetches it by name. phonelock's updater does, and the little-launcher and persistent Linux updaters have not been checked, so those names stay until each is checked; a rename attaches both names for one release. Which files a release attaches is in [releases.md](releases.md) and [distribution.md](distribution.md).

### LAYOUT-13: No build output is committed

- **Status:** Decided (D-042). Existing (13 of 15; the exceptions are a `.vsix` in devkit and a firmware image in sonarbridge)
- **Applies to:** all
- **Reference:** `winui-app-kit` (none tracked)
- **Why:** A committed binary goes stale silently and cannot be matched to the source that built it.

The firmware image moves to GitHub Release assets, which gives it a versioned release path. devkit's `.vsix` is the one exception (D-042, amended 2026-11-18): consumers install devkit from a git tag with no build step and load the file from the installed package, so it stays tracked, and CI fails when a rebuild from source differs from it. Generated files that a platform needs in the tree (MSIX image sets, the HA card bundle) are a separate case covered by ASSET-4 in [assets.md](assets.md) (D-069). This rule owns the no-committed-output point; HYG-12 in [repo-hygiene.md](repo-hygiene.md) and DIST-5 in [distribution.md](distribution.md) link here.

### LAYOUT-14: What every repo git-ignores

- **Status:** Retired. Merged into HYG-6 in [repo-hygiene.md](repo-hygiene.md), which owns the `.gitignore` file, the shared block every repo carries, and the order of its parts.
