# Releases

How a version is chosen, recorded, tagged, published and followed up. How the artifacts are packaged, signed and delivered to users is in [distribution.md](distribution.md); the CI side of release gating is in [ci.md](ci.md); store listing text is in [store-listings.md](store-listings.md).

Counts are out of the 9 in-scope repos that tag or release today (little-launcher, repilot, immich-drive, persistent, printstream, phonelock, sonarbridge, printstream-home-assistant, devkit) unless a rule says otherwise. The other six (displayswitchr, tray-control-for-wsl, game-is-up, store-sign-controller, winui-app-kit, apps-promo) have no tags and no Releases, and follow these rules from their first release.

## Reference flows

No repo follows every rule below. Copy the named part of each reference, as the audit found them.

| Reference | Use it for | Known gaps |
| --- | --- | --- |
| immich-drive | Notes before the tag, for a single-app repo, and the overall model: one version file, a per-version notes file committed before the tag, CI fails when the file is missing, the same text is sent to the Store, tests run before publication | Notes are plain `.txt`; the procedure sits under `.codex/`; the Store submission fires on the tag, and only a manual run can stage a draft for review (see REL-7) |
| printstream | Notes before the tag, for a multi-channel product: an approved changelog entry checked by `changelog:check`, shown in the app and reused for the Release and the stores; a version check across workspaces; an approval stop before version files change; exports named in the procedure | Too heavy to copy whole; lane tags lack the `v` |
| sonarbridge | Android mechanics: version name and version code derived from the tag by a tested script, upload to Play from CI; the GitHub Release links to Play with no APK (D-112) | Notes are generated from commit subjects; the tag workflow runs no tests |
| printstream-home-assistant (exported from printstream) | Satellites: version in the host ecosystem's manifest, an export that refuses unbumped changes, and an idempotent Release created when the manifest version changes | Notes are GitHub's generated text and effectively empty |
| little-launcher | The best written versioning procedure (`docs/versioning.md`) | Notes file is optional and not sent to the Store; no tests on the tag |
| devkit | The closest thing to a kit flow: tagged versions that consumers pin | Releases are made by hand and exist for only some tags |

## Rules

### REL-1: One version source per deliverable

- **Status:** Existing (6 of 9 releasing repos keep one version file per deliverable; the exceptions are the three with Android apps: persistent, phonelock, sonarbridge). Decided (D-059) for Android and (D-060) for where a monorepo's product version lives
- **Applies to:** all
- **Reference:** `immich-drive/Directory.Build.props` (Windows); `printstream/package.json` with its `version:check` script (Web); `sonarbridge/scripts/release-version.sh` (Android)
- **Why:** A version held in two places will disagree, and on Android a version code can never go down once it has shipped.

| Stack | Version source | Notes |
| --- | --- | --- |
| Windows | `<Version>` in `Directory.Build.props`, three-part semver | The packaging script stamps the manifest; the fourth MSIX part is 0 for the Store and is used only for dev builds |
| Web | Root `package.json` | The root version is the product version and every workspace is aligned to it, with a check script that fails on drift (D-060) |
| Android | The tag | For new apps (D-059): version name is the tag without `v`, and the version code is computed from it. The three schemes already on devices (tag-encoded, run number plus offset, commit count) stay as they are |
| HA | `manifest.json` in the parent repo | Independent of the parent product's version (REL-10) |
| Kit | `package.json` (devkit) or `<Version>` in `Directory.Build.props` (winui-app-kit) | See REL-11 |

### REL-2: The bump is one commit that carries the notes

- **Status:** Decided (D-097). Proposed (repos that bump a file spell the commit subject three ways, and none carries the notes file in that commit by rule)
- **Applies to:** Windows, Web, Kit; Android has no file to bump
- **Reference:** `printstream/.agents/skills/release/SKILL.md` (notes approved before version files change)
- **Why:** One commit that holds both the version and its notes makes the tag self-describing and lets CI check the pair.

