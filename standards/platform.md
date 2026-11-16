# Platform baselines

Which SDKs, runtimes, operating system versions and framework generations the apps build on, and where each is pinned. Shared code (winui-app-kit, devkit, any shared web package) can only target one baseline, and a security bump that has to be repeated by hand in each repo gets missed in some.

Counts are against the in-scope repos a rule applies to: the 6 Windows-building repos (hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, persistent), the 5 Node repos (persistent, printstream, game-is-up, store-sign-controller, devkit), the 4 web monorepos (the Node repos without devkit), and the 2 native Android repos (phonelock, sonarphone-nmea-bridge).

**This file states pin policy, not "current" versions.** The table records what the repos pinned when it was last checked (2026-11-16), so drift is visible. It is not a list of what to install. The dependency-version rule in the user-level agent rules (kept outside this repo) applies: check the registry before adding or moving a version, and never take one from memory or from this table.

## Baselines as pinned today

| Stack | Item | Pinned in | Value in the repos today | Target |
| --- | --- | --- | --- | --- |
| Windows | .NET SDK | `global.json` | 10.0.403 (little-launcher, repilot, persistent); 10.0.100 (hub-display-switch, wsl-monitor); no file (immich-drive, winui-app-kit) | PLAT-2 |
| Windows | Target framework | app csproj | `net10.0-windows10.0.22000.0` (6 of 6, and the kit) | PLAT-4 |
| Windows | Windows App SDK | app csproj | 1.8.260209005 (6 of 6 main app projects, and the kit) | PLAT-5 |
| Windows | Minimum Windows build | `Package.appxmanifest` | 22000 (5 of 6); 19045 (immich-drive) | PLAT-6 |
| Windows | Solution format | repo root | `.slnx` (5 of 6); `.sln` (little-launcher) | PLAT-7 |
| Web | Node | `.nvmrc`, `engines` | 22.23.4 (persistent, printstream, game-is-up); 24 by `engines` range, no `.nvmrc` (store-sign-controller); 20.20.2 (devkit) | PLAT-8 |
| Web | npm | `packageManager` | 10.9.9 (3 of 4 monorepos); unset (store-sign-controller) | PLAT-8 |
| Web | TypeScript | `package.json` | 5.8 (persistent); 6 with the native 7 compiler alongside (printstream, game-is-up); 7.0.2 (store-sign-controller) | PLAT-9 |
| Web | React | `apps/web/package.json` | 18.3 (3 of 4); 19.3 (store-sign-controller) | PLAT-9 |
| Web | Prisma | `apps/api/package.json` | 6.5 (3 of 4); 7.10 (store-sign-controller) | PLAT-9 |
| Web | zod | `packages/shared/package.json` | 3.24 (3 of 4); 4.6 (store-sign-controller) | PLAT-9 |
| Web | Express | `apps/api/package.json` | 5 (4 of 4) | PLAT-9 |
| Web | Vite | `apps/web/package.json` | 8 (3 of 4); 6.4 (persistent) | PLAT-9 |
| Web | Dev database | `devkit.config.mjs` | Postgres 16 (3 of 3 devkit consumers) | [shared-kits.md](shared-kits.md) |
| Android | JDK | Gradle `compileOptions`, build container | 17 (2 of 2) | PLAT-11 |
| Android | Gradle | wrapper | 8.7 by wrapper (sonarphone-nmea-bridge); 9.9.1 from a host install, no wrapper (phonelock) | PLAT-12 |
| Android | Android Gradle Plugin | root build script | 8.5.2 (sonarphone-nmea-bridge, a documented hold); 9.5.1 (phonelock) | PLAT-12 |
| Android | `minSdk` | app build script | 29 (sonarphone-nmea-bridge); 33 (phonelock); 24 (persistent's Capacitor shell) | PLAT-11 |
| Android | `targetSdk` | app build script | 36 (2 of 2, and persistent's shell) | PLAT-11 |
| Android | `compileSdk` | app build script | 36 (sonarphone-nmea-bridge, a documented hold); 37 (phonelock) | PLAT-11 |
| HA | Minimum Home Assistant | `hacs.json` | 2024.6.0 | PLAT-14 |

## Rules

### PLAT-1: Every toolchain version is pinned in one file that tools read

- **Status:** Existing in part (.NET: 5 of 6 have a `global.json`; Node: 3 of 5 have `.nvmrc` with exact `engines`; Gradle: 1 of 2 has a wrapper). The upgrade cadence is Decided (D-095)
- **Applies to:** all
- **Reference:** `repilot/global.json`, `game-is-up/package.json` with `.nvmrc`, `sonarphone-nmea-bridge/android/gradle/wrapper/`
- **Why:** A version that lives in a README, a workflow file or someone's machine is a version each of them can get wrong separately.

| Tool | The pin | Read by |
| --- | --- | --- |
| .NET SDK | `global.json` | `dotnet`, CI (`global-json-file`) |
| Node and npm | `.nvmrc`, exact `engines`, `packageManager`, `.npmrc` `engine-strict=true` | nvm, npm, devkit, CI (`node-version-file`) |
| Gradle | the committed wrapper | `gradlew`, CI |
| NuGet and npm packages | the csproj or shared props; `package.json` and the lockfile | the build |

CI, Dockerfiles, docs and scripts read the pin; they do not repeat the number. Where a tool cannot read the file (a Dockerfile base image), a comment names the file it must match. The persistent workflows that hard-code Node are FIX-050; the README that states stale Android versions is FIX-022; how workflows read pins is in [ci.md](ci.md).

Moving a pin is its own change, made in every repo of the stack in one pass, and not mixed into feature work. A pin held back on purpose has a comment or doc line naming the reason, as `sonarphone-nmea-bridge/AGENTS.md` does for its Android levels.

Each stack's pins are reviewed once a quarter and moved together (D-095). A security release does not wait for that pass; its deadline is SEC-12 in [security.md](security.md) (D-107).

### PLAT-2: One .NET SDK pin

- **Status:** Decided (D-053). Existing (3 of 6 pin 10.0.403; 4 of the 5 with a `global.json` use `rollForward: latestFeature`)
- **Applies to:** Windows, Kit (winui-app-kit)
- **Reference:** `repilot/global.json`
- **Why:** Two SDK bands are pinned today, and the older one predates the runtime fix that PLAT-3 requires.

Every Windows-building repo and winui-app-kit has a `global.json` with the same SDK version, `rollForward: latestFeature` and `allowPrerelease: false`. The version is the one repilot pins, 10.0.403 at the time of writing. In a monorepo the file sits with the desktop solution (`persistent/apps/desktop/global.json`).

### PLAT-3: Runtime security guard

- **Status:** Decided (D-053). Existing (3 of 6: little-launcher, repilot, persistent; three separate copies)
- **Applies to:** Windows
- **Reference:** `repilot/Directory.Build.targets`
- **Why:** The apps ship self-contained, so the .NET runtime inside the package is the app's responsibility. A build on an old runtime ships a known vulnerability without any warning.

A `Directory.Build.targets` check fails the build when the runtime pack that was actually resolved is older than the current security floor. The implementation is repilot's, which checks the resolved runtime pack and not just the intent, shipped once from winui-app-kit so the floor is raised in one place (D-005). Until the kit ships it, a repo copies repilot's file unchanged.

### PLAT-4: Target framework and project baseline

- **Status:** Existing (6 of 6, and winui-app-kit)
- **Applies to:** Windows, Kit (winui-app-kit)
- **Reference:** `wsl-monitor/WslMonitor/WslMonitor.csproj`
- **Why:** The kit compiles into each app, so the app and the kit must agree on framework, platforms and packaging model.

The app project targets `net10.0-windows10.0.22000.0`, builds for `x64` and `ARM64` with matching runtime identifiers, and sets `WindowsPackageType` to `None`; packaging is done by the script in the `<App>MSIX` folder (see [repo-layout.md](repo-layout.md)), which publishes the app self-contained. The platform auto-detect block in `Directory.Build.props` is kept as in the kit's copy. Language and nullable settings are in [code-style.md](code-style.md). The version number's home is in [releases.md](releases.md).

### PLAT-5: One Windows App SDK version, and shared package versions in one place

- **Status:** Existing (6 of 6 main app projects and the kit are on the same Windows App SDK version). The central version file is Decided (D-096) (0 of 6)
- **Applies to:** Windows, Kit (winui-app-kit)
- **Reference:** `wsl-monitor/external/winui-app-kit/WinUIAppKit/WinUIAppKit.csproj`
- **Why:** The Windows App SDK agrees today only because it was pasted at the same time. NLog and Sqlite, pinned the same way, have already drifted.

Every WinUI project in the family uses the Windows App SDK version that winui-app-kit uses; an app does not move ahead of the kit. The packages the family shares (Windows App SDK, NLog, CommunityToolkit.Mvvm) get their version from one shared file, not from each csproj. That file is central package management (`Directory.Packages.props`), with the shared versions imported from the kit, which is where D-005 puts shared build props (D-096).

One secondary project is on a newer major today (persistent's widget). A project that needs a newer major than the kit says why in a comment, and the kit is moved as its own change.

### PLAT-6: Minimum Windows version

- **Status:** Decided (D-018). Existing (6 of 6 manifests already carry the decided value: build 22000 in five, 19045 in immich-drive)
- **Applies to:** Windows, Kit (winui-app-kit)
- **Reference:** `repilot/RepilotMSIX/Package.appxmanifest`
- **Why:** A floor nobody tests on is a support liability on a paid app, and the README, website and manifest disagree for the published apps today.

The floor is Windows 11 build 22000 for every app, with one decided exception: immich-drive keeps Windows 10 22H2 (build 19045). winui-app-kit's floor is Windows 10 22H2 so that every app, immich-drive included, can use it. `Package.appxmanifest` is the source of truth; the README, the store listing and the website repeat what it says (see [readme.md](readme.md) and [store-listings.md](store-listings.md)).

The exception has two conditions. Every immich-drive release is tested on Windows 10 before it ships, which is accepted (TEST-9 in [testing.md](testing.md)). And any winui-app-kit feature that needs Windows 11 checks the OS version at runtime and degrades on Windows 10; the kit's README names such features, and the kit is tested on Windows 10 22H2. No other app takes a lower floor without a new decision. The repilot mismatch is FIX-021, whose correct value is now 22000.

### PLAT-7: Solutions are `.slnx`

- **Status:** Decided (D-054). Existing (5 of 6; little-launcher has `.sln` and an unused `.wapproj`)
- **Applies to:** Windows
- **Reference:** `repilot/Repilot.slnx`
- **Why:** It is the format five repos already use, and it merges cleanly because it has no generated identifiers.

One `<App>.slnx` at the repo root (or beside the desktop projects in a monorepo) lists the app, helper and test projects. No `.sln` or `.wapproj` is kept alongside it. little-launcher's leftover is FIX-033.

### PLAT-8: Node is pinned exactly, in four places that agree

- **Status:** Existing (3 of 5 Node repos have all four: persistent, printstream, game-is-up). Which version: Decided (D-007)
- **Applies to:** Web, Kit (devkit)
- **Reference:** `game-is-up/package.json`, `game-is-up/.nvmrc`, `game-is-up/.npmrc`
- **Why:** An exact pin enforced by npm stops "works on my Node" before install, and devkit keys its install cache on these files.

A Node repo has `.nvmrc` with an exact version, `engines.node` and `engines.npm` with the same exact values, a `packageManager` field, and `.npmrc` with `engine-strict=true`. A range in `engines` is not a pin.

The fleet version is store-sign-controller's generation (D-007): it is the target for all web repos, persistent and game-is-up migrate first, printstream moves last because its pin has a documented runtime-parity reason, and devkit moves to the fleet version. devkit's mismatch with its own CI is FIX-057.

### PLAT-9: One web library generation

- **Status:** Decided (D-007). Existing (3 of 4 web monorepos share one generation: persistent, printstream, game-is-up; store-sign-controller is one major ahead on React, Prisma, zod and TypeScript)
- **Applies to:** Web
- **Reference:** `store-sign-controller/package.json` (the target); `printstream/package.json` (the generation three repos share)
- **Why:** The four products share one skeleton (Express, Prisma, zod contracts, Vite, React). Code and fixes move between them only while the majors match.

All web monorepos are on the same major version of TypeScript, React, Prisma, zod, Express and Vite. D-007 takes store-sign-controller's majors as the target, and the table at the top of this file is the versions table that decision calls for. A repo moves one library major at a time, as its own change, and updates the table in the same pass.

Running the native TypeScript compiler beside the standard one (printstream, game-is-up) is allowed as a speed measure; the version that `typecheck` gates on is the one in the table.

### PLAT-10: Dependency versions are exact

- **Status:** Decided (D-037). Existing (1 of 4 web monorepos: store-sign-controller; .NET package references are exact in 6 of 6)
- **Applies to:** Web, Kit (devkit); already the practice on Windows
- **Reference:** `store-sign-controller/package.json`
- **Why:** With exact pins a diff shows precisely what changed, and two repos on "the same" range cannot quietly resolve to different versions.

`package.json` lists exact versions with no `^` or `~`, the lockfile is committed, and updates arrive as grouped Dependabot pull requests (HYG-8 in [repo-hygiene.md](repo-hygiene.md)). A version held below the latest has a comment naming the constraint, as the dependency rule in the user-level agent rules requires. Android dependencies follow PLAT-12.

### PLAT-11: Android SDK levels and JDK

- **Status:** Existing (2 of 2 use JDK 17 for source, target and the Kotlin JVM target; 2 of 2 target SDK 36)
- **Applies to:** Android, and Capacitor shells
- **Reference:** `sonarphone-nmea-bridge/android/app/build.gradle.kts`
- **Why:** Play enforces a target level on a schedule, so the level is a deadline and must be visible. The minimum level decides who can install the app, so it is a product choice and must be written down.

- **JDK:** one version for `sourceCompatibility`, `targetCompatibility`, the Kotlin JVM target and the build container image. All four agree.
- **`targetSdk`:** the level Play currently requires for updates, the same in every app. Raising it is its own change, with the behaviour changes of the new level reviewed.
- **`compileSdk`:** equal to or above `targetSdk`.
- **`minSdk`:** chosen per app from what the app needs, and stated with its reason in the README's requirements and in `AGENTS.md`. It is not raised without the maintainer's decision, because raising it strands installed users.
- A level held back on purpose is documented where the build file sets it, as sonarphone-nmea-bridge does for its build-tools and API 37 holds.

Android application ids are BRAND-7 in [branding.md](branding.md) (D-011).

### PLAT-12: Gradle conventions

- **Status:** Decided (D-055). Existing (1 of 2 has a committed wrapper and Kotlin DSL: sonarphone-nmea-bridge; 0 of 2 has a version catalog)
- **Applies to:** Android
- **Reference:** `sonarphone-nmea-bridge/android/` (wrapper, Kotlin DSL, build container); `phonelock/companion/build.gradle` (each third-party dependency carries an "approved:" comment with the reason)
- **Why:** A build that needs a particular Gradle in a home directory cannot run in CI or a container.

For new work and for phonelock:

- The Gradle wrapper is committed, and every build goes through `gradlew`. No script names a host Gradle install.
- Build scripts are Kotlin DSL (`*.gradle.kts`).
- Plugin and library versions live in a version catalog (`gradle/libs.versions.toml`), not inline.
- Each third-party dependency has a comment saying why it was approved.
- Builds run in the repo's build container, in line with the Docker and memory-limit sections of the user-level agent rules.

sonarphone-nmea-bridge keeps its documented Android Gradle Plugin and Kotlin holds; "latest everywhere" is not the rule.

### PLAT-13: One host technology per platform for web products

- **Status:** Decided (D-057). Existing (3 of 3 repos with host shells follow the same technology per platform: persistent, printstream, little-launcher; store-sign-controller dropped its shells when it became web only)
- **Applies to:** Web products with native shells; Windows for the WebView2 hosts
- **Reference:** `persistent/apps/desktop/` (WinUI with WebView2), `persistent/apps/linux-desktop/` (Electron), `persistent/apps/mobile/` (Capacitor)
- **Why:** The technology per platform is already consistent. What is missing is a written contract between the host and the page it loads, which is where version skew breaks users.

| Platform | Host |
| --- | --- |
| Windows | WinUI with WebView2 |
| Linux | Electron |
| Android and iOS | Capacitor |

A new host shell uses the technology in the table. D-057 leaves the existing host code as it is and writes down the host bridge contract (settings, notifications, updates, version negotiation); that contract is written as a contract document (DOCS-10 in [docs.md](docs.md)) and linked from this rule once it exists. The toolkit for native Android UI (D-056) is in [ui.md](ui.md).

### PLAT-14: Home Assistant baseline

- **Status:** Existing (1 of 1 states a minimum Home Assistant version). Python toolchain: Decided (D-084)
- **Applies to:** HA
- **Reference:** `printstream/integrations/home-assistant/hacs.json`
- **Why:** The integration runs inside the user's Home Assistant, which fixes the Python version and the APIs available, so the minimum release is the whole platform statement.

`hacs.json` states the minimum Home Assistant release the integration supports, and the README's requirements repeat it. The integration uses no Python or Home Assistant API newer than that release provides. Raising the minimum is called out in the release notes. Tooling for the Python source is in [code-style.md](code-style.md) and [testing.md](testing.md).

### PLAT-15: Backend service skeleton

- **Status:** Decided (D-082). Existing (`/api/health` in 4 of 4 web APIs; readiness and status routes, and a logger with correlation ids, in 1 of 4: printstream)
- **Applies to:** Web
- **Reference:** `printstream/apps/api/`
- **Why:** Monitoring a product and reading its errors work differently in each product today, for no product reason.

Every API has the same skeleton, with printstream as the reference (D-082).

| Part | Rule |
| --- | --- |
| Liveness | `GET /api/health`; the container health check calls it (SEC-7 in [security.md](security.md)) |
| Readiness | A route that reports whether the database and the other services the API needs are reachable |
| Error shape | One error shape for every failed request; SEC-8 relies on it for rejected requests |
| Logging | The `logger` module with a request correlation id (LOG-3 in [logging.md](logging.md)) |
| Mail | One mailer module per API |

The skeleton is written down here first and extracted into a shared package only when a third copy is confirmed (KIT-11 in [shared-kits.md](shared-kits.md)).
