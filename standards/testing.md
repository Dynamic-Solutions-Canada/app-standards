# Testing

Which test frameworks each stack uses, where tests live, the minimum each stack must have, and where each kind of test runs. What CI runs, and when, is in [ci.md](ci.md); script names are defined in [dev-scripts.md](dev-scripts.md).

Counts are out of the 15 in-scope repos unless a rule says otherwise. The audit recorded tests as "present and wired" or "present and unwired"; nothing was run, so no count here means "passing".

## Overview

| Stack | Framework | Where tests live | Local entry point |
| --- | --- | --- | --- |
| Windows | Framework-free .NET console harness, run with `dotnet run` | `tests/<App>.<Area>.Tests/` | `test`; `verify:desktop` for the compile check |
| Web | `node --test` with `tsx`; Testing Library and jsdom for components | `*.test.ts(x)` beside the code | `test`, `test:integration`, `validate` |
| Android | JUnit, with Robolectric where resources matter | the module's `src/test` | `test`; `verify:android` for the compile check |
| HA | Python `unittest`, in the parent repo | `integrations/home-assistant/tests/` in printstream | printstream's gate |
| Kit | `node --test` (devkit, apps-promo); compile check (winui-app-kit) | `test/` or beside the code | `test` |

## Rules

### TEST-1: JavaScript and TypeScript tests use `node --test`

- **Status:** Existing (8 of 8 repos with JavaScript or TypeScript tests)
- **Applies to:** Web, Kit, and repo scripts in any stack
- **Reference:** `persistent/scripts/dev/run-tests.mjs` (discovery) and any `*.test.ts` beside its source in `persistent/apps/api/`
- **Why:** It is already universal, needs no dependency beyond `tsx`, and runs the same on the host, in the devkit container and in CI.

Tests use Node's built-in runner, with `tsx` for TypeScript, and Testing Library with jsdom for React components. Test files are named `*.test.ts`, `*.test.tsx` or `*.test.mjs` and sit beside the code they test. No repo adds vitest, jest or Playwright without a recorded reason.

### TEST-2: .NET tests are portable console harnesses

- **Status:** Decided (D-051). Existing (5 of 5 repos with .NET tests: little-launcher, repilot, immich-drive, persistent, store-sign-controller; a README per harness in 2 of 5)
- **Applies to:** Windows, and Windows hosts inside Web monorepos
- **Reference:** `little-launcher/tests/AGENTS.md`; `little-launcher/tests/LauncherItemReorder.Tests/README.md`
- **Why:** A harness with no framework and no Windows-only target runs from WSL and on a Linux runner, so policy code is tested without a Windows machine.

