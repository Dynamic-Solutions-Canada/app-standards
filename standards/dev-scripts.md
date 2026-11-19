# Dev scripts

The names of the commands that run, check, build, package and deploy each app, and what each one must do. The aim is that `npm run validate` or `npm run verify:desktop` means the same thing in every repo, so people, agents, `paseo.json` and CI can call a job without reading the repo first.

Counts are against the in-scope repos a rule applies to. The 8 repos with target-specific root scripts are hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, persistent, store-sign-controller and printstream. The 6 Windows-building repos are the five standalone WinUI apps and persistent. The 4 web monorepos are persistent, printstream, game-is-up and store-sign-controller.

Names are settled by D-043 and the Android entry point by D-044.

## Canonical names

A repo defines only the rows it has a use for. It never uses one of these names for a different job (SCRIPT-3).

| Job | Name | Must do | Stacks |
| --- | --- | --- | --- |
| Run the dev stack | `dev` | Start everything needed to use the app locally, in the foreground | Web |
| Dev lifecycle | `dev:background`, `dev:down`, `dev:cleanup`, `dev:doctor`, `dev:bootstrap`, `dev:host`, `dev:prepare-worktree` | Devkit lifecycle commands (SCRIPT-7) | Web |
| Compile | `build`, `build:<target>` | Compile and bundle. Never packages, signs or installs | Web, Android |
| Lint | `lint` | Run the linter and analyzers; fail on an error | all |
| Format | `format`, `format:check` | Rewrite files; or fail if any file would change | all |
| Typecheck | `typecheck` | Type-check every workspace without emitting | Web |
| Unit tests | `test`, `test:<target>`, `test:scripts` | Run every test of that scope and fail if any fails. Needs no network, device or database | all |
| Integration tests | `test:integration` | Run tests that need a database or other service; fail, not skip, when the service is missing | Web |
| Full gate | `validate` | Everything static the repo has (SCRIPT-4) | all |
| Fast gate | `validate:changed` | The gate, limited to what the working tree changed | optional |
| Native compile check | `verify:<target>` | Prove the native target compiles. Runs no tests and produces no package | Windows (`verify:desktop`), Android (`verify:android`), Linux hosts (`verify:linux`) |
| Dev package | `package:desktop:dev` | Build a signed, installable dev package (today `build:desktop`) | Windows |
| Release package | `package:<target>` | Build the deliverable for a store or release, named and placed as in [repo-layout.md](repo-layout.md) | Windows, Android, Linux hosts |
| Install on a device | `install:<target>` | Install the latest dev package on this machine or the connected device | Windows, Android, Linux hosts |
| Localization check | `check:l10n` | Fail on missing, extra or malformed strings (see [locales.md](locales.md)) | all localized apps |
| Other repo checks | `check:<thing>` | One static check, for example `check:version` | optional |
| Database | `db:generate`, `db:migrate`, `db:migrate:deploy`, `db:seed`, `db:validate`, `db:wait` | See SCRIPT-8 | Web |
| Production start | `start:prod` | Start the built app as production does | Web |
| Deploy | `deploy:prod`, `deploy:staging` | Deploy the current commit to that environment | Web |
| Device helpers | `android:connect`, `android:discover`, `android:pair`, `android:launch`, `android:logs`, `android:emulator` | Tool-style helpers for a phone or emulator | Android, Capacitor shells |
| Store screenshots | `shots`, `shots:<step>` | Produce store screenshots (see [store-listings.md](store-listings.md)) | optional |

Releasing is a guided procedure, not a script name; see [releases.md](releases.md).

## Rules

### SCRIPT-1: A root `package.json` is the entry point in every repo

- **Status:** Existing (10 of 12 repos with something to run: all six Windows-building repos, the four monorepos and devkit; not the two Android repos). Android: Decided (D-044)
- **Applies to:** all
- **Reference:** `hub-display-switch/package.json` (wrapper only, no dependencies); `persistent/package.json` (full set)
- **Why:** One command runner in every repo means an agent, a `paseo.json` entry or a workflow calls `npm run <name>` without knowing the stack underneath.

Every repo with something to run, check or build has a root `package.json` whose `scripts` use the canonical names. In a repo that is not a Node project it is `"private": true`, has no dependencies and no lockfile, and each script is a one-line call to the real tool. The five Windows apps already do this for the Windows build tooling.

