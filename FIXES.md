# Fix regardless

Concrete defects found by the audit that need no decision from the maintainer. Each row was re-checked read-only against the cited file on 2026-11-16; items that no longer hold were dropped and are listed at the end. Decisions are in `DECISIONS.md`; repo codes are in its legend.

- **Severity.** security: exposes or weakens protection of a credential, key, personal detail or deployment (these rows are kept privately; see "Security" below). user-visible: wrong or broken for someone using the app, its listing or its public repo. developer-facing: wrong for people or agents working in the repo.
- **Waits on.** "independent" means the fix never needed a decision. A D-ID means the correct value comes from that decision. Every decision named in this column was settled on 2026-11-16 (see `DECISIONS.md`), so every row can now be fixed.
- **Paths** are relative to `/development/`. HA rows are fixed in `printstream/integrations/home-assistant/` and reach the public repo through the export.
- No secret file was opened to verify these. Ignore rules were checked with `git check-ignore`, and literal values in flagged files were masked, so rows describe where a value sits, not what it is.

## Security

Security fixes FIX-001 to FIX-010 are tracked privately and are not published with this repo (D-004). The standards cite them by ID and state the control each one restores; see [standards/security.md](standards/security.md).

## User-visible

| ID | Repo | Defect | Evidence | Waits on |
| --- | --- | --- | --- | --- |
| FIX-011 | SNB | The published privacy page and the Play listing draft give a personal email address as the contact, where PL uses the company address. | `sonarbridge/docs/privacy.html`, `sonarbridge/docs/PLAY_LISTING.md` | D-017 |
| FIX-012 | PL | The public privacy policy contains a visible `[TODO]` marker. | `phonelock/PRIVACY.md` (line 56) | independent |
| FIX-013 | LL | The About page "source" link points at the previous personal-account URL for LittleLauncher; the repo is `Dynamic-Solutions-Canada/little-launcher`. | `little-launcher/LittleLauncher/Pages/AboutPage.xaml` (line 61) | independent |
| FIX-014 | PER | The desktop About link and the update checker's owner string still name the previous personal-account URL for persistent after the move to the organisation. | `persistent/apps/desktop/Persistent.Desktop/Pages/AboutPage.xaml.cs` (line 43), `.../Services/UpdateService.cs` (line 44) | independent |
| FIX-015 | REP | The About link and the update checker's owner string name the previous personal-account URL for Repilot; GitHub now resolves the repo to `Dynamic-Solutions-Canada/repilot`, so the app depends on a redirect. | `repilot/Repilot/Pages/AboutPage.xaml` (line 82), `repilot/Repilot/Services/UpdateService.cs` (line 19) | independent |
| FIX-016 | IMD | Same as FIX-015 for the previous personal-account URL for ImmichDrive, now `Dynamic-Solutions-Canada/immich-drive`. | `immich-drive/ImmichDrive/Pages/AboutPage.xaml` (line 103), `immich-drive/ImmichDrive/Services/UpdateService.cs` (line 19) | independent |
| FIX-017 | PROMO | The promo page fetches its catalog from the previous personal-account raw URL for technicallyreal-promo, the pre-rename owner and repo name, and the README title and install command use the same old name. Every shipped app depends on that redirect. The old URL must keep working for installed copies after the fix. | `<app>/external/promo/Views/PromotedAppsPage.xaml.cs` (line 24), `<app>/external/promo/README.md` (lines 1, 13, 18) | independent |
| FIX-018 | PS, HA (in PS) | The bridge package `homepage` and the Home Assistant cards' `documentationURL` point at the previous personal-account URL for printstream; the public repo is `PrintStreamApp/printstream`. Found while verifying; not in the audit reports. | `printstream/apps/bridge/package.json` (line 7), `printstream/integrations/home-assistant/card-source/printstream-cards.js` (lines 1763, 1770, 1777) and the generated bundle | independent |
| FIX-019 | REP | Three README screenshots are JPEG data saved with a `.png` extension. | `repilot/docs/1-keyboard-shortcut.png`, `2-application.png`, `3-windows-function.png` | independent |
| FIX-020 | LL | The package manifest declares only `en-US` while the app ships nine interface languages, so Windows and the Store do not know about the other eight. | `little-launcher/LittleLauncherMSIX/Package.appxmanifest` (line 29) | independent |
| FIX-021 | REP | The README says Windows 11 build 22621 or newer; the manifest's minimum is build 22000. | `repilot/README.md` (line 55), `repilot/RepilotMSIX/Package.appxmanifest` (line 27) | D-018 |
| FIX-022 | PL | The README states platform 36 and Gradle 8.11.1 while the build uses `compileSdk 37`, and it runs `./build.sh`, which is tracked without the executable bit. | `phonelock/README.md` (lines 30, 33), `phonelock/companion/build.gradle` (line 20), `phonelock/build.sh` | independent |
| FIX-023 | PL | The parent app's label is "Phone Lock" where the product is "PhoneLock" everywhere else. | `phonelock/companion/src/main/res/values/strings.xml` (line 6) | D-012 |
| FIX-024 | HA (in PS) | The exported `NOTICE` is PS's own and refers readers to `apps/web`, `apps/api`, `apps/slicer` and `apps/bridge`, none of which exist in the public integration repo. | `printstream-home-assistant/NOTICE` (lines 12, 13); copied by `printstream/scripts/export/export-home-assistant.mjs` (line 90) | independent |
| FIX-025 | HA (in PS) | The README gives two different places to create the access token (under Settings, and from the plugin's panel). | `printstream/integrations/home-assistant/README.md` (lines 19, 35) | independent |
| FIX-026 | HA (in PS) | The local clone of the public repo is at 0.3.40 while the source manifest is at 0.3.41 and the clone's README still names only Bambu Lab printers. Confirm whether the remote already has 0.3.41 (not checked, no network use) and export if not. | `printstream/integrations/home-assistant/custom_components/printstream/manifest.json` (line 20), `printstream-home-assistant/custom_components/printstream/manifest.json` (line 20) | independent |
| FIX-027 | PER, SSC | The web manifest declares one SVG as both `any` and `maskable`, so installed icons are cropped or padded wrongly on Android. | `persistent/apps/web/vite.config.ts` (line 64), `store-sign-controller/apps/web/public/manifest.webmanifest` (line 11) | independent |

## Developer-facing

| ID | Repo | Defect | Evidence | Waits on |
| --- | --- | --- | --- | --- |
| FIX-028 | LL | CodeQL triggers on `master`; the branch is `main`, so the scan never runs on push or pull request. | `little-launcher/.github/workflows/codeql.yml` (lines 16, 18) | independent |
| FIX-029 | LL | A one-off `!v1.42.0` tag exclusion is still in the Store publish trigger. | `little-launcher/.github/workflows/store-publish.yml` (line 12) | independent |
| FIX-030 | IMD | CodeQL uses `codeql-action@v3` where its siblings use `@v4`. | `immich-drive/.github/workflows/codeql.yml` (lines 36, 42) | independent |
| FIX-031 | IMD, REP | Copy-paste leftovers in `.gitignore`: both ignore `/LittleLauncherMSIX/bin/`, and IMD ignores `RepilotMSIX/bin/` instead of `ImmichDriveMSIX/bin/`. | `immich-drive/.gitignore` (lines 244, 382), `repilot/.gitignore` (line 244) | independent |
| FIX-032 | LL | `.gitattributes` has a stale `Website/output.css` line, and 40 tracked text files, the README among them, start with a byte-order mark. | `little-launcher/.gitattributes` (line 65), `little-launcher/README.md` | independent |
| FIX-033 | LL | An unused `LittleLauncherMSIX.wapproj` is still tracked. | `little-launcher/LittleLauncherMSIX/LittleLauncherMSIX.wapproj` | D-054 |
| FIX-034 | LL | Three skills tell agents to add strings to `Resources/Localization/Dictionary-en-US.xaml`, which no longer exists, and the `AGENTS.md` documentation table lists "repo memory", which is not a thing in this repo. | `little-launcher/.agents/skills/add-setting/SKILL.md`, `add-settings-page/SKILL.md`, `add-launcher-feature/SKILL.md`; `little-launcher/AGENTS.md` (lines 109, 110) | independent |
| FIX-035 | KIT | `AGENTS.md` contains an absolute host path to one consumer and still says Hub Display Switch is "planned to migrate"; it migrated on 2026-11-15. | `wsl-status/external/winui-app-kit/AGENTS.md` (lines 3, 17), at kit commit `5e9cfcf` | independent |
| FIX-036 | IMD | `AGENTS.md` still uses the old project name "CopilotRekey" for Repilot. | `immich-drive/AGENTS.md` (line 37) | independent |
| FIX-037 | HDS, REP, IMD, PER, LL | The copied Windows build guide says the commands are the same "across all four apps"; six repos carry it. WSL's copy was corrected and the fix did not reach its twins. | `<repo>/.codex/docs/windows-development.md` in HDS, REP, IMD, PER; `little-launcher/docs/windows-development.md` | independent |
| FIX-038 | HDS | The build guide says `build:desktop` yields an ARM64 package while `AGENTS.md` says this PC is x64 and to pass `--platform x64`. | `display-switchr/.codex/docs/windows-development.md` (line 7), `display-switchr/AGENTS.md` (line 88) | independent |
| FIX-039 | PS | **Resolved upstream before 2026-11-16 (confirmed while writing the parity brief).** The `audit-docs` skill refers to `docs/plugins.md`, which does not exist; the guide is `.agents/guides/plugins.md`. | `printstream/.agents/skills/audit-docs/SKILL.md` | independent |
| FIX-040 | PS | `AGENTS.md` gives two locations for the UI conventions (`docs/ui-conventions.md` and `.agents/guides/ui-conventions.md`); both files exist. | `printstream/AGENTS.md` (lines 72, 86) | D-026 |
| FIX-041 | PS | A tracked instruction and a skill require worktrees under an absolute path in the maintainer's home directory. | `printstream/AGENTS.md` (line 144), `printstream/.agents/skills/start-worktree/SKILL.md` (line 14) | D-032 |
| FIX-042 | PS, GIU | Husky is half-installed: `.husky/` holds only the generated `_` folder in both; GIU still has the `prepare: husky` script and dependency with no hook; PS's local `core.hooksPath` points at `/workspace/printstream/.husky/_`, which does not exist on this machine. | `printstream/.husky/`, `game-is-up/.husky/`, `game-is-up/package.json` (lines 39, 51), PS local git config | D-030 |
| FIX-043 | IMD, PER, GIU, SNB | Workflow files restate the global ban on attribution trailers, which the global rules say not to duplicate into projects. | `immich-drive/.codex/commands/commit.md`, `release.md`, `immich-drive/.codex/docs/versioning.md`; `persistent/.agents/skills/commit/SKILL.md`, `release/SKILL.md`; `game-is-up/.agents/skills/commit/SKILL.md`, `deploy/SKILL.md`; `sonarbridge/.agents/skills/commit/SKILL.md` | independent |
| FIX-044 | IMD, LL, REP, HDS, PER, GIU, SNB | Em dashes in tracked docs, against the global writing rule: agent files in IMD (12 files), LL (9 files), REP and HDS (`AGENTS.md`); 11 in PER's README, 8 in GIU's `ARCHITECTURE.md`, 8 in SNB's README. | `immich-drive/AGENTS.md` and `.codex/`, `little-launcher/AGENTS.md` and `.agents/`, `repilot/AGENTS.md`, `display-switchr/AGENTS.md`, `persistent/README.md`, `game-is-up/ARCHITECTURE.md`, `sonarbridge/README.md` | independent |
| FIX-045 | HDS, REP, PER, SNB, PS | Prose is hard-wrapped at about 80 to 100 columns, against the global writing rule. | `display-switchr/README.md`, `repilot/AGENTS.md`, `persistent/AGENTS.md`, `sonarbridge/AGENTS.md`, `printstream/apps/desktop/store/README.md` | independent |
| FIX-046 | REP, IMD, PER | `.gitmodules` still points the promo submodule at the previous personal-account URL for technicallyreal-promo; the other three consumers use `Dynamic-Solutions-Canada/apps-promo`. | `repilot/.gitmodules`, `immich-drive/.gitmodules`, `persistent/.gitmodules` (line 3 in each) | independent |
| FIX-047 | REP, IMD | The local `origin` remote still uses the pre-move personal-account URL (local git config, not a tracked file). | `git -C repilot remote get-url origin`, `git -C immich-drive remote get-url origin` | independent |
| FIX-048 | PER, LL | Store-secret helper scripts default to or list repos by their old personal-account `<Name>` names. Found while verifying; not in the audit reports. | `persistent/apps/desktop/set-ci-secrets.sh` (lines 52 to 55, 92), `little-launcher/LittleLauncherMSIX/set-store-secrets.ps1` (line 32) | independent |
| FIX-049 | PER | `compose.dev.yml` hard-codes the host path `/development/windows-build-tools`, a sibling checkout with no remote. | `persistent/compose.dev.yml` (lines 21, 34) | D-005 |
| FIX-050 | PER | Workflows use three versions of `actions/checkout` (`@v4`, `@v7`, `@v7.0.1`) and hard-code the Node version 17 times instead of reading `.nvmrc`. | `persistent/.github/workflows/*.yml` | independent |
| FIX-051 | PS | The export script defaults to a sibling folder `../printstream-public`, while the release and export skills use `data/exports/` and warn that the default target is wrong. | `printstream/scripts/export/export-public.mjs` (lines 62, 91), `printstream/.agents/skills/export/SKILL.md` (lines 20 to 22) | independent |
| FIX-052 | PS | Dependabot watches only `/`, so the Home Assistant workflow source directory is never updated (its `actions/checkout` is at `@v4`). | `printstream/.github/dependabot.yml`, `printstream/integrations/home-assistant/.github/workflows/` | independent |
| FIX-053 | PS | The "Artwork" section of the desktop store README has a paragraph spliced into the middle of a sentence. | `printstream/apps/desktop/store/README.md` (lines 24 to 28) | independent |
| FIX-054 | GIU | The repo has 37 test files but no root `test` script, `validate` runs none of them, and the API workspace's `test` uses an unquoted `**` glob. | `game-is-up/package.json` (line 30), `game-is-up/apps/api/package.json` (line 9) | independent |
| FIX-055 | SSC | **Mostly resolved upstream (confirmed after the 2026-11-16 fast-forward); the store-sign-controller brief covers what remains.** The root `test` script lists directories by hand with unquoted globs, so new test folders are silently skipped. | `store-sign-controller/package.json` (line 17) | independent |
| FIX-056 | SSC | A `.prettierrc.json` is tracked but Prettier is not a dependency and no script runs it. | `store-sign-controller/.prettierrc.json`, `store-sign-controller/package.json` | D-016 |
| FIX-057 | DK | `.nvmrc` says Node 20.20.2 while CI runs Node 24. | `devkit/.nvmrc`, `devkit/.github/workflows/ci.yml` (line 20) | D-007 |
| FIX-058 | DK | `package.json` declares MIT but the repo has no licence file. | `devkit/package.json` (line 5) | D-008 |
| FIX-059 | GIU | The README still describes the project as "the initial scaffold". | `game-is-up/README.md` (lines 3, 300) | independent |
| FIX-060 | LL, REP, IMD, PER | The packaging script does not follow the signing rule that HDS and WSL already meet: signing material is passed in as a parameter, with no fallback in the script (SEC-11 in `standards/security.md`, DIST-2 in `standards/distribution.md`). Move to the shared signing path under D-005. The detail is tracked privately with the security fixes. | `build-msix.ps1` in each repo's MSIX folder | D-005 |

## Dropped or not carried over

These were raised in the audit but are not listed above.

- **store-sign-controller, `apps/desktop/artifacts/` not ignored (STR 5).** No longer applies: `git check-ignore` reports the path as ignored.
- **store-sign-controller, `ANDROID_APP_ORIGINS` listed twice in `.env.example` (SET 5).** Not a defect: one line is a commented example above the real entry.
- **repilot, two generations of old certificate files in `RepilotMSIX/` (WAP 5).** No certificate files are tracked there; any that exist are local and already ignored.
- **little-launcher stray root files `build_out.txt`, `.crashdiag/` and devkit's untracked `.agents/` (STR 5, WEB 5).** Untracked local state, not repo defects. One part of that finding is FIX-003.
- **sonarphone "Latest" release pinned to v0.2.4 by a repository variable (STR 5).** Lives on GitHub, not in a file; could not be verified read-only without network use.
- **game-is-up Codex hook matchers may never fire (AI 2.6).** The audit marks this as an unverified hypothesis; it needs a live Codex session to confirm.
- **`Co-Authored-By` trailers in existing commits (SET 21.1: HDS 7 of 7, PL, LL, GIU, KIT).** Confirmed for HDS, but existing history cannot be fixed without rewriting it. The rule applies to new commits; nothing to change in a file.
- **One Android signing finding (AND 5).** Not verified, because confirming it would mean opening files the audit rules say not to open. The control is SEC-11 in `standards/security.md`, and it is settled for phonelock by D-021.
- **Stale run links to previous personal-account URLs in historical docs (AI 2.6: `immich-drive/.codex/docs/installer.md`; also LL and REP docs).** Links to past workflow runs; they still redirect and record history.
- **Missing features listed as parity gaps** (no CI, no licence file, no `AGENTS.md`, no tests, no Dependabot, tag workflows that publish without running tests, and similar) are not defects in an existing file. They become checklist items once the related standards and decisions are settled.
- **All gaps listed for out-of-scope repos** (both Shopify apps, android-vm, paseo-shared-browser, windows-build-tools, dynamic-solutions-website), per D-003.