Each harness is an `OutputType=Exe` project under `tests/<App>.<Area>.Tests/`, run with `dotnet run` (the root `test` script discovers and runs every harness; winui-app-kit's `scripts/run-tests` with `scripts/dotnet`, which uses the host SDK or the pinned SDK container, is the reference), that links the production source files it tests and adapts native boundaries so it runs on Linux. It exits non-zero on failure. Each harness has a short README saying what it covers and what it does not prove (anything that needs a real window, the installed package, or the shell). A root `test` script runs every portable harness. No xunit, NUnit or MSTest.

### TEST-3: Android tests are JVM unit tests; the emulator is optional

- **Status:** Decided (D-052). Existing (JVM unit tests in 4 of 4 Android-building repos: phonelock, sonarphone-nmea-bridge, printstream, persistent; emulator instrumentation in CI in 1 of 4: printstream)
- **Applies to:** Android, and Capacitor apps inside Web monorepos
- **Reference:** `sonarphone-nmea-bridge/android/app/src/test/` (JUnit with Robolectric); `printstream/scripts/ci/run-android-instrumentation.sh` for the optional emulator job
- **Why:** JVM tests are fast and stable enough to gate every push; emulator jobs are slow to run and costly to keep green, so they are worth it only where the app has native behaviour to protect.

The required minimum is JUnit tests under the module's `src/test`, with Robolectric where resources or framework classes matter, plus Gradle lint. Emulator instrumentation in CI is optional and printstream is the reference for it. Anything that depends on real hardware (radios, lock state, another device) is checked on a phone before a release (TEST-9).

### TEST-4: The Home Assistant integration is tested in the parent repo

- **Status:** Decided (D-084). Proposed (the two Python tests in printstream are run by no script or workflow)
- **Applies to:** HA
- **Reference:** `printstream/integrations/home-assistant/tests/` (contract fixtures generated from the product and checked against their producer)
- **Why:** The public repo is a generated snapshot, so the only place a failing test can stop a bad release is before the export.

The integration's Python tests (`unittest`) and ruff run in printstream's gate and in its CI, and the export refuses to run when they fail. Tests and `pyproject.toml` stay in the parent and are not exported, because the public repo holds the generated bundle only (D-015). The exported repo keeps hassfest and HACS validation (see [ci.md](ci.md)).

### TEST-5: A root `test` script that discovers tests

- **Status:** Decided (D-050). Existing (2 of 4 Web monorepos use a discovery script: persistent, printstream). FIX-054 and FIX-055 cover the two repos with hand-written globs
- **Applies to:** all stacks with tests
- **Reference:** `persistent/scripts/dev/run-tests.mjs`; `printstream/scripts/dev/run-tests.mjs` as the scale exception
- **Why:** An unquoted `**` glob matches one directory level, so a test added one folder deeper is skipped without any error.

Every repo with tests has a root `test` script that finds them by walking the tree (or by quoted globs), never by an unquoted `**` or a hand-kept list of folders. The standard for Web repos is persistent's small discovery script, shipped from devkit; printstream keeps its own runner (result cache, batching, flake isolation) because of its size. Discovery skips `node_modules`, build output and `external/` submodules, so a kit's tests run in the kit's repo and not by accident in a consumer.

### TEST-6: `validate` runs everything the repo has

- **Status:** Retired. Merged into SCRIPT-4 in [dev-scripts.md](dev-scripts.md), which owns what `validate` runs in each stack and the rule that every test and checker in a repo is reachable from it.

### TEST-7: Database-backed tests never skip silently

- **Status:** Decided (D-049). Existing (1 of 4 Web monorepos splits them into `test:integration`: store-sign-controller; printstream makes the database mandatory in `validate`; persistent skips them when an environment variable is absent; game-is-up has none wired)
- **Applies to:** Web
- **Reference:** `store-sign-controller/package.json` (`test:integration`); `printstream/.github/workflows/ci.yml` (Postgres service)
- **Why:** A test that skips when its database is missing reports success for work that was never checked.

Tests that need a database live behind `test:integration`, so the local `validate` (SCRIPT-4 in [dev-scripts.md](dev-scripts.md)) stays fast and needs nothing provisioned. CI runs both `validate` and `test:integration` against a Postgres service. A database test with no database fails; it does not skip.

### TEST-8: Minimum testing per stack

- **Status:** Proposed (no stack has every repo at the minimum)
- **Applies to:** all
- **Reference:** `immich-drive/tests/CloudFilterLayout` (Windows), `persistent/` (Web), `sonarphone-nmea-bridge/android/` (Android), `devkit/test/` (Kit)
- **Why:** A stated floor turns "has some tests" into something a parity check can answer yes or no.

| Stack | Minimum | Runs in CI |
| --- | --- | --- |
| Windows | The localization check (see [locales.md](locales.md)); at least one portable harness for the app's core non-UI policy; `verify:desktop` before finishing a task; a written manual acceptance list for installed-package behaviour | Check, harnesses and compile: yes. Installed-app and pointer tests: never |
| Web | Root `test` and `validate`; unit tests for shared contracts and API logic; database tests per TEST-7 | Yes, with a Postgres service |
| Android | JVM unit tests for protocol and state logic; Gradle lint; `verify:android`; a device or emulator smoke run before a release | Unit tests and lint: yes. Emulator: optional |
| HA | Contract tests against generated fixtures, in the parent | Yes, in printstream |
| Kit | devkit and apps-promo: `node --test`. winui-app-kit: a compile check, and a harness once it holds non-UI policy | Yes, in the kit's own repo |

### TEST-9: Where each kind of test runs

- **Status:** Proposed (the ladder is practised but no in-scope repo describes all of it)
- **Applies to:** all
- **Reference:** `windows-build-tools/README.md` and `android-vm/README.md` (both out of scope) for the two virtual machines; `devkit/README.md` for the container
- **Why:** Running a check one rung too high wastes a Windows or Android machine; running it one rung too low proves nothing.

Use the lowest rung that can prove the thing:

| Rung | What runs there |
| --- | --- |
| Linux host or the devkit container | Web tests and `validate`; portable .NET harnesses; localization checks; script tests |
| Builder container | Gradle builds, lint and JVM unit tests for Android |
| Windows (host toolchain or the Windows build VM, through `scripts/windows-build`) | XAML compile (`verify:desktop`), packaging, install of the dev package, guarded native UI suites |
| Android VM or emulator | UI checks and screenshots that need a running Android system but no real hardware |
| Physical device | Hardware paths, pairing, lock and power behaviour, and the final check before a store release |

immich-drive supports Windows 10 22H2 (D-018, PLAT-6 in [platform.md](platform.md)), so its Windows rung includes a Windows 10 22H2 machine: every immich-drive release is installed and exercised there before it ships. When no Windows 10 machine or VM run is available, the maintainer may accept other evidence for a release (for example a user report on the published version); the decision and exactly what it covers are recorded in the repo's `docs/development.md` "Compatibility verification" section, and the release skill cites that record instead of skipping the step.

The Windows build VM and the Android VM are never run at the same time; both READMEs say so. Memory caps for builds, emulators and test runs follow the user-level agent rules (kept outside this repo) and are not restated here. Each repo's development doc names which rungs it uses (see [docs.md](docs.md)); it links the tool READMEs instead of copying their setup steps.

### TEST-10: Native UI tests are guarded and never run by default

- **Status:** Existing (2 of 2 repos with such suites: little-launcher, repilot)
- **Applies to:** Windows; any stack with tests that drive a real desktop or device
- **Reference:** `little-launcher/tests/native-drag/README.md`; `repilot/tests/Repilot.InputTests/README.md`
- **Why:** A test that moves the pointer, injects keys or replaces settings can damage the session of whoever is using the machine.

A suite that moves the pointer, sends input, or replaces user settings refuses to start unless it is on the expected machine and has an explicit opt-in. It is never part of `test`, `validate` or CI, and it works on a disposable copy of settings. Its README states the guard and how to run it deliberately.

### TEST-11: Shared kits are tested in their own repo

- **Status:** Existing (1 of 3: devkit; winui-app-kit has no tests or CI, and apps-promo's one test runs only through a consumer's discovery)
- **Applies to:** Kit
- **Reference:** `devkit/test/` with `devkit/.github/workflows/ci.yml`
- **Why:** Several apps depend on each kit, and a kit with no check of its own breaks in whichever app updates its pin first.

Each kit has its own `test` entry point and its own `ci.yml`. winui-app-kit's minimum is a Windows compile of the library (see [ci.md](ci.md) CI-3); apps-promo runs its catalog test itself and checks the catalog data when it changes. Consumers do not rely on walking into `external/` to run a kit's tests (TEST-5).