D-044 gives phonelock and sonarbridge the same wrapper over Gradle (SCRIPT-9). The Home Assistant integration has no entry point of its own, because it is generated; its scripts are in printstream. A kit with tests exposes `test`.

### SCRIPT-2: Names are `job:target[:variant]`

- **Status:** Decided (D-043). Existing (7 of 8 repos with target-specific root scripts; printstream is the exception, and persistent mixes both orders for Android)
- **Applies to:** all
- **Reference:** `persistent/package.json` (`verify:desktop`, `build:linux`, `package:linux`)
- **Why:** With the job first, the bare name (`build`, `test`, `lint`) is the default target and the same job sorts together across targets.

The job comes first, then the target, then a variant: `verify:android`, `package:desktop:dev`. Targets are `desktop` (Windows), `linux`, `android`, `web`, `api` and `scripts`.

`target:*` is kept only as a namespace for tool-style helpers that are not one of the canonical jobs: `dev:*`, `db:*`, `android:*` and `shots:*`. So `android:logs` is a helper, and the Android compile check is `verify:android`, not `android:verify`.

Renames under this rule are made once per repo, together with the docs, skills and `paseo.json` entries that name the old script. An old name is not kept as an alias.

### SCRIPT-3: Each verb has one meaning

- **Status:** Decided (D-043). Existing in part (`verify:desktop` means "compiles" in 7 of 7 repos that define it; `build:desktop` packages and signs in 6 and builds a ZIP in 1)
- **Applies to:** all
- **Reference:** `hub-display-switch/package.json` for `verify`; `persistent/package.json` for `build`
- **Why:** Two names do two jobs today: `verify:linux` runs tests in little-launcher and a typecheck in persistent, and `build:desktop` signs a package where `build` means compile everywhere else.

| Verb | Means | Does not |
| --- | --- | --- |
| `build` | Compile and bundle | Package, sign or install |
| `verify` | The native target compiles | Run tests or produce a package |
| `test` | Tests run and pass | Depend on a device or a signed-in session |
| `package` | Produce an installable or uploadable deliverable | Install or publish it |
| `install` | Put a dev package on a device | Build it, unless the tool does both in one step |
| `check` | One static check passes | Change files |
| `validate` | The whole gate passes | Change files |
| `deploy` | Publish the current commit to an environment | Skip the gate, unless an explicit flag asks for it |

The renames this implies: `build:desktop` becomes `package:desktop:dev`; little-launcher's `verify:linux` becomes `test:linux`; printstream's `desktop:check` becomes `test:desktop`; game-is-up's `deploy:prod:ssh` becomes `deploy:prod`; wsl-monitor's `check:strings` becomes `check:l10n` (D-067).

### SCRIPT-4: `validate` is the one gate, and it runs everything static

- **Status:** Existing (4 of 4 web monorepos have `validate`; 1 of 4 is complete: printstream). Proposed for Windows, Android and Kit (none of those repos has a `validate`)
- **Applies to:** all
- **Reference:** `printstream/package.json` (`validate`, `validate:stages`, `validate:changed`)
- **Why:** "Run validate before finishing" is only a useful instruction if validate cannot pass while a check the repo owns was never run.

`validate` runs, and fails on any failure of, every static check the repo has:

| Stack | `validate` runs |
| --- | --- |
| Web | `format:check`, `lint`, `typecheck`, `test`, `db:validate`, and any `check:*` |
| Windows | `format:check`, `check:l10n`, `test` (the portable harnesses); `verify:desktop` is run as well before finishing a task, where a Windows build host is available |
| Android | `format:check`, `lint`, `test`, `check:l10n`, `verify:android` |
| Kit | `format:check`, `lint` and `test`, as far as the kit has them |

Adding a check to a repo means adding it to `validate` in the same change. A test or checker that exists in the repo is reachable from `test` or `validate`; a suite that only a person remembers to run by hand is treated as missing. Suites that cannot run unattended are the one exception and are marked as such (TEST-10 in [testing.md](testing.md)). CI runs the same script, not a parallel list of steps (CI-1 in [ci.md](ci.md)). This rule owns the gate; TEST-6 was merged into it. The game-is-up gate that runs no tests is FIX-054. Formatter steps wait on D-016 (see [code-style.md](code-style.md)).

Database-backed tests are not inside `validate` (D-049): they stay in `test:integration` locally, with CI running both, and never a silent skip (see [testing.md](testing.md)).

