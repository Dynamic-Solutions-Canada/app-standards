# CI

Which GitHub Actions workflows each repo carries, when they run, and how they are written. What the tests are and where they run locally is in [testing.md](testing.md); what a release does after the gate passes is in [releases.md](releases.md); script names such as `validate` and `verify:desktop` are defined in [dev-scripts.md](dev-scripts.md).

Counts are out of the 15 in-scope repos unless a rule says otherwise. Eight of them have workflows today (little-launcher, repilot, immich-drive, persistent, printstream, sonarbridge, printstream-home-assistant, devkit), 45 workflow files in total. The Home Assistant workflows are authored in `printstream/integrations/home-assistant/.github/workflows/` and exported, so changes to them land in printstream.

## Baseline workflow set

The files every repo of a stack carries. Rules below define each column.

| Stack | `ci.yml` (push to `main`, pull requests) | `codeql.yml` | Release workflows | Also |
| --- | --- | --- | --- | --- |
| Windows | Linux job: localization check and every portable test harness. Windows job: compile for x64 and ARM64 (CI-3) | `csharp` and `actions`, buildless, on `ubuntu-latest` | `build-msix.yml` and `store-publish.yml` on `v*` tags, both gated (CI-4) | `dependabot.yml` (nuget, github-actions); credential-expiry reminder |
| Web | `npm ci`, Postgres service where the app uses one, build, `validate`, integration tests | `javascript-typescript` | Deploy stays a local script; any publishing workflow is gated | `dependabot.yml` (npm grouped, github-actions) |
| Android | Gradle lint and JVM unit tests, plus `node --test` for repo scripts | optional (`java-kotlin`) | `release.yml` on `v*` tags calling a reusable `play-publish.yml`, gated | `play-check.yml`; `dependabot.yml` (gradle, github-actions) |
| HA | hassfest and HACS validation in the exported repo; Python tests in printstream's CI before export | none | `release.yml` on a manifest version change | weekly schedule on validation |
| Kit | devkit: `npm test`. winui-app-kit: Windows compile of the library. apps-promo: its catalog test | optional | on tag | `dependabot.yml` (github-actions) |

## Rules

### CI-1: Every repo has a `ci.yml` that runs the local gate

- **Status:** Existing (2 of 15: printstream, devkit)
- **Applies to:** all
- **Reference:** `printstream/.github/workflows/ci.yml`; `devkit/.github/workflows/ci.yml` for the smallest form
- **Why:** A check that runs only on some paths, or only on a tag, lets broken code reach `main`. Running the same command locally and in CI means a green local gate predicts a green CI run.

Each repo has one workflow file named `ci.yml`, display name "CI", triggered by every push to `main` and every pull request, with no path filter on the workflow as a whole. It runs the same gate a developer runs locally (SCRIPT-4 in [dev-scripts.md](dev-scripts.md)), not a separate list of commands kept in the workflow. Extra path-filtered workflows for expensive targets are allowed beside it, never instead of it: persistent runs `validate` only when Linux desktop paths change, so an API-only change is never checked.

### CI-2: Baseline workflow set per stack