The notes for the version are written and approved first (REL-6). Then the version file is changed and committed together with the notes file. The subject is `Bump version to vX.Y.Z`, the form four of the six repos that bump a file already use, and a secondary lane adds its name: `Bump <lane> version to vX.Y.Z` (D-097). The lane form is new; no repo writes it today. This rule owns the bump subject; HYG-5 in [repo-hygiene.md](repo-hygiene.md) links here. Web monorepos bump every workspace in the same commit. The working tree is clean and `HEAD` matches the remote default branch before the tag is made. Commit rules in general are in the user-level agent rules (kept outside this repo) and are not restated here.

### REL-3: Tag format and lanes

- **Status:** Decided (D-060). Existing (met by the three newer WinUI apps; printstream produces `desktop-v` tags, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `little-launcher` tags (annotated `vX.Y.Z`); `persistent` tags (`desktop-vX.Y.Z`, `linux-vX.Y.Z`)
- **Why:** One rule lets a reader tell what a tag released without knowing the repo's history.

- `vX.Y.Z` is the product's primary deliverable.
- Every other deliverable in the same repo is a lane, tagged `<lane>-vX.Y.Z` (`desktop-v1.2.3`, `android-v1.2.3`, `linux-v1.2.3`). The `v` is always present.
- Tags a person or release script pushes are annotated, always, and sit on the default branch. A tag a workflow creates through the Releases API (a CI-built lane such as printstream's desktop or server packages) may be lightweight.
- Tags produced by automation for content-addressed builds (printstream's server packages) are outside this rule, as D-060 notes.
- Existing tags are never renamed. persistent moves Android to `android-v*` at its next release so that `v*` can mean the product, and printstream's desktop lane gains the `v`.

### REL-4: Deployed web products are tagged

- **Status:** Decided (D-061). Existing (persistent tags what it deploys, rechecked 2026-11-25; recount pending)
- **Applies to:** Web
- **Reference:** none in scope; printstream's public mirror creates `vX.Y.Z` from CI and is the nearest example
- **Why:** "What is live" should be answerable from git, not from a server.

A web product whose version users can see (persistent, printstream) tags the deployed commit `vX.Y.Z` as part of the production deploy. Products with no user-visible version (game-is-up, store-sign-controller) rely on the deployed commit hash until they start versioning.

### REL-5: CI creates the GitHub Release

- **Status:** Existing (met by the three newer WinUI apps, rechecked 2026-11-25; recount pending). The title format and the disclosure wording are Decided (D-097)
- **Applies to:** all
- **Reference:** `immich-drive/.github/workflows/build-msix.yml`
- **Why:** A Release made by hand is skipped under time pressure (devkit has Releases for only some of its tags) and its title and body drift from one release to the next.

The tag workflow creates the Release; nobody creates or edits one by hand. The Release:

- Is titled `<Product> X.Y.Z`, with the lane named for secondary lanes (`Persistent for Linux 0.2.0`) (D-097).
- Uses the committed notes as its body (REL-6), followed by an install or "Get it" section where the Release carries assets.
- Ends with the AI disclosure line that the user-level agent rules require on public GitHub posts, appended by CI to every Release in the one wording confirmed in D-097: `*AI disclosure: this release and the related code were written with the assistance of AI.*` It is written by 5 of the 7 CI-releasing repos today, in three wordings.
- Is marked "Latest" only for the primary lane.
- Carries the assets defined in [distribution.md](distribution.md), or none.

A flow that must stay a local script (phonelock, until it has workflows) meets the same content rules.

### REL-6: Release notes are a committed file, written for users

- **Status:** Decided (D-058). Existing (met by the three newer WinUI apps, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `immich-drive/release-notes/` with `immich-drive/.github/scripts/set-store-release-notes.ps1`; `printstream/apps/web/src/product-changelog.json` with `changelog:check`
- **Why:** Notes derived from commit subjects describe the work, not the benefit, and notes typed into a store console exist nowhere else.

- One path grammar, committed before the tag: `release-notes/[<lane>-]<version>[.<identity>].md`. `release-notes/<version>.md` holds the `en-US` text and is the only file an English-only app has. A localized app adds one sibling per other wording catalog (`<version>.fr-FR.md`), and the two Canadian identities reuse `en-US` and `fr-FR` through the alias mapping (LOC-5 in [locales.md](locales.md)). A secondary lane prefixes its name (`desktop-1.2.3.md`). A product whose app displays its changelog keeps a JSON changelog as the source instead (printstream) and applies the same rules to it.
- The body is user-facing bullets only, with no heading and no disclosure line; CI adds both. It is plain enough to fit a store's "What's new" limits. What counts as user-facing is the changelog rule in the user-level agent rules and is not restated here.
- CI fails when the notes for the tagged version, or a sibling a localized app requires, are missing or longer than a store's limit (LIST-5 in [store-listings.md](store-listings.md)). It never falls back to commit subjects or generated notes, and never truncates.
- The same text goes to the GitHub Release (the `en-US` file) and to the store's "What's new" field in each locale the listing ships (LIST-10 in [store-listings.md](store-listings.md)).
- Notes files stay in history. A single file overwritten each release is not a notes file.

There is no separate `CHANGELOG.md`; the per-version files are the changelog, and the README does not carry one (see [readme.md](readme.md)).

### REL-7: A tag creates a store draft; publishing is a second step

- **Status:** Decided (D-062). Existing (tag-triggered submission in 5 of 8 store flows; manual dispatch in 3 of 8)
- **Applies to:** Windows, Android, and store lanes of Web products
- **Reference:** `persistent/.github/workflows/store-publish.yml` (manual dispatch with a draft or publish choice)
- **Why:** Tag-triggered publishing has already needed a hard-coded tag exclusion (FIX-029) and repair workflows in three repos.

The tag workflow builds, runs the gate, and creates the store submission as a draft with the notes attached. Committing the draft for review is a separate, deliberate step: a manual workflow dispatch that checks the draft against the tag. A release that should skip a store is expressed by not running the second step, never by excluding a tag in a trigger.

An app's first Microsoft Store submission is the exception: Microsoft's submission API cannot create or edit it, so it is completed in Partner Center by hand (package upload, listings, age rating, then Submit), with the package built from the tag by the same scripts CI uses. The tag's store job stops at its pending-submission guard while that draft is open, which is expected. Every later release follows the flow above.

### REL-8: The release procedure lives in a `release` skill

- **Status:** Existing (met by the three newer WinUI apps; persistent uses the release-notes grammar, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `printstream/.agents/skills/release/SKILL.md`
- **Why:** A release is a fixed sequence that agents run, and a skill is the one place both an agent and a person will look.

Each releasing repo has `.agents/skills/release/SKILL.md` holding the full procedure, built from the checklist for its stack below, and a short human summary with a link in its development doc (see [docs.md](docs.md) and [ai-docs.md](ai-docs.md)). Invoking the skill authorises its own commit, tag and push end to end, with no second confirmation for those steps (D-013; AI-14 in [ai-docs.md](ai-docs.md)); a `deploy` skill carries the same authority for its own steps. As written before that decision, the skill stops for approval of the notes before anything is committed, and stops on a dirty tree; whether those two stops survive "end to end" is an open point noted in D-013, and they stay until the maintainer says otherwise. Procedures kept as Codex command files or in `AGENTS.md` move to the skill (D-027).

Notes approval and the go-ahead for the commit, tag and push may come from the maintainer or from someone the maintainer has delegated release approval to; the skill records who approved in its report.

### REL-9: Post-release steps are part of the release

- **Status:** Decided (D-063). Proposed (no releasing repo names the website or promo catalog step; printstream names its exports)
- **Applies to:** all
- **Reference:** `printstream/.agents/skills/release/SKILL.md` for a procedure that names every downstream step; `scripts/refresh-catalog.ps1` in apps-promo, documented in its README, for the catalog refresh
- **Why:** The website and the promo catalog are updated by hand outside every release document today, so they go stale without anyone deciding to leave them.

One shared post-release skill, delivered as AI-9 in [ai-docs.md](ai-docs.md) describes, is called by every `release` skill and covers:

| Step | When |
| --- | --- |
| Verify the Release, the store submission and the update path on an installed copy | Every release |
| Refresh the promo catalog (see [promo.md](promo.md)) | When the listing name, blurb or icon changed, or the app is newly listed |
| Update the product's website page and screenshots | When features, the minimum OS version, price or screenshots changed |
| Run exports to mirrors and satellites (REL-10) | Products that have them |
| Bump kit pins in consumers (REL-11) | After a kit release |
| Record the accepted store submission | Store apps |

Website and catalog mechanics are documented by their own repos; this rule only makes the steps part of every release.

### REL-10: Satellite repos release from the parent's export

- **Status:** Existing (1 of 1: printstream-home-assistant). The notes are Decided (D-058) and the minimum parent version is Decided (D-064)
- **Applies to:** HA, and any add-on generated from a parent repo
- **Reference:** `printstream/scripts/export/export-home-assistant.mjs` and `printstream/integrations/home-assistant/.github/workflows/release.yml`
- **Why:** A generated repo has no place for hand edits, so the release has to be a consequence of the export.

A satellite is developed in its parent and published as a generated snapshot whose commit records the source revision. Nobody edits the snapshot repo directly. Its version lives in the host ecosystem's manifest and is independent of the parent's product version. The export refuses to run when shipped files changed without a version bump, and pushing the export creates `vX.Y.Z` and its Release idempotently. The Release body is the parent's approved user-facing bullets instead of generated notes, and each release states the minimum parent version it needs, in the notes and the README. Stale public copies are FIX-026.

### REL-11: Kits are tagged and consumers pin tags

- **Status:** Decided (D-005). Existing (1 of 3: devkit, pinned by tag in all 3 of its consumers; winui-app-kit and apps-promo are pinned by bare commit)
- **Applies to:** Kit, and every consumer of one
- **Reference:** `devkit` tags and the tag pin in `persistent/package.json`
- **Why:** A bare commit pin says nothing about what changed, and six consumers currently sit on four different apps-promo commits.

Every kit release is an annotated `vX.Y.Z` tag with a Release created by CI and notes per REL-6. Consumers pin the tag, by npm git reference or by a submodule checked out at the tag. After a kit release, the pin bump is opened in each consumer as its own change; how far a consumer may fall behind, and the bump commit's subject, are KIT-6 in [shared-kits.md](shared-kits.md). Data that shipped apps fetch at runtime (the promo catalog) is not versioned this way; see [promo.md](promo.md). When to use each kit is in [shared-kits.md](shared-kits.md).

### REL-12: Release checklist per stack

- **Status:** Proposed (assembled from the reference flows; no repo follows a whole list)
- **Applies to:** all
- **Reference:** the reference flows table above
- **Why:** A short list per stack is what a `release` skill is built from and what a parity check compares against.

Common to every stack:

1. The working tree is clean and `HEAD` equals the remote default branch.
2. Choose the version. Write the notes (REL-6) and get them approved before touching version files.
3. Bump the version source and commit it with the notes (REL-2).
4. Create the annotated tag (REL-3) and push the branch and the tag together. Invoking the `release` skill is the request for these steps (D-013).
5. CI on the tag runs the gate, builds, fails if the notes are missing, and creates the Release (REL-5).
6. Verify the Release and the store draft, publish the draft (REL-7), then run the post-release steps (REL-9).

Additions per stack:

| Stack | Additions |
| --- | --- |
| Windows | Version in `Directory.Build.props`. CI builds x64 and ARM64 and prepares the Store upload; the Store "What's new" text is the notes file. Assets follow [distribution.md](distribution.md). Post-release: promo catalog, website page, and submodule bumps only when kit or promo code changed |
| Web | Version in the root `package.json`, mirrored to workspaces and checked. Deploy staging, then production, with the deploy script; tag the deployed commit (REL-4). Companion apps follow the Windows or Android list on their own lane tag. Post-release: exports and mirrors |
| Android | No file to bump for new apps; the tag is the version (REL-1). CI runs unit tests, builds the bundle, uploads to Play with the notes for each listing locale (REL-6; text over the store's limit fails the run), then attaches the Play-signed APK. Post-release: track status check, website and promo |
| HA | Bump `manifest.json` in the parent. Run the parent's tests, then the export. Pushing the export creates the tag and Release (REL-10) |
| Kit | Bump, tag and let CI create the Release. Post-release: open the pin bump in each consumer (REL-11) |