In a web monorepo that also ships a native app, a suite that needs that platform's toolchain (the .NET SDK for a Windows host, the Android SDK for a Capacitor shell) runs as its own `test:desktop` or `test:android` script rather than inside the root `validate`, so the root gate runs on any machine with only Node. CI runs each such script as its own job, and the release gate includes them, which is what makes them reachable. `verify:<target>` stays compile-only (SCRIPT-3).

### SCRIPT-5: `test` exists at the root and finds tests by itself

- **Status:** Existing (2 of 5 repos with a root `test` use a discovery script: persistent, printstream)
- **Applies to:** all
- **Reference:** `persistent/scripts/dev/run-tests.mjs` (small); `printstream/scripts/dev/run-tests.mjs` (large)
- **Why:** A test that no script runs is not a test. A hand-kept list of folders, or an unquoted `**`, drops new tests without an error.

Root `test` runs every unit test in the repo, in every workspace or project. `test:<target>` narrows to one target and `test:scripts` covers the repo's own scripts. How the script finds test files (a discovery script or quoted globs, never a hand-kept folder list or an unquoted `**`) is owned by TEST-5 in [testing.md](testing.md), which also tracks the two known cases. Frameworks, file names and what must be tested are in the same file.

### SCRIPT-6: Windows entry points go through `scripts/windows-build`

- **Status:** Existing (6 of 6 Windows-building repos; the wrapper is identical in all). The `build:desktop` rename: Decided (D-043)
- **Applies to:** Windows
- **Reference:** `hub-display-switch/package.json`, `hub-display-switch/scripts/windows-build`
- **Why:** One wrapper and four names let the same commands build any app from WSL, a container or a Windows shell.

| Script | Does |
| --- | --- |
| `verify:desktop` | Full Debug compile with the Windows toolchain, including XAML. The check to run before finishing a change. It compiles only the `project` named in `windows-build.json`; a repo that ships a second project (immich-drive's thumbnail provider) uses `package:desktop:dev` as its compile proof for changes that touch that project, until the tooling compiles every shipped project |
| `package:desktop:dev` | Builds a signed Release dev MSIX and keeps the run under `.artifacts/windows-build/` |
| `install:desktop` | Builds, updates the installed dev package in place, then launches it |
| `package:desktop` | Builds the unsigned Release MSIX with the Store identity |

All four are one-line calls to `scripts/windows-build`, which hands the working tree to the shared Windows build tooling using the repo's `windows-build.json`. The repo does not call the packaging script directly, and does not copy the tooling's guide into its own docs; it links to the tooling's README (the stale copies are FIX-037 and FIX-038). The target architecture is passed as `-- --platform <arch>`; it is not baked into the script. None of the four commits, pushes or publishes.

The wrapper lives only in winui-app-kit; apps call `external/winui-app-kit/scripts/windows-build` from their npm scripts and keep no copy (decided 2026-11-16). It takes the app's root from the directory npm runs it in, not from its own path, since inside a submodule its own path points at the kit. A repo that needs a step before staging (persistent's localization preparation) runs it in its own npm script before calling the wrapper. The tooling has its own GitHub repo, Dynamic-Solutions-Canada/windows-build-tools (D-005). The absolute host path in one consumer is FIX-049.

### SCRIPT-7: Web entry points go through devkit

- **Status:** Existing (3 of 4 web monorepos: persistent, printstream, game-is-up; store-sign-controller is not on devkit). Who owns the copies: Decided (D-045)
- **Applies to:** Web
- **Reference:** `persistent/package.json`, `persistent/paseo.json`
- **Why:** The dev stack needs ports, hostnames and a database per checkout. Devkit provides them, and the same eight names make every web repo start the same way.

A web monorepo defines `dev` and the seven lifecycle names in the table, each calling devkit. What each one does is documented in devkit's README and is not repeated in the app (see [shared-kits.md](shared-kits.md)).

`paseo.json` mirrors `dev`, `validate`, `test` and `build` and the lifecycle scripts by calling the npm script, so the two never disagree. A repo may add project-specific `dev:*` helpers (`dev:db`, `dev:demo`) but does not rename the eight.

Under D-045, devkit generates these scripts, `paseo.json` and the runner from templates, so they stop being hand-copied.

### SCRIPT-8: Database scripts