- **Status:** Proposed (no repo carries its stack's full set; printstream is closest)
- **Applies to:** all
- **Reference:** the table above; `printstream/.github/` for Web, `immich-drive/.github/` for Windows release workflows, `sonarbridge/.github/` for Android release workflows
- **Why:** A fixed set per stack makes a missing check visible as a missing file, and lets one repo's workflow be compared line by line with its sibling's.

A repo carries the workflows in its stack's row of the table, under those file names (CI-9). A repo with no release yet still carries `ci.yml`. Product-specific workflows (store listing sync, package retention, review repair) are added beside the baseline, not folded into it.

### CI-3: Windows compile on pull requests

- **Status:** Decided (D-024). Existing (1 of 6 WinUI-building repos: persistent)
- **Applies to:** Windows, Kit (winui-app-kit)
- **Reference:** `persistent/.github/workflows/build-desktop-msix.yml` (its header comment explains why)
- **Why:** XAML and packaging breaks cannot be caught in a Linux container, and today they surface only on a release tag or in a consuming app.

The Windows compile job (`windows-2025`, x64 and ARM64) runs on pull requests and pushes that touch the app's source, packaging or build configuration paths, while the Linux job in `ci.yml` (localization check, portable harnesses) runs on every pull request. The path filter keeps Windows runner minutes down, a recurring cost accepted in D-024. winui-app-kit needs the same compile job, because a break there otherwise appears only in an app that consumes it.

### CI-4: Publishing workflows run the gate first

- **Status:** Existing (1 of 7 repos with a publishing workflow gates every lane: immich-drive; 3 more gate one lane: repilot's MSIX build, persistent's Linux release, printstream's Play upload)
- **Applies to:** all
- **Reference:** `immich-drive/.github/workflows/store-publish.yml` (tests before submission); `printstream/.github/workflows/android-play.yml` (lint and unit tests before the bundle); `persistent/.github/workflows/release-linux.yml`
- **Why:** A tag push is the one moment code reaches users, and most tag workflows today build and publish without running the repo's tests.

Any workflow that publishes (a GitHub Release, a store submission, a container image, a package) either runs the gate as its first job or declares `needs:` on a job that does. A workflow triggered by a push to `main` does not rely on `ci.yml` having passed separately; it gates itself. The release-side view of this rule, including the notes-file check, is in [releases.md](releases.md).

### CI-5: Permissions, timeouts and concurrency on every workflow

- **Status:** Existing (3 of 45 workflow files have all three; top-level `permissions` in 32 of 45, `timeout-minutes` in 4, `concurrency` in 14). FIX-010 covers the three files with no `permissions` block at all
- **Applies to:** all
- **Reference:** `printstream/.github/workflows/android-ci.yml`
- **Why:** The default token scope is wider than any check needs, a hung job burns minutes for hours, and two release runs racing each other can publish twice.

Every workflow file has:

- A top-level `permissions:` block set to `contents: read`. A job that needs more (creating a Release, uploading scan results) widens it at job level only.
- `timeout-minutes` on every job, with a comment when the cap is not obvious.
- A `concurrency` group. Checks cancel the run in progress; anything that publishes never cancels.

### CI-6: Tool versions come from the repo's pin

- **Status:** Existing (Node from `.nvmrc`: 1 of 3 repos whose workflows set up Node, printstream; .NET from `global.json`: 3 of 4, little-launcher, repilot, persistent). FIX-050 and FIX-057 cover the known mismatches
- **Applies to:** all
- **Reference:** `printstream/.github/workflows/ci.yml` (`node-version-file`); `repilot/.github/workflows/build-msix.yml` (`global-json-file`)
- **Why:** A version typed into a workflow drifts from the one developers use, and CI then tests a toolchain nobody runs.

Workflows read the Node version from `.nvmrc`, the .NET SDK from `global.json`, and Gradle from the committed wrapper. No workflow hard-codes a toolchain version. Which versions the pins hold is in [platform.md](platform.md). Runner images are named explicitly for builds (`windows-2025` for MSIX work); buildless scans use `ubuntu-latest`.

Android jobs cache Gradle's downloads with `gradle/actions/setup-gradle` (pinned by commit SHA, `cache-provider: basic`, the MIT provider on the standard Actions cache), placed after setup-java and setup-android, in CI and in the release bundle job. Without it every run fetches the whole Kotlin, AGP and AndroidX tree from Maven Central, which rate-limits shared CI addresses (HTTP 429, sometimes reported as "Could not find"; seen in persistent 2026-11-19). Only the default branch, or the branch the job itself runs on when that is not the default (printstream's Android CI runs on main while dev is the default), writes the cache; tags and pull requests restore it. Reference: `persistent/.github/workflows/ci.yml`.

Runner images move too: `ubuntu-latest` moved to Ubuntu 26 from 2026-10-19. A job that depends on a tool preinstalled on the image, rather than installed by a setup action, names the image version (`ubuntu-26.04`) or installs the tool itself.

`actions/setup-node` turns npm caching on by itself when `package.json` has a `packageManager` field, and then fails if there is no lockfile. A repo whose `package.json` only wraps scripts and has no dependencies (the Android and Windows repos) sets `package-manager-cache: false` on that step, with a comment.

### CI-7: Action pinning and Dependabot for actions

- **Status:** Decided (D-046). Existing (Dependabot `github-actions` entry in 1 of 8 repos with workflows: printstream; SHA pins in 0 of 8). FIX-009 and FIX-052 cover the floating branches and the unwatched satellite directory
- **Applies to:** all
- **Reference:** `printstream/.github/dependabot.yml` for the Dependabot entry; the out-of-scope `paseo-shared-browser/.github/workflows/ci.yml` for SHA pins with a version comment
- **Why:** Third-party actions run with access to signing and store secrets, and a moved tag changes what runs without any commit in the repo.

- Third-party actions (store publishers, release creators, Android and Home Assistant helpers, signing actions) are pinned to a full commit SHA with the version in a trailing comment. An action whose project publishes no current release and documents a branch instead (hacs/action, home-assistant/actions) is pinned to that branch head's SHA with the branch and date in the comment (`# main, 2026-06-08`), refreshed by hand, rather than to an old release tag that would downgrade the check.
- `actions/*` and `github/*` actions are pinned to a major tag.
- No action is referenced by a branch.
- Each action appears at one version per repo.
- Every repo with workflows has a Dependabot `github-actions` entry. Its schedule and directory coverage are HYG-8 in [repo-hygiene.md](repo-hygiene.md).

### CI-8: CodeQL

- **Status:** Existing (4 of the 9 Windows and Web product repos: little-launcher, repilot, immich-drive, printstream). FIX-028 and FIX-030 cover the wrong branch and the old action version
- **Applies to:** Windows, Web; optional for Android and Kit
- **Reference:** `immich-drive/.github/workflows/codeql.yml` (buildless on the cheaper runner)
- **Why:** It is already present in every repo that ships a paid Store app, and it reviews the workflows themselves as well as the code.

`codeql.yml` runs on push to `main`, on pull requests, and weekly. C# repos scan `csharp` and `actions` in buildless mode on `ubuntu-latest`; web repos scan JavaScript and TypeScript. The trigger branch must be the repo's actual default branch.

A repo that consumes kits as submodules checks them out so calls resolve, but scans only its own code (`paths-ignore: [external/**]` in the CodeQL config); each kit is scanned in its own repo.

### CI-9: Workflow file names and display names

- **Status:** Decided (D-048). Existing (Title Case display names in 4 of 6 repos recorded: little-launcher, repilot, immich-drive, printstream)
- **Applies to:** all
- **Reference:** `repilot/.github/workflows/` (`build-msix.yml` "Build and Package", `store-publish.yml` "Publish to Microsoft Store")
- **Why:** The same job under the same name can be found, compared and copied across repos; today `release.yml` means an Android release in two repos and a GitHub Release in a third.

One file name per job across repos, and a Title Case display name. All eight file names are set by D-048.

| Job | File | Display name |
| --- | --- | --- |
| Push and pull request gate | `ci.yml` | CI |
| Code scanning | `codeql.yml` | CodeQL |
| MSIX build and GitHub Release | `build-msix.yml` | Build and Package |
| Microsoft Store submission | `store-publish.yml` | Publish to Microsoft Store |
| Tag-triggered release for other stacks | `release.yml` | Release |
| Play upload (reusable) | `play-publish.yml` | Publish to Google Play |
| Play track status | `play-check.yml` (2 of 3 repos that have one: persistent, printstream) | Play Check |
| Credential expiry reminder | `credential-expiry.yml` | Credential Expiry |

A monorepo with several lanes adds the lane after the job word (`release-linux.yml`).

### CI-10: Shared workflows

- **Status:** Decided (D-047). Existing (8 of 8 repos with workflows copy them; reuse inside one repo in 2 of 8: sonarbridge, little-launcher)
- **Applies to:** all
- **Reference:** `sonarbridge/.github/workflows/play-publish.yml` (a `workflow_call` workflow used by its own `release.yml`)
- **Why:** The three copies of `store-publish.yml` already differ, and each fix has to be made once per repo.

The baseline workflows for CI, CodeQL, MSIX build and Store publish are reusable `workflow_call` workflows that each repo calls, pinned to a tag. The shared MSIX build workflow signs its packaging check with the per-run throwaway certificate from DIST-2 by default, so no app decides this again; until it exists, `immich-drive/.github/workflows/build-msix.yml` ("Build MSIX" step) is the reference. This is possible because the repo that hosts them is public (D-004), so every caller can read them, the public repos included. A repo does not keep a hand-edited copy of a baseline workflow; a needed difference becomes an input.

### CI-11: Store credentials in workflows

- **Status:** Existing (shared secret names in 4 of 5 repos that publish to the Microsoft Store and 3 of 3 that publish to Play; expiry reminder in 2 of 5 repos that use expiring store credentials: persistent, printstream)
- **Applies to:** Windows, Web, Android repos that publish to a store
- **Reference:** `repilot/.github/workflows/store-publish.yml` (Store secret names); `sonarbridge/.github/workflows/release.yml` (Play secret names); `persistent/.github/workflows/store-secret-expiry.yml` (reminder)
- **Why:** One set of names lets a workflow move between repos unchanged, and an expired credential otherwise shows up as a failed release.

Workflows that publish to a store use the secret names already shared by the reference workflows, and never a repo-specific name for the same credential. Every Microsoft Store app publishes through the one shared Entra app registration ("MS Store Publisher"), so the client and tenant IDs are the same in every repo and a new Store app reuses them rather than registering its own. The registration has one active client secret, used by every organisation that publishes (Dynamic-Solutions-Canada and PrintStreamApp), so a rotation is one new secret. All the apps are in one Partner Center account, so the seller ID is the same everywhere too. In Dynamic-Solutions-Canada the four credentials and the `STORE_SECRET_EXPIRES` variable (ISO date) are organisation secrets and an organisation variable, granted to the Store repos, and repos hold no copies (a repo-level secret of the same name would override them). Entra descriptions cannot be edited later, so a new secret's description names its purpose when it is created. A new secret is saved under the user-level credential rules when it is created, since Entra shows the value only once, and the old secret is deleted once a Store run has succeeded on the new one. Any repo whose workflows use a credential that expires carries a weekly reminder workflow that fails or opens an issue before the expiry date. How a secret may be used inside a job is SEC-10 in [security.md](security.md); how credentials are stored and rotated is in the user-level agent rules (kept outside this repo).
