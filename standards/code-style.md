# Code style

How source code is formatted, linted and named in each stack, and which tool checks it. Today style is consistent by habit and checked almost nowhere, so most rules here describe an existing habit that D-016 now enforces, rolled out repo by repo with printstream on its own schedule.

Counts are against the in-scope repos a rule applies to: the 6 repos with a WinUI app on the shared build tooling (hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, persistent), the 4 web monorepos (persistent, printstream, game-is-up, store-sign-controller), and the 2 Android repos (phonelock, sonarbridge).

How code is commented, documented and split into functions and files is covered by the user-level agent rules (kept outside this repo) (code clarity and structure, conventions, writing) and is not restated here.

## Tools per stack

| Stack | Formatter | Linter and analyzers | Config files at the repo root | Run by |
| --- | --- | --- | --- | --- |
| Windows (C#, XAML) | `dotnet format` | .NET analyzers, code style in build | `.editorconfig`, shared build props (STYLE-7) | `format`, `format:check`, `lint` |
| Web (TypeScript) | Prettier | ESLint flat config, `tsc` | `.editorconfig`, `.prettierrc.json`, `eslint.config.mjs`, `tsconfig.base.json` | `format`, `format:check`, `lint`, `typecheck` |
| Android (Kotlin, Java) | Spotless running ktlint (STYLE-8, D-092) | Android lint | `.editorconfig`, `gradle.properties` | `format`, `format:check`, `lint` |
| HA (Python) | ruff | ruff | `pyproject.toml` in the printstream source folder | printstream's gate |
| Kit | The tool for its language, as above | As above | As above | As above |

The script names are defined in [dev-scripts.md](dev-scripts.md). Where they run in CI is in [ci.md](ci.md).

## Rules

### STYLE-1: One `.editorconfig` at every repo root

- **Status:** Decided (D-016). Proposed (0 of 15 have a root `.editorconfig`; printstream has a nested one for C# only)
- **Applies to:** all
- **Reference:** `printstream/apps/desktop/.editorconfig` (C# section only); no root example exists
- **Why:** It is the one file every editor, formatter and agent reads, so it is where the basics that all stacks share are written once.

The root `.editorconfig` sets `root = true` and, for every file: UTF-8, LF line endings, a final newline, and no trailing whitespace. Indentation is four spaces for C#, XAML, Kotlin, Java, Python, PowerShell and shell, and two spaces for TypeScript, JavaScript, JSON, YAML, CSS and Markdown. Tabs are used only where a format requires them. Language sections carry the settings in STYLE-5 and STYLE-8.

The canonical text is kept in one place and reaches each repo by the mechanism chosen in D-005 (see [shared-kits.md](shared-kits.md)). A repo does not hand-edit its copy.

### STYLE-2: Formatting is checked by a tool, not by habit

- **Status:** Decided (D-016). Proposed (0 of 15 enforce a formatter)
- **Applies to:** all
- **Reference:** `store-sign-controller/.prettierrc.json` (the config; nothing runs it yet, see FIX-056)
- **Why:** Style that is only a habit drifts, and review time is spent on whitespace. little-launcher already shows the drift (FIX-032).

A formatter is enforced in every stack, using the tools in the table above (D-016). Enforcement is rolled out one repo at a time, each taking its one-off reformat commit when it adopts the check; printstream does so on its own schedule. Each repo exposes `format` (rewrites files) and `format:check` (fails on a difference), and `format:check` is part of the `validate` gate (see [dev-scripts.md](dev-scripts.md)).

For Windows repos, `format:check` runs `dotnet format <project> --verify-no-changes` on Linux with `EnableWindowsTargeting=true`, using the host SDK when present and otherwise the `mcr.microsoft.com/dotnet/sdk` image at the `global.json` version, under a memory cap. Reference: winui-app-kit's `scripts/dotnet`. Against an app (not a library) the workspace loads with warnings and code-behind may be only partly checked, so the Windows compile in CI (CI-3) remains the backstop. An app that imports shared-items code from a submodule (apps-promo's `.projitems`, the kit's `Xaml/`) passes `--exclude external/`, otherwise `dotnet format` rewrites the submodule's files (KIT-8); reference: immich-drive's `scripts/format`. The one-off reformat commit is listed in `.git-blame-ignore-revs` so blame skips it.

A repo adopts the formatter in one commit that contains only the reformat, with no behaviour change, so it can be skipped in blame and reviewed by its size alone. New and small repos take that commit now. printstream takes it on its own schedule because of its size.

### STYLE-3: Web lint uses the shared ESLint flat config

- **Status:** Existing (3 of 4 web monorepos: persistent, printstream, game-is-up)
- **Applies to:** Web, and Kit repos written in JavaScript or TypeScript
- **Reference:** `persistent/eslint.config.mjs`
- **Why:** The three configs already share one skeleton; naming it stops a fourth variant.

`eslint.config.mjs` at the root, flat config, built from: the JavaScript recommended set, the `typescript-eslint` recommended set, Node globals for `.js`, `.mjs` and `.cjs` files, and React hooks and react-refresh rules scoped to `apps/web/**`. It adds unused-variable checking with a `^_` ignore pattern and `no-alert` for web code. store-sign-controller and devkit have no lint today.

The React hooks set is eslint-plugin-react-hooks 7 or later, with its compiler rules `react-hooks/refs`, `react-hooks/set-state-in-effect` and `react-hooks/purity` at `error`. State that follows a prop is reset during render (`useResetOnChange` from web-app-kit), the latest value or callback an effect or listener reads comes through React's `useEffectEvent` on React 19 (a ref written in `useLayoutEffect`, never during render, on React 18), an invalid stored choice is derived rather than overwritten, and the clock is read in render through one shared `useNow` store and in handlers through `Date.now()`. A waiver is a one-line disable with its reason, only for a real sync with an external system. Reference: `game-is-up/eslint.config.mjs` and `game-is-up/apps/web/AGENTS.md`.

A product may add rules that encode its own UI conventions, as `printstream/eslint.config.mjs` does (no native confirm or prompt, icon decorators). The conventions themselves are in [ui.md](ui.md). Formatting rules do not go in ESLint once Prettier is in place (STYLE-2).

### STYLE-4: Web house style and the shared TypeScript base

- **Status:** Existing (4 of 4 follow the style by habit; 2 of 4 share an identical `tsconfig.base.json`: persistent, printstream)
- **Applies to:** Web
- **Reference:** `store-sign-controller/.prettierrc.json`, `printstream/tsconfig.base.json`
- **Why:** One style and one compiler strictness level let code move between products without rework.

The style is single quotes, no semicolons, two spaces, and Markdown prose left unwrapped (`proseWrap: preserve`). The Prettier config is store-sign-controller's file.

Every monorepo has a root `tsconfig.base.json` that workspaces extend. It is the printstream and persistent file: ES2022, bundler module resolution, `strict`, `noUncheckedIndexedAccess`, `noImplicitOverride`, `noFallthroughCasesInSwitch`, `isolatedModules`, `verbatimModuleSyntax`, `forceConsistentCasingInFileNames`. DOM libraries are added by `apps/web`, not by the base. store-sign-controller is one option short and game-is-up uses an older variant.

### STYLE-5: C# style

- **Status:** Existing (6 of 6 by habit, and winui-app-kit; nothing enforces it)
- **Applies to:** Windows, Kit
- **Reference:** `wsl-monitor/external/winui-app-kit/WinUIAppKit/`
- **Why:** The family already writes C# one way; recording it lets the `.editorconfig` and `dotnet format` hold it.

- File-scoped namespaces.
- Four-space indentation, braces on every block.
- Private fields are `_camelCase`, including mutable private static fields. Private `static readonly` fields are PascalCase, like constants (`Logger`, `LatestReleaseUri`). No `s_` prefix. Types, methods, properties and constants are PascalCase.
- MSBuild and manifest XML (`.csproj`, `.props`, `.targets`, `.projitems`, `.appxmanifest`, `.resw`, `.config`, `.nuspec`) use 2-space indentation; XAML keeps 4 (STYLE-1).
- Namespaces match the project name and folder path, with no company prefix.

These go in the `.editorconfig` (STYLE-1) so that `dotnet format` applies them. The static field rules are two naming rules, not an exemption: `private static readonly` fields match PascalCase, other private fields (static or not) match `_camelCase`. Decided 2026-11-16 from the code: `_camelCase` is the majority form for mutable private statics (62 of about 103 across the Windows repos; `s_` appeared only in winui-app-kit and once in wsl-monitor), PascalCase was already the form for `static readonly`, and all 36 MSBuild and manifest files already used 2-space indentation.

### STYLE-6: Nullable and language settings are on in every .NET project

- **Status:** Existing (6 of 6 main app projects and winui-app-kit set all three; 4 of 6 test projects omit `LangVersion`)
- **Applies to:** Windows, Kit
- **Reference:** `wsl-monitor/WslMonitor/WslMonitor.csproj`
- **Why:** A project without nullable checking can pass a null to one that has it, and the mismatch surfaces as a crash, not a warning.

Every project, including helper executables and test harnesses, has `Nullable` enable, `ImplicitUsings` enable and the same `LangVersion` as the app (`preview` in all six today). Set them once in `Directory.Build.props` so new projects inherit them, and do not repeat them per csproj. Nullable warnings are fixed, not suppressed; a `!` or `#nullable disable` carries a comment saying why.

### STYLE-7: Analyzers on, and no new warnings

- **Status:** Decided (D-016, D-091, D-005). Proposed (0 of 6 set `AnalysisLevel`, `EnforceCodeStyleInBuild` or `TreatWarningsAsErrors`)
- **Applies to:** Windows, Kit
- **Reference:** `wsl-monitor/external/winui-app-kit/WinUIAppKit/WinUIAppKit.csproj` (documentation warnings on public members, the only compiler-checked style rule in the family)
- **Why:** The compiler is the only reviewer that reads every line on every build.

An app that is not yet warning-free sets `WinUIAppKitWarningsAsErrors=false` in its `Directory.Build.props` before the kit's props import (it is evaluated at import time and ignored if set after), with a comment saying why, and removes it once the build is clean; that switch is how "once a repo builds without warnings" below is put into practice for apps on the kit.

The shared build props turn on the .NET analyzers and `EnforceCodeStyleInBuild`, so an `.editorconfig` violation shows up in a normal build. A change does not add warnings: a build that was clean stays clean. Once a repo builds without warnings, its CI and Release builds set `TreatWarningsAsErrors`; Debug builds leave it off so work in progress still compiles. A warning that cannot be fixed is suppressed at the narrowest scope with a reason.

The shared props pin `AnalysisLevel` to the SDK's major (`10.0` today), so a newer SDK feature band cannot add warnings that become errors. Build props of a project apply wherever it is built: when an app builds winui-app-kit through a project reference, the kit's warnings-as-errors fires inside the app's Release build, so a kit tag is warning-free before apps take it (KIT-5).

A library with a public API (winui-app-kit) also sets `GenerateDocumentationFile`, so undocumented public members are reported.

Warnings are errors in CI and Release builds (D-091).

### STYLE-8: Kotlin and Java style

- **Status:** Existing (2 of 2 set `kotlin.code.style=official`). The formatter is Decided (D-016, D-092) (0 of 2)
- **Applies to:** Android
- **Reference:** `sonarbridge/android/gradle.properties`; `phonelock/companion/build.gradle` (lint block)
- **Why:** The two repos share one formatting setting today and differ in line length and density; a formatter settles it without debate.

`kotlin.code.style=official` in `gradle.properties`, four spaces, and one Kotlin formatter wired into Gradle and exposed as `format` and `format:check`. The formatter is Spotless (D-092), running ktlint for Kotlin and a Java formatter for Java, set up the same way in both repos.

Android lint runs as `lint` and is part of the gate. The translation checks that phonelock sets to fatal are covered in [locales.md](locales.md). New Android code is Kotlin; phonelock's Java lock app is the existing exception.

### STYLE-9: Python style for the Home Assistant integration

- **Status:** Decided (D-084). Proposed (no Python tooling today)
- **Applies to:** HA
- **Reference:** none in scope; Home Assistant core's ruff setup is the model
- **Why:** Matching the host ecosystem's tool keeps the integration reviewable by its community at the lowest upkeep.

The toolchain is ruff alone, for both formatting and lint, configured in a `pyproject.toml` beside the source in `printstream/integrations/home-assistant/` and run in printstream's gate before export. The config is not exported: the public repo keeps the generated bundle only (D-015).

### STYLE-10: UTF-8 without a byte-order mark, LF line endings

- **Status:** Retired. Merged into HYG-7 in [repo-hygiene.md](repo-hygiene.md), which owns `.gitattributes`, encoding, line endings and the executable bit. The matching editor settings are STYLE-1.

### STYLE-11: Names of repos, projects and packages

- **Status:** Existing (15 of 15 repo folders are kebab-case; 6 of 6 .NET app projects are PascalCase; 4 of 4 monorepos use an `@<app>/*` scope). TypeScript module file names are Decided (D-093)
- **Applies to:** all
- **Reference:** `repilot/` (Windows), `persistent/package.json` (Web)
- **Why:** A name that follows the pattern can be guessed, by people and by scripts that loop over repos.

| Thing | Form | Example |
| --- | --- | --- |
| Repo and its folder | kebab-case | `hub-display-switch` |
| .NET project, assembly, root namespace | PascalCase, no company prefix | `HubDisplaySwitch` |
| MSIX packaging folder | `<App>MSIX` | `RepilotMSIX` |
| npm package name (root) | the repo name | `game-is-up` |
| npm workspace packages | `@<app>/<workspace>` | `@persistent/shared` |
| React component files | PascalCase, named after the component | `EmptyState.tsx` |
| Test files | see [testing.md](testing.md) | |
| Docs | see [docs.md](docs.md) | |

Other TypeScript module files (D-093): camelCase in `apps/web`, kebab-case in `apps/api`, `packages/` and `scripts/`. That is what the repos already do (`apps/web` in 4 of 4; `apps/api` in 3 of 4, store-sign-controller's API being the exception). It applies to new files, and nothing is mass-renamed.

Product display names, publisher names, package identities and Android application ids are not style choices; they are in [branding.md](branding.md) and [distribution.md](distribution.md). GitHub repo names that differ from the folder name are covered by BRAND-4 in [branding.md](branding.md) (D-009).

### STYLE-12: Routed web pages are `*Page.tsx`

- **Status:** Decided (D-039). Existing (2 of 4: persistent, store-sign-controller)
- **Applies to:** Web
- **Reference:** `persistent/apps/web/src/pages/`
- **Why:** The split is even and has no functional meaning; "Page" matches the `Pages/` folders of the Windows apps.

A component rendered by a route is named `<Name>Page.tsx` and lives in `apps/web/src/pages/`. This applies to new files. printstream keeps its `*View.tsx` files and is not mass-renamed.

### STYLE-13: No source file headers

- **Status:** Retired. Merged into HYG-2 in [repo-hygiene.md](repo-hygiene.md), which owns the no-header rule. Writing-rule violations in existing files are FIX-044.