- **Status:** Existing (4 of 4 have `db:generate` and `db:migrate`; 3 of 4 agree on what `db:migrate` does)
- **Applies to:** Web
- **Reference:** `game-is-up/package.json` (all six names)
- **Why:** `db:migrate` creates a migration in three repos and applies existing ones in the fourth. Running the wrong one against the wrong database is not recoverable.

| Script | Does |
| --- | --- |
| `db:generate` | Generates the database client |
| `db:migrate` | Development only: creates and applies a migration from schema changes |
| `db:migrate:deploy` | Applies existing migrations; the only migrate command used in production and CI |
| `db:seed` | Loads development seed data; variants are `db:seed:<name>` |
| `db:validate` | Validates the schema; part of `validate` |
| `db:wait` | Waits until the database accepts connections |

### SCRIPT-9: Android entry points wrap Gradle

- **Status:** Decided (D-044). Proposed (0 of 2 native Android repos)
- **Applies to:** Android
- **Reference:** `hub-display-switch/package.json` for the wrapper pattern; `sonarbridge/android/README-dev.md` for the Gradle commands
- **Why:** Today one repo has a root shell script with a hard-coded tool path and no test or lint path, and the other documents Gradle commands in prose. Neither can be called by name.

Each native Android repo has a root `package.json` (SCRIPT-1) with:

| Script | Runs |
| --- | --- |
| `verify:android` | The debug compile of every app module |
| `build:android` | A debug build that can be installed |
| `lint` | Android lint |
| `test` | JVM unit tests, plus `node --test` for the repo's scripts |
| `check:l10n` | The translation check |
| `package:android` | The release bundle and APK |
| `install:android` | Installs the debug build on the connected device |

Gradle is called through the committed wrapper (see [platform.md](platform.md)), and builds run in the repo's build container where it has one. Capacitor shells in the monorepos use the same names at the monorepo root; today persistent and printstream use `android:build` and `android:verify`, which SCRIPT-2 renames.

### SCRIPT-10: Deploy scripts

- **Status:** Existing (2 of 4 web monorepos have `deploy:prod` under that name: persistent, printstream)
- **Applies to:** Web
- **Reference:** `persistent/package.json`, `persistent/scripts/deploy/`; `phonelock/tools/deploy-hub.sh` for the clean-tree refusal (the only script that has it)
- **Why:** Deploying is the one script whose effect reaches users, so its name and its refusal conditions must not vary.

`deploy:prod` and `deploy:staging` deploy the current commit to that environment. The transport is not part of the name. A deploy script refuses to run from a dirty tree or a commit that is not on the remote's `main`, runs the gate first unless an explicit flag skips it, and takes its target from the environment, never from a value in the script. This rule owns what the script checks. When an agent may run it is AI-14 in [ai-docs.md](ai-docs.md): invoking the `deploy` skill authorises that skill's own commit, tag and push (D-013).

### SCRIPT-11: Scripts are thin, portable and safe to call

- **Status:** Proposed
- **Applies to:** all
- **Reference:** `hub-display-switch/scripts/windows-build` (overridable tool path, clear error when the tool is missing)
- **Why:** Scripts are run by agents in containers and worktrees as often as by a person at a prompt, so they cannot assume one machine.

- A `package.json` script is a short call. Logic longer than one command lives in a file under `scripts/` (see [repo-layout.md](repo-layout.md)).
- No script depends on a path in a home directory or on a sibling checkout without an override and a clear error. phonelock's `build.sh` is the counter-example.
- A script exits non-zero on failure and prints what failed. It does not prompt for input when run without a terminal.
- A script that can change something outside the working tree (install, deploy, publish, pointer or key injection) says so in its name or requires an explicit flag. It is never part of `validate` or `test`.
- Heavy scripts follow the memory-limit section of the user-level agent rules (kept outside this repo), which is not restated here.

### SCRIPT-12: Project-specific scripts are allowed and follow the grammar

- **Status:** Existing (printstream and persistent each carry more than 40 root scripts)
- **Applies to:** all
- **Reference:** `printstream/package.json`
- **Why:** The canonical table is a floor. A product with a slicer or a firmware image needs more names, and they should still be guessable.

A repo may add scripts beyond the table. They use `job:target[:variant]` or one of the helper namespaces, and they do not reuse a canonical name for another job. If the same extra script appears in a third repo, it is added to the table here.
