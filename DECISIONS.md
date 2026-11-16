# Decision register

This register deduplicates every "divergence needing a decision" raised in the audit reports under `audit/` (142 numbered items across ten reports, many of them the same question asked several times) into one list. Where reports conflict, the second-round reports (`dev-ci-testing.md`, `store-locales-assets.md`, `structure-releases.md`, `settings-crosscutting.md`) and the "(refreshed 2026-11-16)" edits win. Concrete defects that need no decision are in `FIXES.md`. Entries D-085 onward were not audit divergences: they are decisions the standards writers found missing while writing `standards/`, collected in `audit/RECONCILE.md`, and each one records the recommendation its rule already states.

Every entry is settled as of 2026-11-16. "Already decided" holds the maintainer's own answers, the last 24 of them given that day (the raw answers are in the private `audit/ANSWERS.md`). "Adopted defaults" holds the entries that were offered as recommended defaults and not vetoed. `audit/` is private working material and is not published with this repo (D-004), so the source codes below point at files outside the public copy.

## Legend

- **Sections.** "Already decided" records the maintainer's decisions, each with the date, what it changes, and where it differs from what was recommended. "Open decisions" is for decisions that are irreversible, product- or user-facing, legal, cost-bearing, or that set direction for many other rules; it is empty today and lists only the points still open inside answered entries. "Adopted defaults" is everything else, grouped by topic, kept with its question, options and recommendation. IDs are stable and say nothing about position: an entry keeps its ID when the order changes, and entries added later (D-085 onward) sit in the section and topic group they belong to.
- **Counts** are "N of M applicable", counted against the 15 in-scope repos in `SCOPE.md` only. M is the number of in-scope repos the question applies to. Out-of-scope repos appear only as reference examples.
- **Reversibility.** cheap: a rename or config change inside the repo. costly: touches shipped apps, installed users, store listings or many repos at once. permanent: cannot be undone once published (store identities, published package ids, signing keys, public disclosure).
- **Blocks** lists the entries and fixes that could not be finalised until this one was answered; **Waits on** is the same link seen from the other side. Both survive on the adopted defaults as a record of the dependency; nothing is blocked now.
- **Repo codes.** HDS hub-display-switch, WSL wsl-monitor, LL little-launcher, REP repilot, IMD immich-drive, PER persistent (PD is its Windows desktop app), PS printstream, GIU game-is-up, SSC store-sign-controller, PL phonelock, SNB sonarphone-nmea-bridge, HA printstream-home-assistant (fixes land in PS), DK devkit, KIT winui-app-kit, PROMO apps-promo.
- **Source codes** are report plus section number: AI `ai-docs.md`, AND `android-and-misc.md`, DCT `dev-ci-testing.md`, PSA `printstream-addons.md`, SET `settings-crosscutting.md` (SET 10.n is its section 10, "Other areas worth standardizing"), SLA `store-locales-assets.md`, STR `structure-releases.md`, WAP `windows-apps-product.md`, WAR `windows-apps-repo.md`, WEB `web-apps.md`. "4.n" is item n of that report's section 4. REC is `audit/RECONCILE.md`; entries sourced from it also name the rule (for example HYG-3) that carries the recommendation.
- **Topic tags:** readme, docs, ai-docs, repo-hygiene, branding, code-style, repo-layout, dev-scripts, ci, testing, platform, releases, distribution, store-listings, locales, assets, ui, tray, promo, settings-config, shared-kits, logging, security.

## Already decided

### D-001 Scope of the standards
`repo-hygiene`
- **Decided (2026-11-15):** the 15 repos listed in `SCOPE.md` are held to the standards. The three shared projects (DK, KIT, PROMO) are both a source of conventions and held to them.
- **Consequence:** every count in this register is out of those 15. HA is generated from `printstream/integrations/home-assistant/`, so its fixes land in PS.
- **Reversibility:** cheap. **Repos:** all 15. **Sources:** `SCOPE.md`.

### D-002 How apps consume the standards
`ai-docs` `shared-kits`
- **Decided:** each app links to the standards repo on GitHub from its `AGENTS.md`, with an optional git submodule for apps that want an offline copy.
- **Consequence:** AI 3.4 recommended a mandatory submodule at `external/standards`; that is superseded. Recommendations below that assumed a mandatory submodule (D-029 shared skills, D-047 shared workflows) are adjusted to work with a link alone. A link only resolves if the reader can see the repo, which is why D-004 comes first.
- **Reversibility:** cheap. **Repos:** all 15. **Sources:** AI 3.4, AI 4.1.

### D-003 Excluded repos
`repo-hygiene`
- **Decided:** client work, both Shopify apps and the other repos listed under "Out of scope" in `SCOPE.md` are not held to the standards and may appear only as reference examples.
- **Consequence:** three audit questions are closed without further action: whether the Shopify apps get a profile (WEB 4.7), what to do with `shopify-form-filler/.github/copilot-instructions.md` (AI 4.13), and where android-vm lives (AND 4.13). Parity gaps listed for android-vm, paseo-shared-browser, windows-build-tools, dynamic-solutions-website and the Shopify pair are not carried into `FIXES.md`. windows-build-tools hosting survives only as part of D-005, because six in-scope repos depend on it.
- **Reversibility:** cheap. **Repos:** none in scope. **Sources:** `SCOPE.md`, WEB 4.7, AI 4.13, AND 4.13.

### D-108 Agent tooling: Claude Code only, tool-neutral files first
`ai-docs`
- **Decided (2026-11-16):** Claude Code is the only agent tool; Codex is not used and need not be supported. Files stay in tool-neutral locations; a Claude-branded path (`.claude/`) is used only where Claude Code cannot read anything else.
- **Consequence, tested 2026-11-16 on Claude Code 2.1.322:** `AGENTS.md` is the only instruction file and no repo has a `CLAUDE.md` (its presence stops `AGENTS.md` loading). Skills stay in `.agents/skills/` behind a tracked `.claude/skills` symlink, which Claude Code follows. Hooks are registered in `.claude/settings.json` (no other location exists) and their scripts live in `scripts/hooks/`. Subagents, if any, go in `.claude/agents/` (no other location exists). `.codex/` is removed everywhere after its content is moved. This settles D-027, D-028, D-033 and D-087, and the location half of D-030.
- **Reversibility:** cheap. **Repos:** all 15. **Sources:** the maintainer; `standards/ai-docs.md` "Where Claude Code looks".

### D-004 Standards repo visibility, and which repos are public
`ai-docs` `security` `repo-hygiene`
- **Decided (2026-11-16):** the standards repo is public. `audit/` and the security rows of `FIXES.md` are kept private and moved out before the first push. Every in-scope repo is public or will be made public, except PS, which stays private and publishes its snapshot: the Windows apps (HDS, WSL, LL, REP, IMD), the web apps PER, GIU and SSC, and KIT, PROMO and DK, together with the already public PL, SNB and HA.
- **Consequence:** `audit/` is git-ignored here, and the security section of `FIXES.md` (FIX-001 to FIX-010) now lives in `audit/security-fixes.md`; public files cite those fixes by ID and state the control, not the defect. The earlier recommendation left the visibility of nine repos unstated, so this answer changes several other entries. The identifier registry gains a visibility column (D-036, BRAND-10). The security policy file and issue template apply to 14 of 15 repos (D-106, HYG-9). Shared workflows can be reusable ones, because their host is public (D-047). No in-scope repo is left in the "private repo" licence class (D-008). The rule that a paid Store app in a public repo attaches no binaries now covers every paid Windows app (D-020). Making a repo public publishes its history, so the security fixes for a repo land before its visibility changes. Still open: the standards repo has no remote yet, so the URL that AI-4 puts in every `AGENTS.md` is filled in after the first push.
- **Reversibility:** permanent once pushed public (history included). **Repos:** all 15. **Sources:** AI 4.1, AI 3.4, DCT 4.5, REC (HYG-9).

### D-005 How shared code is delivered (winui-app-kit, devkit, build tooling, shared web UI)
`shared-kits`
- **Decided (2026-11-16):** kits for both stacks. KIT is a tagged submodule that carries the shared Windows build configuration, the settings shell, the About page and one packaging script. A new `web-app-kit` npm package starts with the Back-aware modal. DK stays an npm package. windows-build-tools gets a GitHub repo.
- **Consequence:** KIT and PROMO are tagged and consumers pin tags, not bare commits (REL-11, KIT-6). Each Windows app migrates to the kit one at a time; IMD migrates under the limit D-018 sets. `web-app-kit` is pinned by git tag the way DK is, targets the baseline D-007 sets, and is not part of DK, which stays development tooling. Other web primitives stay copied under PS's names until each reaches a third copy. This unblocks D-019, D-045, D-053, D-065, D-067, D-071, D-073, D-074 and D-096, and FIX-049 and FIX-060. Not stated in the answer: the owner of the two new repos, and the licence of `web-app-kit`.
- **Reversibility:** costly (every WinUI app's csproj, CI and build script). **Repos:** HDS, WSL, LL, REP, IMD, PER, KIT, PROMO, DK; PS, PER and SSC for the web package. **Sources:** WAR 4.1, WAR 4.2, WAR 4.3, WAP 4.2, WAP 4.16, STR 4.11, WEB 4.12, WAP Area 10, REC (KIT-10, UI-9).

### D-006 Publisher display name and copyright holder
`branding`
- **Decided (2026-11-16):** the publisher display name is "Dynamic Solutions (CA)", and PD is corrected to it. The copyright holder is the company, Dynamic Solutions, not an individual. GIU and SSC are personal projects and have the individual owner (the repo owner's legal name) as copyright holder.
- **Consequence:** the recommendation was to keep the individual owner as holder; the decision is the company. All seven existing licence files name the individual owner today, so the copyright line changes in LL, REP, IMD, PER and KIT, and every new licence file under D-008 names the company from the start (BRAND-3, HYG-1, README-5, and any About page that shows the line). The holder is recorded as given in the decision, "Dynamic Solutions", which is a different string from the publisher display name; the exact legal name to print is confirmed by the maintainer when the first file is changed. PS and HA keep their own licence (D-008), and the holder named in it was not part of this answer. The registry rows for publisher and holder are filled in (D-036).
- **Reversibility:** costly (store listings, shipped manifests, published licence files). **Repos:** PER, PROMO, PL, plus every repo with a licence file. **Sources:** SET 4.9, WAP 4.13, WAR 4.11, SET 10.8, SET 20.8.

### D-010 Locale policy: are English-only apps exempt
`locales` `store-listings`
- **Decided (2026-11-16):** nine locales with named exemptions. IMD is translated. PS is exempt until it has an i18n layer; GIU, SSC and HA are exempt. Localized apps translate their listings and release notes too.
- **Consequence:** an app is either fully in (interface, listing and release notes in all nine identities) or English only everywhere. IMD joins the localized set, which makes eight localized apps once it ships. Listing gaps in LL, REP, WSL and PL become parity work. This unblocks D-058, D-066, D-067 and D-098.
- **Reversibility:** costly (adding a locale is cheap; removing one from a published listing is user-visible). **Repos:** IMD, PS, GIU, SSC, HA; listing gaps in LL, REP, WSL, PL. **Sources:** SLA 4.6, SLA 3 (standards 5 and 8), WAP 4.15.

### D-009 GitHub owner and repo naming
`repo-hygiene` `branding`
- **Decided (2026-11-16):** GIU and SSC are personal projects and stay on the maintainer's personal account. PS and HA stay under `PrintStreamApp`. Everything else is in the `Dynamic-Solutions-Canada` organisation, so SNB and DK move there.
- **Consequence:** the recommendation was one organisation for everything except PS and HA; the decision keeps GIU and SSC personal, so there are three owners, not two (BRAND-4), and the registry records the owner per repo (D-036). SNB moves together with its privacy page address and in-app links, following BRAND-11. Repo names stay lower-case kebab-case. The two personal projects take the individual owner as copyright holder (D-006). Still open: whether devkit's package name, which carries the personal-account scope, changes when DK moves; consumers pin it by git URL, so the move itself needs the pin updated in PER, PS and GIU either way.
- **Reversibility:** costly. Installed copies keep old hardcoded URLs and rely on GitHub redirects; SNB's GitHub Pages privacy URL changes. **Repos:** SNB, DK to move; REP, IMD, LL, PER, PROMO to clean up. **Sources:** AND 4.5, SET 4.10, WAR 4.13, WAP 4.13, SET 20.8, STR 2.3.

### D-007 Web stack baseline: is store-sign-controller the next baseline or an outlier
`platform`
- **Decided (2026-11-16):** SSC's newer stack is the target baseline. PER and GIU migrate, PS last. SSC adopts DK.
- **Consequence:** the versions table in `standards/platform.md` is the record of the target (PLAT-8, PLAT-9). DK moves to the fleet Node version, which gives FIX-057 its value. Each repo moves one library major at a time, as its own change. This unblocks D-037, D-045 and D-050.
- **Reversibility:** costly (major-version upgrades across three products). **Repos:** PER, PS, GIU, SSC, DK. **Sources:** SET 4.6, WEB 4.2, WEB 4.3, SET 10.2.

### D-008 Licence for every repo
`repo-hygiene`
- **Decided (2026-11-16):** PolyForm Noncommercial for every app and for PROMO. MIT for DK and KIT. PS keeps its own licence. One file name, `LICENSE`. No source headers. The copyright holder follows D-006: the company, and the individual owner for GIU and SSC, which are PolyForm Noncommercial as well.
- **Consequence:** the recommendation gave GIU and SSC an all-rights-reserved file as private repos; they are public under D-004 and take PolyForm Noncommercial with the individual owner as holder. KIT carries PolyForm Noncommercial today and is relicensed to MIT. `LICENSE.md` is renamed to `LICENSE` in LL, REP, IMD, PER and KIT, and every copyright line is rewritten to the holder D-006 names. LL's partial set of source headers is removed and no repo adds any (HYG-2). HDS, WSL, GIU, SSC, PL, SNB and PROMO get their first licence file; DK gets the MIT file its `package.json` already declares (FIX-058). HA follows PS. This unblocks D-015.
- **Reversibility:** permanent for code already published under a permissive licence, which now includes KIT and DK; otherwise cheap. **Repos:** HDS, WSL, GIU, SSC, PL, SNB, DK, PROMO; rename in LL, REP, IMD, PER, KIT. **Sources:** AND 4.10, SET 4.14, WAR 4.11, WEB 4.14, WEB 4.15, SET 10.8, SET 21.3.

### D-016 Formatter and lint enforcement
`code-style`
- **Decided (2026-11-16):** formatting is enforced, rolled out repo by repo. PS moves on its own schedule.
- **Consequence:** every repo gets a root `.editorconfig`, a formatter for its stack and a `format:check` inside `validate` (STYLE-1, STYLE-2). Each repo takes its one-off reformat commit when it adopts the check. This unblocks D-091 and D-092, and FIX-056.
- **Reversibility:** cheap to switch on; the reformat commit is costly to review and disrupts open branches. **Repos:** all except HA (see D-084). **Sources:** WEB 4.1, WAR Area 2, AND 2.2, WEB 2.2.

### D-017 Privacy policy home and contact address
`docs` `distribution`
- **Decided (2026-11-16):** the privacy policy is a website page per app, on dynamic-solutions.ca or the product's own domain, written from one template, with the company contact address. Follow-up the same day: the two personal projects, GIU and SSC, are the exception; they use the individual owner's contact and host their own privacy page.
- **Consequence:** PL's GitHub file and SNB's GitHub Pages page move to website pages, and the five standalone Windows apps get their first policy (DOCS-8). Policy URLs registered in store listings change once. FIX-011 takes the company address. GIU and SSC (personal projects, D-009) keep the same template but publish it on their own site and give the individual owner's contact. This unblocks D-086 and D-106.
- **Reversibility:** costly (policy URLs are registered in store listings). **Repos:** PL, SNB, HDS, WSL, LL, REP, IMD. **Sources:** AND 4.4, SET 4.12, SET 20.3.

### D-011 Android package ids
`platform` `branding`
- **Decided (2026-11-16):** PL's parent app is renamed to `ca.dynamicsolutions.phonelock.parent` before its first Play upload. New apps use `ca.dynamicsolutions.<app>`, or the product's own domain if it has one.
- **Consequence:** existing child devices reinstall, and the ids pinned in PL's setup code are updated in the same change. The registry records the new id (D-036). This unblocks D-021.
- **Reversibility:** permanent once listed on Play. **Repos:** PL; rule for new apps. **Sources:** AND 4.1, SET 4.11.

### D-012 Product names
`branding`
- **Decided (2026-11-16):** "ImmichDrive" is the product name everywhere; the Store listing and the site change from "Drive for Immich". "PrintStream" is the name of the HA integration. "PhoneLock" is PL's app label. Follow-up the same day: "Drive for Immich" was an extra reserved name; it stays reserved in Partner Center and unused, as a fallback if a naming issue comes up.
- **Consequence:** the recommendation kept "Drive for Immich" for the Store and site and retired "ImmichDrive" from display strings; the decision is the opposite. The Store listing title and the website page are renamed; check in Partner Center that the name "ImmichDrive" can be reserved before relying on it. The README and prose that say "Immich Drive" change to the one-word form too. It is an exception to the spaced-name rule, alongside PhoneLock (BRAND-1). The repo stays `immich-drive`. HA's `manifest.json` and `strings.json` drop "PrintStream Bridge". FIX-023 takes "PhoneLock". The registry's display-name column is filled in (D-036), and the consistency check in LIST-12 has one name to check against.
- **Reversibility:** costly (Store names are reserved; display strings ship in releases). **Repos:** IMD, PROMO, HA (via PS), PL. **Sources:** SLA 4.12, WAR 4.14, PSA 4.1, AND 5 (phonelock), SET 20.8.

### D-013 Agent commit, push and deploy authority
`ai-docs`
- **Decided (2026-11-16):** invoking a release or deploy skill authorizes its own commit, tag and push, end to end. Other commits are still made only when asked.
- **Consequence:** the recommendation was that a deploy skill must stop and ask before committing; the decision grants the wider authority. AI-14, REL-8, REL-12 and SCRIPT-10 now say that running the skill is the request for those steps, with no second confirmation. The grant covers the skill's own work only: nothing outside the release or deploy is committed with it. Still open: whether the two stops the release procedure had before (approval of the notes, and a dirty tree) survive "end to end"; REL-8 keeps them until the maintainer says otherwise. The shared `commit` skill (D-029) carries the general rule and names this exception. The user-level agent rules (kept outside this repo) still say to commit only on an explicit request after review; they need the same exception written into them, which is the maintainer's to add, since the standards do not edit or restate those rules. This unblocks D-029.
- **Global rule (2026-11-16):** the user-level agent rules now carry the matching exception: invoking a release or deploy skill is the explicit request for its own commit, tag and push, and the skill still stops when the tree holds unrelated uncommitted work or the release notes are not approved.
- **Reversibility:** cheap. **Repos:** PER, PS, GIU, LL, SNB, IMD. **Sources:** AI 4.6, AI 2.11.

### D-014 Store plans for Hub Display Switch and WSL Monitor, and update depth
`distribution`
- **Decided (2026-11-16):** HDS and WSL go to the Microsoft Store, and their names are reserved now. The standard update path is "open the Store updates page", moved into KIT. PER keeps its in-app installer as the exception.
- **Consequence:** both apps replace their placeholder identity with the Store-assigned one before any install base exists (BRAND-6, DIST-4), and gain CI, tags, a licence file and a promo catalog entry as parity work. The small update service is shared from KIT (DIST-6). This unblocks D-065.
- **Reversibility:** permanent for the reserved identity; cheap for update depth. **Repos:** HDS, WSL; KIT for the update service. **Sources:** WAP 4.10, WAR 4.12, AI 2.6.

### D-015 What the public Home Assistant repo contains
`distribution`
- **Decided (2026-11-16):** the public HA repo stays as it is for now, a custom HACS repository holding the compiled bundle only. Revisit after the licence work.
- **Consequence:** card sources, tests and Python tool configuration stay in PS and are not exported (STYLE-9, TEST-4). Default-store listing is not pursued yet. This unblocks D-084.
- **Reversibility:** permanent for published sources; cheap otherwise. **Repos:** HA, PS. **Sources:** PSA 4.5, PSA 4.6.

### D-018 Minimum Windows version
`platform`
- **Decided (2026-11-16):** Windows 11 (build 22000) for all apps except IMD, which keeps Windows 10 22H2. Testing IMD on Windows 10 is accepted. Follow-up the same day: winui-app-kit's own floor is lowered to Windows 10 22H2 so IMD can adopt the kit like every other app.
- **Consequence:** the recommendation made IMD's lower floor conditional on the maintainer wanting it; the decision confirms it. Every manifest already carries the decided value, and `Package.appxmanifest` stays the source of truth (PLAT-6). FIX-021 takes 22000 for REP. IMD's releases are tested on Windows 10 22H2 before they ship (TEST-9). KIT's floor becomes build 19045, so IMD takes the full kit migration (D-005), the restyle through the kit's shared styles (D-019) and the shared update service (D-014). Any kit feature that needs Windows 11 checks the OS version at runtime and degrades on Windows 10; the kit's README names such features, and the kit itself is tested on Windows 10 22H2. The apps' own floors are unchanged.
- **Reversibility:** costly (raising the floor later strands Windows 10 buyers). **Repos:** IMD, REP, KIT. **Sources:** WAR 4.15, WAP 4.14, SET 10.2.

### D-019 Page styling generation for the Windows apps
`ui`
- **Decided (2026-11-16):** all Windows apps are restyled to the newest page look through KIT's shared styles, promo page first.
- **Consequence:** LL, REP, IMD and PD change visibly and need new store screenshots in the release that restyles them (UI-3). Each app is restyled once, during its kit migration. This unblocks D-072.
- **Reversibility:** costly (user-visible, needs new store screenshots). **Repos:** LL, REP, IMD, PER, PROMO. **Sources:** WAP 4.1.

### D-020 Little Launcher portable zips on a public repo
`distribution`
- **Decided (2026-11-16):** LL stops attaching portable zips to GitHub Releases.
- **Consequence:** "a paid Store app in a public repo attaches no binaries" has no exception (DIST-5). Zips on already-published Releases stay downloadable unless the maintainer removes them, which the answer does not cover. This unblocks D-041.
- **Reversibility:** cheap. **Repos:** LL. **Sources:** STR 4.10, STR 3 (standard 18).

### D-021 PhoneLock signing across GitHub and Play
`distribution` `security`
- **Decided (2026-11-16):** enrol PL's existing key as Play's app signing key. If Google refuses it, the two channels take separate app ids. Debug builds get their own key.
- **Consequence:** both channels keep one signer, so the in-app updater and the Play build can replace each other (DIST-8, SEC-11). It is done before the first Play upload and after the rename in D-011.
- **Reversibility:** permanent after the first Play upload. **Repos:** PL. **Sources:** AND 4.2, AND 1.

### D-022 Dead `printstream-public` sibling clone
`repo-hygiene`
- **Decided (2026-11-16):** delete `/development/printstream-public` after PS's export scripts are repointed (FIX-051).
- **Consequence:** FIX-051 comes first. The clone has no remote, so the deletion cannot be undone; it is outside this repo and is the maintainer's or a later task's to carry out (HYG-12).
- **Reversibility:** permanent. **Repos:** PS. **Sources:** STR 4.14, STR 2.3.

### D-023 Crash reporting and telemetry
`logging` `security`
- **Decided (2026-11-16):** no telemetry or crash reporting. The apps stay log-only, "no telemetry" is stated in READMEs and policies, and every app has "Open logs folder" and a support link.
- **Consequence:** LOG-8, LOG-9, DOCS-9 and BRAND-8 apply as written. The support link's target is D-086.
- **Reversibility:** cheap to add later; costly to remove trust once telemetry ships. **Repos:** all apps. **Sources:** SET 4.13, SET 20.2, SET 20.3.

### D-024 Windows compile on every pull request
`ci`
- **Decided (2026-11-16):** the Windows compile runs in CI only when relevant files change (path-filtered). Linux checks run on every pull request.
- **Consequence:** CI-3 applies as written, and the recurring cost of the Windows runner minutes it uses is accepted.
- **Reversibility:** cheap. **Repos:** HDS, WSL, LL, REP, IMD, KIT. **Sources:** DCT 4.6.

### D-025 Data backup, export and account deletion
`security` `docs`
- **Decided (2026-11-16):** backup, export and deletion are required and documented for every store-listed product, with PS as the reference. GIU and SSC document the answer. Uninstall leftovers are documented.
- **Consequence:** SEC-13 and DOCS-9 apply as written.
- **Reversibility:** cheap. **Repos:** PER, PS, GIU, SSC, IMD, SNB. **Sources:** SET 10.6, SET 20.9.

### D-085 Third-party notices in shipped apps
`repo-hygiene` `distribution`
- **Decided (2026-11-16):** every shipped app carries a generated third-party notices file, linked from the About page.
- **Consequence:** HYG-3 applies as written; ten apps need a generator script and an About link.
- **Reversibility:** cheap. **Repos:** HDS, WSL, LL, REP, IMD, PER, GIU, SSC, PL, SNB. **Sources:** REC (HYG-3).

### D-086 Support link target
`branding` `logging` `ui`
- **Decided (2026-11-16):** the support link goes to the repo's GitHub issues, which has an issue template. PS, being private, links to its help channel or the company contact.
- **Consequence:** under D-004 every app but PS has a public repo, so issues are the target in ten of the eleven end-user apps, including GIU and SSC on the personal account. The issue template (HYG-9) reminds the reporter what a log may contain, because a public issue is where a user will attach one (BRAND-8, LOG-8, UI-5).
- **Reversibility:** costly once shipped (installed copies keep the link they were built with). **Repos:** all 11 end-user apps. **Sources:** REC (BRAND-8, LOG-8, UI-5; raised by two writers).

## Open decisions

No entry is open. D-004 to D-025, D-085 and D-086 were answered on 2026-11-16, and they are recorded under "Already decided". The points left open inside those answers were settled the same day:

- **D-004:** the standards repo is `Dynamic-Solutions-Canada/app-standards` (https://github.com/Dynamic-Solutions-Canada/app-standards).
- **D-005:** `web-app-kit` and `windows-build-tools` both live in the Dynamic-Solutions-Canada organisation and are MIT.
- **D-006:** the company copyright line reads "Dynamic Solutions". printstream keeps its own licence and holder.
- **D-009:** devkit's package is renamed to `@dynamic-solutions-canada/devkit` when the repo moves to the organisation; PER, PS and GIU (and SSC when it adopts devkit) update their dependency in the same change.
- **D-013:** a release skill still stops on unrelated uncommitted work or unapproved notes; the user-level agent rules carry the matching exception.
- **D-017:** GIU and SSC use the individual owner's contact and their own privacy page.
- **D-018:** winui-app-kit's floor is lowered to Windows 10 22H2 so IMD adopts the kit fully.

A new question that only the maintainer can answer is added here with the next free ID.

## Adopted defaults

These entries were offered as "adopted unless vetoed". None was vetoed, so each is adopted as recommended on 2026-11-16: the option under **Recommend** is the decision, and the rules that cite the entry are marked `Decided (D-xxx)`. The entries are kept as written, with one marker line each, which also says where one of the other decisions changes how the entry applies. Four entries (D-027, D-028, D-033, D-087) were already closed by D-108 and carry that line instead. They are grouped by topic, not ranked.

### D-026 Where agent topic guides live
`ai-docs` `docs`
- **Adopted (2026-11-16).**
- **Question:** one home for topic guides that agents read.
- **Options:** (a) `.agents/guides/` [2 of 7 repos with guides: PER, PS]; (b) `.codex/docs/` [5 of 7: HDS, WSL, REP, IMD, PER]; (c) `docs/` [1 of 7: LL]. PER has both (a) and (b); PS keeps a `docs/` twin of most guides.
- **Recommend:** `docs/` for anything people also read, `.agents/guides/` for agent-only material, nothing under `.codex/` (removed under D-108). Reason: it is tool-neutral and matches the path the global rules already name for UI conventions.
- **Reversibility:** cheap. **Blocks:** FIX-040. **Repos:** HDS, WSL, REP, IMD (ten files), PER, PS (collapse each pair to one copy). **Sources:** AI 4.4, WAR 4.6, AI 3.1 (standards 11, 12, 14).

### D-027 `.codex/commands/`
- **Closed by D-108 (2026-11-16).** `.codex/` is removed from every repo after its workflows become skills. The text below is kept for the record.
`ai-docs`
- **Question:** keep the copied Codex command files or replace them with skills.
- **Options:** (a) delete in favour of skills [skills exist in 9 of 15 repos]; (b) keep as plain task guides [`.codex/commands/` in 5 of 15: HDS, WSL, REP, IMD, PER, as 5 identical sets of three build guides].
- **Recommend:** (a); convert IMD's five real workflows (commit, release, docs) to skills first. Reason: the `desktop` skill already covers the three build guides, and the copies have drifted.
- **Reversibility:** cheap. **Repos:** HDS, WSL, REP, IMD, PER. **Sources:** AI 4.5, AI 2.5.

### D-028 How Claude Code sees project skills
- **Closed by D-108 (2026-11-16).** Option (a), the tracked `.claude/skills -> ../.agents/skills` symlink, was tested and is the rule (AI-8). The text below is kept for the record.
`ai-docs`
- **Question:** project skills live in `.agents/skills/`, which Claude Code does not read, so all tracked project skills are invisible to it.
- **Options:** (a) a tracked symlink `.claude/skills -> ../.agents/skills` [0 of 9 repos with skills]; (b) move skills to `.claude/skills` and point Codex at them [0 of 9]; (c) leave Claude Code without them and list skills in `AGENTS.md` as files to read [9 of 9 today, by omission].
- **Recommend:** (a), after a one-repo test that Claude Code follows the symlink. Reason: one line per repo and no move; every repo is developed from WSL, so the Windows-clone fragility does not bite.
- **Reversibility:** cheap. **Repos:** HDS, WSL, LL, REP, IMD, PER, PS, GIU, SNB. **Sources:** AI 4.2, AI 1.

### D-029 How shared skills reach each repo
`ai-docs` `shared-kits`
- **Adopted (2026-11-16).** Follow-on from D-013: the shared `commit` skill states the commit and push rule together with the release and deploy exception.
- **Question:** the `desktop` skill (4 identical copies, 2 variants) and a generic `commit` skill are copied between repos. How are they shared?
- **Options:** (a) symlink from an `external/standards` submodule into `.agents/skills/` [0 of 15]; (b) install once at user level in `~/.agents/skills` with `~/.claude/skills` linked to it, as AI-8 does per repo [0 of 15; covers this machine, not cloud sessions]; (c) keep copies, with a drift check against the standards repo [copies in 9 of 15].
- **Recommend:** (b) now, plus (c) for repos that must work in cloud sessions; (a) only in repos that opt into the submodule. Reason: D-002 made the submodule optional, so the AI report's symlink recommendation cannot be the baseline.
- **Reversibility:** cheap. **Blocks:** D-063. **Repos:** every repo with skills. **Sources:** AI 4.3, AI 2.12. **Waits on:** D-004, D-013.

### D-030 Hooks and the git hook tool
`ai-docs` `repo-hygiene`
- **Adopted (2026-11-16).**
- **Question:** where do "never commit this" protections run, and with which tool?
- **Options:** (a) real git hooks through lefthook [0 of 15]; (b) husky [half-installed, no hook scripts, in 2 of 15: PS, GIU]; (c) tool hooks only [Codex hooks in 3 of 15: PER, PS, GIU; no Claude Code hooks anywhere]; (d) none [12 of 15].
- **Recommend:** (a) for what must never be committed (generated files, secrets), Claude Code hooks (registered in `.claude/settings.json`, scripts in `scripts/hooks/`, per D-108) only for mid-session guidance, Codex hooks ported or dropped, husky removed. Reason: lefthook is one binary and one file, is not tied to Node, and applies to every tool and to people.
- **Reversibility:** cheap. **Blocks:** FIX-042. **Repos:** PER, PS, GIU first. **Sources:** AI 4.8, AI 4.9, AI 2.7.

### D-031 Nested `AGENTS.md` depth in monorepos
`ai-docs`
- **Adopted (2026-11-16).**
- **Question:** how fine-grained may nested instruction files be?
- **Options:** (a) workspace level only (`apps/*`, `packages/*`), finer rules in guides; (b) keep fine-grained files [PS has 21 files and 24,526 words, nine of them under 500 characters]. Root file under about 1,500 words: 8 of 10 in-scope repos with an `AGENTS.md` (PS at 8,735 and LL at 1,930 are over).
- **Recommend:** (a). Reason: tiny nested files are loaded on demand and easy to miss; guides are discoverable from the guidance map.
- **Reversibility:** cheap. **Repos:** PS, PER, GIU. **Sources:** AI 4.10.

### D-032 Worktree location
`ai-docs` `repo-layout`
- **Adopted (2026-11-16).**
- **Question:** one location for agent worktrees, stated in one place.
- **Options:** (a) `/development/<repo>.worktrees` [what the audit and `SCOPE.md` assume]; (b) a worktree folder under the user's home directory [required by 1 of 15: PS, in a tracked file]. Both folders exist for PS.
- **Recommend:** (a), stated in DK or the global rules and removed from PS's tracked docs. Reason: a host path in a project file breaks in containers and for anyone else.
- **Reversibility:** cheap. **Blocks:** FIX-041. **Repos:** PS. **Sources:** AI 4.11, AI 2.6.

### D-033 Global rule parity between Codex and Claude Code
- **Closed by D-108 (2026-11-16).** Codex is not used, so there is no second global file to keep in parity. The text below is kept for the record.
`ai-docs`
- **Question:** the Codex global file lacks the memory-cap section and the device guide exists only under `~/.claude`. The global rules say the two files are deliberately not synced. Does that still hold for machine-safety rules?
- **Options:** (a) keep them separate [today; the gap affects agents in all 15 repos, 0 of 15 repos need a change]; (b) one shared file for machine-safety rules (memory caps, device handling, shared-computer rules) that both tools read; (c) sync everything.
- **Recommend:** (b). Reason: an out-of-memory kill takes down every agent regardless of which tool caused it. This changes the maintainer's user-level agent files, not any repo, so it is the maintainer's to apply.
- **Reversibility:** cheap. **Repos:** none (global files). **Sources:** AI 4.12.

### D-087 `CLAUDE.md` pointer file
- **Closed by D-108 (2026-11-16).** No repo has a `CLAUDE.md` at all, not even a pointer: Claude Code stops reading `AGENTS.md` when one exists. The text below is kept for the record.
`ai-docs`
- **Question:** may a repo have a `CLAUDE.md`, and what may it hold?
- **Options:** (a) none, unless a tool version in use fails to load `AGENTS.md`; then a `CLAUDE.md` holding the single line `@AGENTS.md` [0 of 15 have a `CLAUDE.md` today]; (b) the one-line pointer in every repo as a safeguard; (c) a separate instruction file per tool.
- **Recommend:** (a). Reason: both tools load `AGENTS.md` today (checked 2026-11-15), and a second instruction file is a second thing to drift. D-035's root file list allows the pointer for the one case it is needed.
- **Reversibility:** cheap. **Repos:** all 15. **Sources:** REC (AI-5).

### D-088 `paseo.json` in the standalone Windows repos
`ai-docs` `dev-scripts`
- **Adopted (2026-11-16).**
- **Question:** do the five standalone Windows repos get a `paseo.json`?
- **Options:** (a) yes: worktree setup initialises submodules, and there is one named script per `*:desktop` command [0 of 5]; (b) no, worktree setup stays manual [5 of 5]. The web monorepos have one in 3 of 4 (PER, PS, GIU).
- **Recommend:** (a). Reason: a new worktree without its submodules fails to build in a way that looks like a code bug, and named scripts let an agent call the four build commands the same way in every repo.
- **Reversibility:** cheap. **Repos:** HDS, WSL, LL, REP, IMD. **Sources:** REC (AI-12).

### D-089 Agent instructions for the generated Home Assistant repo
`ai-docs`
- **Adopted (2026-11-16).**
- **Question:** the public HA repo is generated and the export removes agent files, so where do its agent instructions live?
- **Options:** (a) a nested `AGENTS.md` in `printstream/integrations/home-assistant/`, with the generated repo exempt from the "every repo has a root `AGENTS.md`" rule [0 of 1: no nested file exists today]; (b) export a short `AGENTS.md` that says the repo is generated and where development happens; (c) nothing, as today.
- **Recommend:** (a). Reason: nobody edits the generated repo, so instructions belong where the work happens, and the exported README already says where that is.
- **Reversibility:** cheap. **Repos:** PS (for HA). **Sources:** REC (AI-1).

### D-034 README header and section names
`readme`
- **Adopted (2026-11-16).**
- **Question:** one README shape for product repos.
- **Options:** (a) centred header with icon, title, tagline, store badge and screenshots, then fixed sections [4 of 11 product repos: LL, REP, IMD, PER]; (b) plain `#` heading with ad hoc sections [HDS, WSL, SNB, PL]. Section names for the same content differ in every repo ("Install", "Installing", "Setup (minimal)"; "Building from source", "Development", "Build").
- **Recommend:** (a) for every product, with the fixed headings `## Features`, `## Get it`, optional product sections, `## Developers` (links only), `## License`; tool and add-on repos use their own templates (see `standards/readme.md`). Reason: it is the shape of the three shipped Store apps, Persistent and store-sign-controller, and keeping the Developers section to links stops README and docs from drifting.
- **Reversibility:** cheap. **Repos:** HDS, WSL, PL, SNB, GIU; renames in LL, REP, IMD. **Sources:** WAR 4.8, AND 4.11.

### D-035 Developer doc naming and placement
`docs`
- **Adopted (2026-11-16).**
- **Question:** where do development and architecture docs live and what are they called?
- **Options:** (a) lower-case files under `docs/` [developer guide as `docs/development.md` in 4 of the 7 repos with a standalone one: LL, PER, PS, SSC; architecture under `docs/` in 3 of the 6 repos with an architecture document: LL, SSC, and PER as per-area files; a `docs/README.md` index in 1: LL]; (b) upper-case root files, `DEVELOPMENT.md` or `ARCHITECTURE.md` [developer guide 1 of 7: IMD; architecture 3 of 6: IMD, PS, GIU]; (c) another name [REP's root `DEVELOPING.md`, SNB's `android/README-dev.md`]. Android: root `PRIVACY.md` and `PROTOCOL.md` (PL) against `docs/` (SNB). No developer doc: HDS, WSL.
- **Recommend:** the root holds only `README.md`, `LICENSE`, `AGENTS.md`, plus, where they exist, `ARCHITECTURE.md` and `GLOSSARY.md` (agents and people both look there), a `NOTICE` where the licence or shipped third-party code calls for one (D-085); never a `CLAUDE.md` (D-108). `SECURITY.md` goes under `.github/` (D-106). Every other document lives under `docs/` in lower-case names, the developer guide as `docs/development.md`. Reason: it keeps the root short, and `docs/development.md` is already the majority name.
- **Reversibility:** cheap. **Repos:** REP, IMD, PL, SNB, LL, SSC, PER. **Sources:** WAR 4.6, WEB 4.9, AND 4.14, REC (DOCS-1).

### D-036 Identifier registry
`branding` `repo-hygiene`
- **Adopted (2026-11-16).** Follow-on from D-004, D-006, D-009, D-011 and D-012: the registry under BRAND-10 has a visibility column, and its name, owner, publisher, holder and namespace cells are filled in.
- **Question:** should the standards repo hold one table of each product's permanent identifiers and every URL that shipped binaries fetch?
- **Options:** (a) a registry file, checked before any rename or transfer [0 of 15 have one]; (b) nothing, as today.
- **Recommend:** (a), kept in one place: the table under BRAND-10 in `standards/branding.md`, which every other rule links to instead of keeping its own list. Besides store identities, application ids and fetched URLs, it records each repo's visibility (D-004) and the per-app identifiers that must survive a kit migration (tray GUID, mutex name, startup task id, data folder name). Reason: three GitHub owners, four Android id schemes, two package identity forms and a promo manifest URL on a pre-rename name mean a repo move can silently break update checks and the promo page in installed copies. Record REP's `CopilotKeyRemapper` identity there as permanent.
- **Reversibility:** cheap. **Repos:** all products. **Sources:** SET 10.4, SET 3 (standard 20), REC (BRAND-10). **Waits on:** D-006, D-009, D-011, D-012.

### D-037 npm dependency ranges
`repo-hygiene` `platform`
- **Adopted (2026-11-16).**
- **Question:** exact pins or caret ranges.
- **Options:** (a) exact pins with Dependabot [1 of 4 web monorepos: SSC]; (b) carets with a committed lockfile [3 of 4: PER, PS, GIU].
- **Recommend:** (a), together with weekly grouped Dependabot (present in 1 of 15 for npm: PS). Reason: diffs show exactly what changed and repos stop drifting apart silently.
- **Reversibility:** cheap. **Repos:** PER, PS, GIU. **Sources:** SET 4.5, WEB 4.4. **Waits on:** D-007.

### D-038 Secret folder name
`security` `repo-hygiene`
- **Adopted (2026-11-16).**
- **Question:** one name for the project-local secrets folder.
- **Options:** (a) `.secrets/` [3 of 4 repos with a secrets folder: PER, PS, SSC]; (b) `secrets/` [1 of 4: PL]; SSC also has a second folder, `.private/`.
- **Recommend:** (a), ignored by a shared `.gitignore` block that also carries PER's backstop patterns and `.aws/`. Reason: it is the name the global credential rule already uses.
- **Reversibility:** cheap (PL's build script and docs change). **Repos:** PL, SSC. **Sources:** SET 4.8, SET 19.6.

### D-090 Topic branch names
`repo-hygiene`
- **Adopted (2026-11-16).**
- **Question:** one naming scheme for topic branches, and when they are deleted.
- **Options:** (a) `fix/<slug>` and `feature/<slug>` in kebab-case, deleted once merged, with tool-created branches keeping the tool's prefix [branch names were not counted in the audit]; (b) no rule, as today. The default branch is `main` in 15 of 15.
- **Recommend:** (a). Reason: two prefixes are enough to tell a fix from new work in a branch list, and a merged branch left behind is read by agents as live work.
- **Reversibility:** cheap. **Repos:** all 15. **Sources:** REC (HYG-4).

### D-039 Web page file naming
`code-style`
- **Adopted (2026-11-16).**
- **Question:** `*View.tsx` or `*Page.tsx` for routed pages.
- **Options:** (a) `*Page.tsx` [2 of 4: PER, SSC]; (b) `*View.tsx` [2 of 4: PS, GIU].
- **Recommend:** (a) for new work, no mass rename in PS. Reason: an even split with no functional difference; "Page" matches the WinUI apps' `Pages/` folders.
- **Reversibility:** cheap. **Repos:** GIU; PS for new files. **Sources:** WEB 4.8.

### D-091 Warnings as errors in .NET builds
`code-style`
- **Adopted (2026-11-16).**
- **Question:** does a compiler or analyzer warning fail the build?
- **Options:** (a) `TreatWarningsAsErrors` in CI and Release builds once a repo builds clean, off in Debug [0 of 6]; (b) on everywhere, Debug included [0 of 6]; (c) never, warnings are only read [6 of 6 today].
- **Recommend:** (a), set once in the shared build props. Reason: a clean build only stays clean if the build fails when it stops being one, and leaving Debug alone keeps work in progress compiling.
- **Reversibility:** cheap. **Repos:** HDS, WSL, LL, REP, IMD, PER, KIT. **Sources:** REC (STYLE-7). **Waits on:** D-016.

### D-092 Kotlin and Java formatter
`code-style`
- **Adopted (2026-11-16).**
- **Question:** D-016 names ktlint or Spotless for Android without choosing. Which?
- **Options:** (a) ktlint through its Gradle plugin [0 of 2]; (b) Spotless, running ktlint for Kotlin and a Java formatter for Java [0 of 2]. Today both repos set `kotlin.code.style=official` and run no formatter.
- **Recommend:** (b), the same setup in both repos. Reason: PL's lock app is Java, and Spotless formats Kotlin, Java and the Gradle scripts from one plugin, where ktlint alone would leave the Java unformatted.
- **Reversibility:** cheap to switch; the reformat commit is the cost, as in D-016. **Repos:** PL, SNB. **Sources:** REC (STYLE-8). **Waits on:** D-016.

### D-093 TypeScript module file naming
`code-style`
- **Adopted (2026-11-16).**
- **Question:** kebab-case or camelCase for TypeScript files that are not components?
- **Options:** (a) by workspace, as the repos already do: camelCase in `apps/web` (PascalCase for components), kebab-case in `apps/api`, `packages/` and `scripts/` [`apps/web` is camelCase in 4 of 4; `apps/api` is kebab-case in 3 of 4: PER, PS, GIU; SSC's API is mostly camelCase]; (b) kebab-case everywhere; (c) no rule, follow the folder.
- **Recommend:** (a) for new files, with no mass rename; SSC's API (about ten files) is renamed in one commit. Reason: it is what the repos do almost without exception (counted by file name on 2026-11-16), so writing it down costs one small rename.
- **Reversibility:** cheap. **Repos:** SSC. **Sources:** REC (STYLE-11).

### D-040 Build output folder and retention
`repo-layout`
- **Adopted (2026-11-16).**
- **Question:** where does local build output go, and what prunes it?
- **Options:** (a) git-ignored root `.artifacts/` everywhere [6 of 6 Windows-building repos: HDS, WSL, LL, REP, IMD, PER]; (b) per-app folders in monorepos [`artifacts/` in SSC, PER Linux, PS desktop; `release/` in PS server and bridge]. Retention: nothing prunes today (about 25 GB in total, 14 GB in LL).
- **Recommend:** (a), with fixed subfolders and automatic pruning to the last N bridge runs plus anything installed or released. Reason: one ignore rule and one place to clean.
- **Reversibility:** cheap. **Repos:** SSC, PS, PER; pruning in the six Windows-building repos. **Sources:** STR 4.1, STR 4.3.

### D-041 Artifact naming
`repo-layout` `releases`
- **Adopted (2026-11-16).** Follow-on from D-020: LL's portable zip is no longer attached, so it needs no new name.
- **Question:** adopt `<Product>-<version>-<platform>-<arch>[-<channel>].<ext>` and rename existing assets?
- **Options:** (a) new names for new products only; (b) migrate, attaching both names for one release. Today no two families agree: `App-<ver>-<arch>.msix` (5 of 6 MSIX-building repos), underscores (PD), no version (LL portable zip, PL APKs, SSC zips), lower case (SNB).
- **Recommend:** (a), plus (b) only where an updater does not fetch by name. Reason: PL's updater fetches assets by fixed name, and the LL and PER Linux updaters were not checked, so a blanket rename can break installed copies.
- **Reversibility:** costly where updaters depend on names; cheap otherwise. **Repos:** PL, LL, PER, SSC, SNB. **Sources:** STR 4.2. **Waits on:** D-020.

### D-042 Committed build outputs
`repo-layout` `releases`
- **Adopted (2026-11-16).**
- **Question:** do the two tracked binaries move to Release assets?
- **Options:** (a) move them [13 of 15 repos already commit no build output]; (b) keep [2 of 15: DK `vscode-extension/*.vsix`, SNB `esp32-client/*.factory.bin`].
- **Recommend:** (a). Reason: it also gives SNB's firmware the versioned release path it lacks today.
- **Reversibility:** cheap. **Repos:** DK, SNB. **Sources:** STR 4.13.

### D-094 PhoneLock's Gradle project root
`repo-layout` `platform`
- **Adopted (2026-11-16).**
- **Question:** does PL move its Gradle project under `android/`, as the Android layout puts it?
- **Options:** (a) move it, and rename `tools/` to `scripts/` in the same change [the layout is followed by 1 of 2: SNB]; (b) keep Gradle at the repo root and apply the layout to new repos only [1 of 2: PL].
- **Recommend:** (a), made together with PL's Gradle work under D-055 (wrapper, Kotlin DSL). Reason: build, CI and doc paths then change once instead of twice, and the root is left for the things every repo has.
- **Reversibility:** cheap (paths in the build script, docs and release helpers). **Repos:** PL. **Sources:** REC (LAYOUT-4). **Waits on:** D-055.

### D-043 Script names and grammar
`dev-scripts`
- **Adopted (2026-11-16).**
- **Question:** one vocabulary for package scripts across stacks.
- **Options:** rename `build:desktop` (which packages and signs) to `package:desktop:dev` so `build` always means compile, or keep it [`build:desktop` in 7 of 15: HDS, WSL, LL, REP, IMD, PER as a signed MSIX, SSC as a ZIP]. Grammar `job:target` [7 of 8 repos with target scripts] against `target:job` [1 of 8: PS]. `verify` meaning "compiles" [Windows family, PER] against "tests pass" [LL `verify:linux`, PS `desktop:check`].
- **Recommend:** rename once during the rollout; `job:target` for canonical jobs with `target:*` kept for device helpers; `verify:<target>` means compile only and `test:<target>` runs tests. Reason: these are DCT's recommendations and remove the two cases where one name does two jobs.
- **Reversibility:** cheap (mechanical, but touches docs and the `desktop` skill). **Repos:** HDS, WSL, LL, REP, IMD, PER, SSC, PS, GIU (`deploy:prod:ssh` becomes `deploy:prod`). **Sources:** DCT 4.1, DCT 4.2, DCT 4.3.

### D-044 Script entry points for the Android repos
`dev-scripts`
- **Adopted (2026-11-16).**
- **Question:** do PL and SNB get a root `package.json` wrapping Gradle under the canonical names?
- **Options:** (a) root `package.json` wrappers, as the five Windows apps do for `windows-build` [0 of 2]; (b) Gradle and shell stay the entry points and only Gradle task names are standardised [2 of 2].
- **Recommend:** (a). Reason: agents and `paseo.json` can then call `test`, `verify:android` and `package:android` the same way in every repo.
- **Reversibility:** cheap. **Repos:** PL, SNB. **Sources:** DCT 4.11.

### D-045 How much dev orchestration devkit owns
`shared-kits` `dev-scripts`
- **Adopted (2026-11-16).**
- **Question:** do `run-dev.mjs`, the compose `node` service, `paseo.json`, the `dev:*` scripts and the `deploy-over-ssh.mjs` deploy script move into DK?
- **Options:** (a) DK owns them through an `init` or template command [0 of 3 consumers; each hand-copies about 100 lines, and the deploy script exists in three implementations]; (b) keep copying [3 of 3: PER, PS, GIU].
- **Recommend:** (a) as templates DK generates, not a runtime dependency, with PS's 317-line runner allowed to stay custom. The deploy script ships the same way, as a template written into `scripts/deploy/` from PER's copy, so DK still takes no part in a production deploy at run time. Reason: it removes the copies without making DK opinionated about PS's host bridge, slicer and resume needs.
- **Reversibility:** cheap. **Repos:** DK, PER, PS, GIU, SSC. **Sources:** WEB 4.13, WEB 2.6, REC (KIT-10). **Waits on:** D-005, D-007.

### D-046 GitHub Action pinning
`ci` `security`
- **Adopted (2026-11-16).**
- **Question:** how are actions pinned?
- **Options:** (a) major tags plus Dependabot [Dependabot for actions in 1 of 8 repos with workflows: PS]; (b) SHA pins everywhere [0 of 8]; (c) SHA pins for third-party actions, major tags for `actions/*` and `github/*`.
- **Recommend:** (c), with a Dependabot `github-actions` entry in every repo. Reason: the third-party actions are the ones that touch signing and store secrets.
- **Reversibility:** cheap. **Repos:** LL, REP, IMD, PER, PS, SNB, HA, DK. **Sources:** DCT 4.4.

### D-047 Where shared workflows live
`ci` `shared-kits`
- **Adopted (2026-11-16).** Follow-on from D-004: the hosting repo is public, so this resolves to option (b), reusable workflows (CI-10).
- **Question:** copy-paste templates or reusable `workflow_call` workflows?
- **Options:** (a) templates copied from the standards repo [8 of 8 repos with workflows copy today, 45 files; the three `store-publish.yml` files already differ]; (b) reusable workflows hosted in DK or the standards repo [0 of 8 across repos; within-repo reuse in SNB and LL].
- **Recommend:** (b) if D-004 makes the hosting repo public, otherwise (a) with a drift check. Reason: reusable workflows must be readable by every caller, including the public repos.
- **Reversibility:** cheap. **Repos:** every repo with CI. **Sources:** DCT 4.5. **Waits on:** D-004.

### D-048 Workflow display names
`ci`
- **Adopted (2026-11-16).**
- **Question:** Title Case sentences or lowercase kebab matching the file name.
- **Options:** (a) Title Case [4 of 6 repos recorded: LL, REP, IMD, PS]; (b) lowercase kebab [2 of 6: PER, SNB]. File names for the same job differ too: the Play track check is `play-check.yml` in 2 of the 3 repos that have one (PER, PS) and `play-status.yml` in SNB, and the credential-expiry reminder is `credential-expiry.yml` in PS and `store-secret-expiry.yml` in PER.
- **Recommend:** (a), with one file name per job across repos (`ci.yml`, `codeql.yml`, `build-msix.yml`, `store-publish.yml`, `release.yml`, `play-publish.yml`, `play-check.yml`, `credential-expiry.yml`). Reason: majority practice and easier to read in the Actions tab.
- **Reversibility:** cheap. **Repos:** PER, PS, SNB. **Sources:** DCT 4.12, REC (CI-9).

### D-049 Database tests in the gate
`testing`
- **Adopted (2026-11-16).**
- **Question:** are database-backed tests part of `validate`?
- **Options:** (a) mandatory in `validate` with a provisioned Postgres [1 of 4: PS]; (b) skipped silently without an env var [1 of 4: PER]; (c) split into `test:integration` [1 of 4: SSC]; no database tests wired [1 of 4: GIU].
- **Recommend:** (c) locally, with CI running both `validate` and `test:integration` against a Postgres service; never a silent skip. Reason: the local gate stays fast and nothing passes by not running.
- **Reversibility:** cheap. **Repos:** PER, GIU, SSC, PS. **Sources:** DCT 4.7.

### D-050 Web test runner
`testing` `shared-kits`
- **Adopted (2026-11-16).**
- **Question:** which test discovery script is the standard for `node --test`?
- **Options:** (a) promote PS's `run-tests.mjs` (cache, batching, flake isolation) into DK for everyone [1 of 4]; (b) PER's small discovery script as the standard, PS's as a scale exception [1 of 4]; hand-written globs [2 of 4: GIU, SSC].
- **Recommend:** (b), shipped from DK. Reason: three of the four repos do not need PS's machinery, and unquoted globs are the actual defect to remove.
- **Reversibility:** cheap. **Repos:** GIU, SSC, PER, DK. **Sources:** DCT 4.8. **Waits on:** D-007.

### D-051 .NET test approach
`testing`
- **Adopted (2026-11-16).**
- **Question:** keep framework-free console harnesses or adopt a test framework?
- **Options:** (a) console harnesses under `tests/<App>.<Area>.Tests/`, run with `dotnet run` [5 of 5 repos with .NET tests: LL, REP, IMD, PER, SSC]; (b) xunit or similar [0 of 5].
- **Recommend:** (a), with one naming rule and a `test` script that runs them all. Reason: it is the universal convention, has no dependencies and runs from WSL and Linux CI.
- **Reversibility:** cheap. **Repos:** LL, REP, IMD, PER, SSC; HDS and WSL gain a first harness. **Sources:** DCT 4.9, WAR 4.9.

### D-052 Android emulator tests in CI
`testing` `ci`
- **Adopted (2026-11-16).**
- **Question:** is emulator instrumentation in CI a standard or a PrintStream-only investment?
- **Options:** (a) standard for every Android-building repo [1 of 4: PS, two API levels]; (b) optional, with JVM unit tests and lint as the required minimum [3 of 4 run no emulator: PER, SNB, PL]. SSC built an Android shell at audit time and is now web only.
- **Recommend:** (b). Reason: emulator jobs are slow and flaky to maintain; the smaller apps get more from wiring their existing unit tests into CI first.
- **Reversibility:** cheap. **Repos:** PER, SSC, SNB, PL. **Sources:** DCT 4.10.

### D-053 .NET SDK pin and runtime guard
`platform`
- **Adopted (2026-11-16).**
- **Question:** one `global.json` and one implementation of the runtime CVE guard.
- **Options:** SDK: 10.0.403 [3 of 6: LL, REP, PER], 10.0.100 [2 of 6: HDS, WSL], no `global.json` [1 of 6: IMD]. `rollForward`: `latestFeature` 4 of 5, `latestPatch` 1 of 5 (LL). Guard: present in 3 of 6 (LL, REP, PER), in three implementations (LL checks intent, REP checks the resolved runtime pack, PER checks the resolved pack and also the SDK version).
- **Recommend:** 10.0.403 with `latestFeature` in every repo and KIT; REP's guard (it checks what actually resolved), shipped once from KIT, with PER's extra SDK-version check kept for builds that run outside the folder `global.json` sits in. Reason: the guard's 10.0.14 floor needs the 10.0.4xx band, and one copy removes the drift.
- **Reversibility:** cheap. **Repos:** HDS, WSL, IMD, LL, PER, KIT. **Sources:** WAR 4.4, WAR 4.5, SET 10.2. **Waits on:** D-005.

### D-054 Solution format
`platform`
- **Adopted (2026-11-16).**
- **Question:** `.slnx` everywhere?
- **Options:** (a) `.slnx` [5 of 6: HDS, WSL, REP, IMD, PER]; (b) `.sln` plus a leftover `.wapproj` [1 of 6: LL].
- **Recommend:** (a). Reason: it is clearly the direction and LL's `.wapproj` is unused.
- **Reversibility:** cheap. **Blocks:** FIX-033. **Repos:** LL. **Sources:** WAR 4.10.

### D-055 Gradle conventions
`platform`
- **Adopted (2026-11-16).**
- **Question:** build script language, wrapper and version catalog for native Android repos.
- **Options:** Kotlin DSL [1 of 2: SNB] or Groovy [1 of 2: PL]; committed Gradle wrapper or a pinned host install [PL has no wrapper and a hard-coded host path]; version catalog [0 of 2].
- **Recommend:** Kotlin DSL, committed wrapper and a version catalog for new work and for PL; SNB keeps its documented AGP and Kotlin holds. Reason: the wrapper makes builds reproducible in CI and containers; "latest everywhere" is not automatically right for SNB.
- **Reversibility:** cheap. **Blocks:** D-094. **Repos:** PL, SNB. **Sources:** AND 4.7.

### D-056 Android UI toolkit for new work
`ui` `platform`
- **Adopted (2026-11-16).**
- **Question:** Compose or Views in code.
- **Options:** (a) Compose [1 of 2: PL]; (b) programmatic Views [1 of 2: SNB, by an explicit rule in its `AGENTS.md`].
- **Recommend:** (a) for new apps; SNB stays as it is; no shared Android UI kit until a third native app exists. Reason: Compose is the platform direction and a kit is only practical with one toolkit.
- **Reversibility:** costly for an existing app, cheap as a rule for new ones. **Repos:** rule for new apps. **Sources:** AND 4.6.

### D-057 Native host shells for web products
`platform` `shared-kits`
- **Adopted (2026-11-16).**
- **Question:** three products wrap a web app in native hosts built four ways (WinUI plus WebView2 in PD and SSC desktop; Electron in PS desktop, PER Linux and LL's Linux browser; Capacitor in PER, PS and SSC mobile). Converge?
- **Options:** (a) one host technology per platform and one shared host bridge contract [0 of 4 repos with host shells]; (b) leave the hosts as is and only write down the contract (settings, notifications, updates, version negotiation) [4 of 4 keep their own today: PER, PS, SSC, LL].
- **Recommend:** (b) now. Reason: the technology per platform is already consistent (WebView2 on Windows, Electron on Linux, Capacitor on mobile); the real gap is the unwritten contract.
- **Reversibility:** costly to converge code later, cheap to document. **Repos:** PER, PS, SSC, LL. **Sources:** SET 10.3.

### D-095 Upgrade cadence for platform pins
`platform`
- **Adopted (2026-11-16).**
- **Question:** when do the pinned SDKs, runtimes and library majors move?
- **Options:** (a) a scheduled pass per stack: every pin in the baselines table is reviewed once a quarter and moved in every repo of the stack together, with security releases handled sooner (D-107); (b) when a repo happens to need it, as today [two .NET SDK bands and three Node versions are pinned]; (c) only at each long-term-support release.
- **Recommend:** (a). Reason: the pin rule already says a pin moves in every repo in one pass, but with no date nobody starts the pass; a quarter is short enough that majors are taken one at a time.
- **Reversibility:** cheap. **Repos:** every repo with a pin. **Sources:** REC (PLAT-1).

### D-096 Central NuGet package versions
`platform` `shared-kits`
- **Adopted (2026-11-16).**
- **Question:** where do the Windows apps get the versions of the packages they share?
- **Options:** (a) central package management: a `Directory.Packages.props` holding each package version once, with the shared versions imported from the kit [0 of 6]; (b) a version in each csproj, as today [6 of 6; the Windows App SDK agrees, NLog and Sqlite have drifted]; (c) a shared `.props` file of version properties that each csproj reads.
- **Recommend:** (a) for the packages the family shares (Windows App SDK, NLog, CommunityToolkit.Mvvm), with app-only packages added in the repo's own file. Reason: it is the platform's own mechanism and leaves one line to change when a version moves.
- **Reversibility:** cheap. **Repos:** HDS, WSL, LL, REP, IMD, PER, KIT. **Sources:** REC (PLAT-5). **Waits on:** D-005.

### D-058 Release notes: location, format, localized storage and changelog policy
`releases` `store-listings`
- **Adopted (2026-11-16).** Follow-on from D-010: localized notes are required for the localized apps, IMD included.
- **Question:** where do release notes live, in what format, how are localized notes stored, and are they written or generated?
- **Options:** (a) per-version committed file [2 of 9 releasing repos: IMD `release-notes/X.Y.Z.txt`, LL `docs/releases/vX.Y.Z.md`]; (b) JSON changelog that also feeds the app [1 of 9: PS]; (c) generated from commit subjects [3 of 9: REP, SNB, PER Android; PER also has two overwritten single files]; (d) fixed sentence, GitHub-generated or none [3 of 9: PL, HA, DK]. Only IMD and PS feed one approved file to both GitHub and the store and fail when it is missing. Localized notes exist in 2 of 9 (PER: a per-version JSON file for Play and a `releaseNotes` field in `listings.json` for the Store; SNB: fastlane `changelogs/`). Ways to store them: (i) one Markdown file per wording catalog beside the English one; (ii) one JSON file per version with `locales` and `copyFrom`, the shape the store-listings audit proposed; (iii) a field in `listings.json` [PER, HDS].
- **Recommend:** one path grammar for every repo, `release-notes/[<lane>-]<version>[.<identity>].md`, with (i) for localized notes. `release-notes/<version>.md` is the `en-US` text, the GitHub Release body, and the only file an English-only app has. A localized app (D-010) adds one sibling per other wording catalog (`<version>.fr-FR.md`; six files for the nine-locale set), and `en-CA` and `fr-CA` reuse `en-US` and `fr-FR` through the alias mapping, with no files of their own. A secondary lane prefixes its name (`desktop-1.2.3.md`). The files are plain enough for the stores' text limits, written for users and committed before the tag; CI fails when the English file or a required sibling is missing or over a store's limit, and sends the same text to the store. `listings.json` carries no `releaseNotes` field and there is no per-version JSON; JSON stays only where the app displays its changelog (PS). HA gets written bullets instead of `--generate-notes`. Reason: it is IMD's flow, the cleanest in the family, it works unchanged for an English-only app, and one file per wording catalog is how the interface strings are already stored, so one translation pass covers both. It matches the global changelog rule.
- **Reversibility:** cheap. **Repos:** LL, REP, IMD, PER, PL, SNB, HA, DK; HDS and WSL from their first release. **Sources:** STR 4.4, WAR 4.7, WAP 4.12, AND 4.9, PSA 4.3, SLA 3, REC (LIST-10, REL-6). **Waits on:** D-010.

### D-059 Android version source and versionCode scheme
`releases` `platform`
- **Adopted (2026-11-16).**
- **Question:** does the Android version come from a file or the tag, and how is `versionCode` derived?
- **Options:** (a) tag-encoded semver [1 of 3 recorded: SNB]; (b) CI run number plus offset [1 of 3: PER, which has already needed an offset variable]; (c) commit count [1 of 3: PL, history must never be rewritten].
- **Recommend:** (a) for new apps, version derived from the tag; the three live schemes stay, since all are on devices. Reason: readable, deliberate and tested in SNB.
- **Reversibility:** permanent in direction (a versionCode can never go down); a rule for new apps is cheap. **Repos:** rule for new apps; PL, PER, SNB unchanged. **Sources:** AND 4.3, STR 4.5, STR 4.6.

### D-060 Tag lanes and where the product version lives
`releases`
- **Adopted (2026-11-16).**
- **Question:** tag prefixes for multi-artifact products, and whether the root version is the product version.
- **Options:** tags: `vX.Y.Z` for the primary deliverable [8 of 9 tagging repos], `<lane>-vX.Y.Z` for secondary lanes [1 of 2 multi-lane repos: PER; PS uses `desktop-X.Y.Z` with no `v`]. PER's bare `v*` means Android. Version: every workspace aligned to one root version [PS] or product version in `apps/web` with the root left at 0.1.0 [PER]. Annotated tags: 3 of 9.
- **Recommend:** root version is the product version with workspaces aligned; `vX.Y.Z` means the primary deliverable and every other lane is `<lane>-vX.Y.Z`, always annotated; PER moves Android to `android-v*` at its next release and PS's desktop tag gains the `v`. Tags that automation creates for content-addressed builds (PS's server packages) are outside the scheme. Reason: one rule a reader can apply without knowing the repo's history.
- **Reversibility:** cheap going forward (old tags stay). **Repos:** PER, PS, REP (lightweight tags since v1.0.21). **Sources:** STR 4.7, WEB 4.11.

### D-061 Tags for web deploys
`releases`
- **Adopted (2026-11-16).**
- **Question:** is a deployed web product tagged?
- **Options:** (a) tag `vX.Y.Z` at deploy [0 of 4 web monorepos]; (b) rely on the deployed commit hash [4 of 4].
- **Recommend:** (a) for products with a version users can see (PER, PS); (b) stays for GIU and SSC until they version. Reason: it makes "what is live" answerable from git.
- **Reversibility:** cheap. **Repos:** PER, PS. **Sources:** STR 4.9.

### D-062 Tag pushes straight to the store
`releases` `ci`
- **Adopted (2026-11-16).**
- **Question:** does a tag push submit to the store in one step?
- **Options:** (a) the tag creates a draft submission and a second manual step commits it; (b) as is: tag-triggered submission [5 of 8 store flows] and manual dispatch [3 of 8].
- **Recommend:** (a). Reason: tag-triggered publishing has already needed a hard-coded tag exclusion in LL and repair workflows in three repos.
- **Reversibility:** cheap. **Repos:** LL, REP, IMD, PER, SNB. **Sources:** STR 4.8.

### D-063 Who owns the post-release website and promo steps
`releases` `promo`
- **Adopted (2026-11-16).**
- **Question:** no release procedure in any repo includes the website or promo catalog step (0 of 9 releasing repos). Where does it go?
- **Options:** (a) a line in each app's release skill; (b) one shared post-release skill that every release skill calls.
- **Recommend:** (b), delivered the way D-029 settles. Reason: the steps are identical for every app (site page, `apps-promo/ids.json`, `refresh.ps1`), so one copy cannot drift.
- **Reversibility:** cheap. **Repos:** every releasing repo, PROMO. **Sources:** STR 4.12. **Waits on:** D-029.

### D-064 Home Assistant integration version relationship to PrintStream
`releases`
- **Adopted (2026-11-16).**
- **Question:** does each integration release record the minimum PS version it needs?
- **Options:** (a) fully independent versions [1 of 1: HA today]; (b) independent versions plus a stated minimum PS version per release.
- **Recommend:** (b), as a line in the release notes and README. Reason: the integration depends on the PS Home Assistant plugin's API, and users on older servers have no way to know.
- **Reversibility:** cheap. **Repos:** HA, PS. **Sources:** PSA 4.4.

### D-097 Release wording: bump commit subject, Release title and disclosure line
`releases` `repo-hygiene`
- **Adopted (2026-11-16).** Both strings were also confirmed directly: CI appends the disclosure line as worded here to every GitHub Release, and the bump subject is `Bump version to vX.Y.Z` with lanes adding their name.
- **Question:** three strings every release writes are spelled several ways. Which is the one form of each?
- **Options:** bump commit: `Bump version to vX.Y.Z` [the usual form in LL and IMD], `Bump version to X.Y.Z` [PER, REP], `Release vX.Y.Z` or `Release <Product> X.Y.Z` [PS, DK, once or twice each]; PER's two secondary-lane bumps read `Release Windows X.Y.Z` and `Release Linux X.Y.Z`. GitHub Release title: `<Product> X.Y.Z`, or the bare tag name [not counted in the audit]. AI disclosure line on the Release: three wordings across the 5 repos whose CI writes one ("this release and the related code" in LL and IMD, "this comment and the related code" in PER and SNB, "these release notes and the related code" in PS).
- **Recommend:** `Bump version to vX.Y.Z`, and for a secondary lane `Bump <lane> version to vX.Y.Z`, which is a new form no repo uses yet; the title `<Product> X.Y.Z`, with the lane named for a secondary lane; and the line `*AI disclosure: this release and the related code were written with the assistance of AI.*`, appended by CI. Reason: "Bump version to" is what four of the six repos that bump a file already write and the `v` matches the tag, and a Release is not a comment, so "this release" is the wording that reads correctly.
- **Reversibility:** cheap. **Repos:** LL, REP, IMD, PER, PS, SNB, DK. **Sources:** REC (REL-2, REL-5, HYG-5).

### D-065 Windows dev package identity
`distribution` `platform`
- **Adopted (2026-11-16).** Follow-on from D-014: HDS and WSL get a Store identity now, so the rule covers all six apps.
- **Question:** do dev builds reuse the Store identity or use a separate one?
- **Options:** (a) separate dev identity with opt-in Store identity [2 of 4 apps with a Store identity: IMD, PD]; (b) dev builds reuse the Store identity [2 of 4: LL, REP].
- **Recommend:** (a), driven by `windows-build.json`. Reason: it is the only model where a dev install does not collide with the Store install and an upgrade over the Store copy can be tested.
- **Reversibility:** cheap. **Repos:** LL, REP; HDS and WSL once D-014 reserves their names. **Sources:** WAP 4.11, WAR 4.12. **Waits on:** D-005, D-014.

### D-066 Store listing source format and location
`store-listings`
- **Adopted (2026-11-16).**
- **Question:** one source of truth for listing text and one place for it.
- **Options:** (a) `listings.json` with `schemaVersion`, `locales` and `copyFrom` aliases under root `store-assets/<store>/`, with fastlane and CSV trees generated from it [2 of 9 listed apps: PER, HDS]; (b) fastlane `metadata/` as the Play source [1 of 9: SNB, which also keeps a Markdown copy]; (c) other shapes [PS single-locale `listing.json`, PL Markdown draft]; no listing text in the repo [4 of 9: LL, REP, IMD, WSL]. PER currently splits material between `store-assets/` and `apps/mobile/store/`.
- **Recommend:** (a) at the repo root, including in monorepos; tool-specific trees are generated, never a second hand-kept copy. Reason: one validator and one translation pass, and it is already in two repos.
- **Reversibility:** cheap. **Blocks:** D-101, D-102. **Repos:** LL, REP, IMD, WSL, PS, PL, SNB, PER. **Sources:** SLA 4.1, SLA 4.2, AND 4.8. **Waits on:** D-010.

### D-098 Screenshot language in store listings
`store-listings` `locales`
- **Adopted (2026-11-16).**
- **Question:** are listing screenshots captured once in English or once per language?
- **Options:** (a) English captures for every listing locale, with captions and alt text translated [2 of the 2 apps with a nine-locale listing: PER, SNB]; (b) one capture set per wording catalog, seven sets; (c) a translated first screenshot only.
- **Recommend:** (a). Reason: seven capture sets on every interface change is a recurring cost that would leave screenshots stale; revisit for one app when a market matters enough to pay it.
- **Reversibility:** cheap. **Repos:** the seven localized apps. **Sources:** REC (LIST-7). **Waits on:** D-010.

### D-067 Windows localization: mechanism, key style, placeholders, folder and check name
`locales` `code-style`
- **Adopted (2026-11-16).**
- **Question:** five localized Windows apps use four mechanisms. What is the rule for new work and which existing apps move?
- **Options:** mechanism: KIT JSON catalogs through `UiText` [2 of 5: HDS, WSL], a private copy of the same design [2 of 5: LL, PD], `.resw` with `x:Uid` [1 of 5: REP]. Key style: English sentence as key [2 of 5: HDS, PD], dotted ids [2 of 5: WSL, LL], resource names [1 of 5: REP]. Placeholders: `{{name}}` [3 of 5 plus KIT], `{name}` [LL], `{0}` [REP]. Folder: four different paths. Check script: `check:l10n` (HDS), `check:strings` (WSL).
- **Recommend:** KIT catalogs, dotted ids, `{{name}}`, `Localization/Catalogs/<locale>.json` and `check:l10n` for new work and for the two kit apps (WSL renames its folder and script; HDS keeps English keys until its next string pass); REP's `.resw` is an accepted second mechanism for apps with no resident UI; LL and PD move to KIT's classes when they adopt the kit. Android stays snake_case. Reason: dotted keys survive wording fixes, and three of five already use `{{name}}`.
- **Reversibility:** costly (all of it is committed in five apps and seven catalogs each). **Repos:** HDS, WSL, LL, REP, PER, KIT. **Sources:** SLA 4.3, SLA 4.4, SLA 4.5, WAR 4.16, WAP 4.15. **Waits on:** D-005, D-010.

### D-068 Canadian alias catalogs on Android
`locales`
- **Adopted (2026-11-16).**
- **Question:** duplicate `values-en-rCA` and `values-fr-rCA` folders, or rely on platform fallback?
- **Options:** (a) rely on Android fallback [1 of 2 native apps: PL]; (b) keep three duplicate alias folders [1 of 2: SNB, 294 duplicated strings].
- **Recommend:** (a). Reason: aliases are a mapping, not files to keep in step; it matches the Windows rule that `en-CA` reads `en-US`.
- **Reversibility:** cheap. **Repos:** SNB. **Sources:** SLA 4.7.

### D-099 Web localization key style
`locales` `code-style`
- **Adopted (2026-11-16).**
- **Question:** D-067 settles key style for Windows only. What do web catalogs use?
- **Options:** (a) dotted ids, as D-067 recommends for Windows [PER uses them for error codes only]; (b) the English sentence as the key [1 of 1 localized web product: PER].
- **Recommend:** (a) for new web products and for new areas of an existing one; PER's English-sentence keys stay until a planned string pass and are not rewritten for their own sake. Reason: the same as D-067: a dotted key survives a wording fix, and one style across stacks lets one checker serve them.
- **Reversibility:** costly for PER (every key in seven catalogs); cheap as a rule for new work. **Repos:** PER; rule for new work. **Sources:** REC (LOC-7).

### D-100 Where the translation review note lives
`locales` `docs`
- **Adopted (2026-11-16).**
- **Question:** every catalog so far is AI-drafted. Where does an app record which catalogs a native speaker has reviewed?
- **Options:** the app's localization doc, the listing companion file, a comment in a catalog file, or the README [4 of 7 localized apps record it, each in a different one of these: SNB, HDS, PER, PL].
- **Recommend:** the app's localization doc (`docs/localization.md`), one row per wording catalog, beside the list of product terms that must be translated consistently. Reason: it is a document people read, it survives a regenerated catalog, and it gives every app the same place to look.
- **Reversibility:** cheap. **Repos:** the seven localized apps. **Sources:** REC (LOC-12).

### D-101 Promo page and catalog text localization
`promo` `locales`
- **Adopted (2026-11-16).**
- **Question:** who translates the promo page's own text and the app descriptions it shows?
- **Options:** page text: translated inside PROMO through its locale hook [used by 1 of 6 consumers: HDS, on the newest pin]; translated by each host through a copied adapter [3 of 6: WSL, REP, PER]; shown in English [2 of 6]. App descriptions: English only, taken from the Store when the catalog is refreshed [every entry today]; or one description per locale, taken from each product's listing source.
- **Recommend:** page text is translated only in PROMO, and the three host adapters are deleted when each consumer's pin is bumped. App descriptions stay English for now and are listed as deliberately English; once D-066 gives every app a `listings.json`, the catalog takes each locale's `shortDescription` from it. Reason: shared UI is translated where it ships, and the descriptions will already exist in every language in the listing sources, so they are not translated a second time.
- **Reversibility:** cheap. **Repos:** PROMO, WSL, REP, PER. **Sources:** REC (LOC-13, PROMO-7). **Waits on:** D-066.

### D-069 Icon source of truth and generated icons
`assets`
- **Adopted (2026-11-16).**
- **Question:** one master per mark, and are generated images committed?
- **Options:** master: (a) SVG master plus one shared generator that only sizes and packs [closest: 1 of 11 apps, GIU; PS derives Android from the web SVG]; (b) keep per-app PowerShell GDI+ drawing scripts and share only the sizing half [5 diverged copies: HDS, WSL, REP, IMD, PD; LL has raster masters and no generator]. Generated MSIX images: committed [5 of 6 WinUI apps: the five standalone apps] or ignored and regenerated at build [1 of 6: PD].
- **Recommend:** (a), generator in DK, with generated files committed where the platform needs them in-tree. Reason: it is the only route that also feeds the website, the promo catalog and non-Windows targets; committing keeps builds independent of the generator. The cost is redrawing five marks as SVG. This entry settles the master and whether generated files are committed; which folders they live in is D-102.
- **Reversibility:** cheap per app, done one at a time. **Blocks:** D-102. **Repos:** HDS, WSL, LL, REP, IMD, PER, DK. **Sources:** SLA 4.8, SLA 4.9.

### D-070 How the website and promo catalog get images
`assets` `promo`
- **Adopted (2026-11-16).**
- **Question:** stop hand-copying icons and screenshots?
- **Options:** website: hand copy [5 of 5 products with a site page, in sync today with three exceptions]; a sync script pulling named files at a pinned commit; hot-linking. Promo catalog icons: the Store's 50 px logo [all Store apps in the catalog] or a 256 px icon pulled through the existing `iconUrl` field [1 entry: PS at 192 px].
- **Recommend:** a sync script for the website (the site itself is out of scope, so this is a rule for where app repos publish their images) and 256 px `iconUrl` icons in PROMO. Reason: drift becomes a failed check, and 50 px icons are upscaled on most displays.
- **Reversibility:** cheap. **Repos:** PROMO, every app with a site page. **Sources:** SLA 4.10, SLA 4.11.

### D-102 Asset folder layout and README image home
`assets` `readme`
- **Adopted (2026-11-16).**
- **Question:** which folders hold icon masters and their exports, and where do README images come from?
- **Options:** (a) a root `assets/` folder with `brand/` for masters and `generated/` for the small cross-surface exports; the README takes its icon from `assets/generated/` and its screenshots from `store-assets/web/screenshots/` [0 of 11 end-user apps; PER is closest, embedding its listing graphics]; (b) README images in `docs/` or `docs/images/` [4 of 11: LL, REP, IMD, SNB]; (c) masters stay wherever each platform keeps them (`Resources/`, `apps/web/public/`), as today.
- **Recommend:** (a), as ASSET-1 in `standards/assets.md` lays it out. Reason: a master with a fixed address lets the README, the website and the promo catalog all point at one file, and one screenshot set serves the listing, the README and the site.
- **Reversibility:** cheap (file moves and README paths). **Repos:** the 11 end-user apps. **Sources:** REC (ASSET-1, README-9). **Waits on:** D-066, D-069.

### D-103 Home Assistant brand image sizes
`assets`
- **Adopted (2026-11-16).**
- **Question:** which sizes are the integration's `brand/` images generated at?
- **Options:** (a) the sizes the Home Assistant brands rules require, read from those rules when the generator is written and checked at each release; (b) keep today's files [192 and 512 px, 1 of 1, never checked against the rules].
- **Recommend:** (a). Reason: HACS and Home Assistant reject brand images outside their rules, and the rules are the host's to change.
- **Reversibility:** cheap. **Repos:** PS (for HA). **Sources:** REC (ASSET-11).

### D-071 Tray behaviour: double click, icon source, menu and tooltip
`tray`
- **Adopted (2026-11-16).**
- **Question:** four details the kit currently leaves to each app.
- **Options:** double click: opens the flyout [3 of 5 resident apps: HDS, WSL, PD], opens Settings [IMD], unhandled [LL]. Icon: drawn in code at exact pixel size with taskbar-theme awareness [3 of 5: HDS, WSL, LL] or `.ico` state variants [2 of 5: IMD, PD]. Menu: "Open <App>" first [2 of 5]; "Open logs folder" in the menu [2 of 5]; LL says "App Settings" where the rest say "Settings". Stable tray GUID: 3 of 5. Tooltip: `<App> – <state>` with a spaced en dash [2 of 5: HDS, WSL]; the app name alone [PD]; LL and IMD were not checked.
- **Recommend:** double click does the same as single click; either icon source is allowed but a stable GUID and a light-taskbar check are required; menu order is app actions, Settings, Open logs folder, separator, Exit; the tooltip is `<App> – <state>` as the two kit apps write it, or the app name alone when there is no state to show. Reason: both kit apps and Persistent already agree on double click, and WSL has the menu order exactly.
- **Reversibility:** cheap. **Repos:** LL, IMD, PER, KIT. **Sources:** WAP 4.3, WAP 4.4, WAP 4.5, REC (TRAY-2). **Waits on:** D-005.

### D-072 Settings labels
`ui`
- **Adopted (2026-11-16).**
- **Question:** one wording for the startup switch and the theme choice.
- **Options:** startup: "Start with Windows" [2 of 4 recorded: HDS, WSL], "Start at sign-in" [IMD], "Launch on Startup" [LL]. Theme: "Use system setting" [2 of 6: HDS, WSL] or "System default" [4 of 6].
- **Recommend:** "Start with Windows" and "Use system setting", in sentence case, shipped as KIT strings. Reason: they are the kit's wording, so the two kit apps need no change.
- **Reversibility:** cheap (seven catalogs per app). **Repos:** LL, REP, IMD, PER. **Sources:** WAP 4.7, WAP 4.8. **Waits on:** D-019.

### D-073 Where "start with Windows" state lives
`settings-config` `tray`
- **Adopted (2026-11-16).**
- **Question:** is the startup flag stored in settings or read from Windows?
- **Options:** (a) mirrored in `settings.json` [2 of 5 resident apps: LL, IMD]; (b) Windows is the only source of truth [1 of 5: PD, with a written rationale]; (c) KIT's route: read `StartupStatus` from Windows, show it, overwrite the stored copy [2 of 5: HDS, WSL].
- **Recommend:** (c). Reason: it cannot disagree with Task Manager and it is what the kit ships; "reset settings" then leaves the startup task alone.
- **Reversibility:** cheap. **Repos:** LL, IMD, PER. **Sources:** WAP 4.6, SET 4.4. **Waits on:** D-005.

### D-074 Exit call and host window
`tray`
- **Adopted (2026-11-16).**
- **Question:** how does a resident app exit, and how is the host window hidden?
- **Options:** (a) `Environment.Exit(0)` with an off-screen 1x1 window [3 of 5: HDS, WSL, LL]; (b) `Application.Current.Exit()` with a hidden window [2 of 5: IMD, PD].
- **Recommend:** (a) as the kit default, with an orderly-exit hook that runs before it. Reason: PD must close a connection and unregister notifications first, so the kit has to allow teardown either way.
- **Reversibility:** cheap. **Repos:** IMD, PER, KIT. **Sources:** WAP 4.9. **Waits on:** D-005.

### D-075 Promo on Android and web
`promo`
- **Adopted (2026-11-16).**
- **Question:** do the Android apps and the web apps show sibling apps?
- **Options:** Android: (a) no promo surface [2 of 2 native apps: PL, SNB]; (b) add one. Web: (c) optional, built from the web app's own components over the catalog [1 of 4 web monorepos: PER]; (d) required in every web product; (e) none. PROMO is not WinUI-only: it ships the WinUI page (wired in 6 of 6 WinUI apps) and a platform-neutral `catalog.ts` with a bundled manifest, which PER's web app consumes.
- **Recommend:** (a) for Android for now, and (c) for web: a product shows the catalog where it has a natural place for it, a client-branded or internal product (SSC) does not, and none is required to. Add HDS, WSL, PL and SNB to the catalog data when they have listings so the Windows apps, the web page and the site can show them. Reason: PL is a parental-control tool and SNB a single-purpose utility, so a promo page is a poor fit and needs new UI in two toolkits; on web the data is already there to read, but a required page would put the publisher's other apps inside a client's product.
- **Reversibility:** cheap. **Repos:** PL, SNB, PER, PROMO. **Sources:** AND 4.12, REC (PROMO-8).

### D-076 Where Windows app data lives
`settings-config`
- **Adopted (2026-11-16).**
- **Question:** roaming or local app data.
- **Options:** (a) everything in `%AppData%\<App>\` [6 of 7 Windows hosts: HDS, WSL, LL, REP, IMD, PD]; (b) `%LocalAppData%` [1 of 7: SSC desktop]; (c) settings roaming, logs and caches local.
- **Recommend:** (a), and SSC desktop follows when it moves to KIT. Reason: it is the family norm, packaged apps are redirected anyway, and splitting would move existing users' logs for little gain.
- **Reversibility:** costly once users have data (needs a migration). **Repos:** SSC. **Sources:** SET 4.1.

### D-077 Settings schema versioning and default values
`settings-config`
- **Adopted (2026-11-16).**
- **Question:** how are settings files versioned, and are default values written to disk?
- **Options:** versioning: integer `SchemaVersion` with ordered migrations [0 of 6], app version stamp [LL `LastRunVersion`], shape detection [HDS]. Defaults: written [2 of 6: HDS, WSL through KIT's `JsonSettingsStore`] or omitted [4 of 6: LL, REP, IMD, PD].
- **Recommend:** integer `SchemaVersion` in KIT's store with a backup before migrating; write defaults; LL keeps `LastRunVersion` as well for upgrade notices. Reason: simplest to reason about, and switching to written defaults is safe if it lands before any default changes.
- **Reversibility:** cheap now, costly after a default has changed. **Blocks:** D-104. **Repos:** all six WinUI apps, KIT. **Sources:** SET 4.2, SET 4.3.

### D-078 Env example naming and shared variable names
`settings-config`
- **Adopted (2026-11-16).**
- **Question:** one naming scheme for example env files and for variables that mean the same thing.
- **Options:** (a) by role, `.env.<role>.example` [1 of 4: PS]; (b) by mode, `.env.development.example` and `.env.production.example` [1 of 4: GIU]; (c) single `.env.example` [2 of 4: PER, SSC]. Android origin variable: `ANDROID_APP_CERT_FINGERPRINTS`, `ANDROID_APP_ORIGIN`, `ANDROID_APP_ORIGINS` across three apps.
- **Recommend:** (a) where there is more than one audience, (c) for one-container apps; one shared variable list in the standards with `ANDROID_APP_ORIGINS` as the Android name and booleans ending in `_ENABLED`. Reason: role-based naming scales to bridge and server deployments.
- **Reversibility:** cheap in the repo; renaming a variable is costly on deployed servers. **Repos:** GIU, PER, PS, SSC. **Sources:** SET 4.7, WEB 4.10, WEB 4.16.

### D-079 Production compose naming and topology
`settings-config` `distribution`
- **Adopted (2026-11-16).**
- **Question:** what is the production compose file called, is it tracked, and does every product ship one image?
- **Options:** naming: tracked `compose.server.yml` used directly [2 of 4: PER, SSC] or `compose.*.example.yml` copied to an ignored `compose.yml` [2 of 4: PS, GIU]. Topology: single image [3 of 4: PS, PER, SSC] or API plus nginx containers [1 of 4: GIU].
- **Recommend:** tracked `compose.server.yml` for products the maintainer hosts, an `.example` file only for self-hosted products (PS); GIU converges on a single image at its next deploy change. Reason: a tracked file is reviewable; an example file is right only when operators are expected to edit it.
- **Reversibility:** costly (renames on the server). **Repos:** GIU, PS. **Sources:** WEB 4.5, WEB 4.6.

### D-104 Reset and export of settings
`settings-config` `ui`
- **Adopted (2026-11-16).**
- **Question:** does every app offer "Reset settings", and which apps offer export and import?
- **Options:** reset: an action on the Settings page that keeps the old file under another name and loads defaults [0 of 6 WinUI apps], or none, so the user deletes the file by hand [6 of 6]. Export and import: in every app; only where settings are content the user authored [1 of 6: LL]; or nowhere.
- **Recommend:** reset in every app, provided by the kit's settings store, and "reset this device" for web device preferences; export and import only where settings are authored content (launchers, mappings, layouts). An export never contains credentials, and reset leaves state Windows owns alone. Reason: the support answer today is to delete a file in a folder that packaged apps redirect.
- **Reversibility:** cheap. **Repos:** the six WinUI apps, KIT; PER, PS and GIU for device preferences. **Sources:** REC (CONF-4). **Waits on:** D-077.

### D-105 Browser storage key grammar
`settings-config`
- **Adopted (2026-11-16).**
- **Question:** one naming scheme for browser storage keys.
- **Options:** (a) `<app>.<area>.<name>`, with a `.v<N>` suffix only when the stored shape changes, and every key declared in one module per app [0 of the 3 web apps with device preferences]; (b) per app, as today [PS alone uses four grammars].
- **Recommend:** (a) for new keys; an existing key is renamed only with a read-old, write-new migration. Reason: keys that are declared in one place can be listed, reset and checked for collisions.
- **Reversibility:** cheap for new keys; costly for existing ones, which hold users' current preferences. **Repos:** PER, PS, GIU. **Sources:** REC (CONF-7).

### D-080 Windows log file naming
`logging`
- **Adopted (2026-11-16).**
- **Question:** one log file per launch or one rolling file.
- **Options:** (a) per launch, `logs.<start time>.txt`, three archives kept [6 of 6 WinUI apps]; (b) one rolling file with a size cap [0 of 6].
- **Recommend:** (a), from one shared `NLog.config` in KIT with verified retention. Reason: universal today and easier to attach to a support request.
- **Reversibility:** cheap. **Repos:** none change; KIT gains the shared config. **Sources:** SET 4.15.

### D-081 Security baseline per stack
`security`
- **Adopted (2026-11-16).**
- **Question:** should the standards include a one-page security baseline (where credentials may live, required middleware, origin rules for hosts)?
- **Options:** (a) write it, with LL's DPAPI store, PS's secret encryption and SSC's production env guards as references; (b) leave each app to decide. Today: DPAPI for credentials 1 of 2 Windows apps that hold one (LL; the other is FIX-001); `helmet` 4 of 4 APIs; `express-rate-limit` 1 of 4 (SSC); origin allowlists written separately in 4 hosts.
- **Recommend:** (a). Reason: these are the mistakes that cannot be patched after a leak, and each app currently re-decides them.
- **Reversibility:** cheap. **Repos:** IMD, PER, PS, GIU, SSC, LL. **Sources:** SET 10.1, SET 3 (standard 5).

### D-082 Backend service conventions
`platform` `logging`
- **Adopted (2026-11-16).**
- **Question:** one service skeleton (health and readiness routes, error shape, logger with correlation id, mailer module) for the four APIs?
- **Options:** (a) write the skeleton as a standard, extract a shared package when a third copy is confirmed; (b) leave as four designs. Today: `/api/health` 4 of 4; readiness and status routes 1 of 4 (PS); logger with correlation ids 1 of 4 (PS); bare `console.*` 2 of 4 (GIU, SSC).
- **Recommend:** (a), with PS as the reference. Reason: operations (monitoring, log reading) currently differ per product for no product reason.
- **Reversibility:** cheap. **Repos:** PER, GIU, SSC. **Sources:** SET 10.5, SET 3 (standard 14).

### D-083 Demo mode and store-review access
`security` `store-listings`
- **Adopted (2026-11-16).**
- **Question:** one pattern for demo data and reviewer logins.
- **Options:** (a) one pattern: shared naming, refused in production by the env schema, credentials rotated after review; (b) per-app as today [four different mechanisms in 4 of 4 web products; only SSC's schema forbids its simulator in production].
- **Recommend:** (a), with SSC's production guard as the safety rule. Reason: reviewer logins are a recurring store need and a recurring way to leave a back door open.
- **Reversibility:** cheap. **Repos:** PER, PS, GIU, SSC. **Sources:** SET 10.7.

### D-106 Security policy file in public repos
`security` `repo-hygiene`
- **Adopted (2026-11-16).** It was also confirmed that the template promises fixes for the latest published version only. Follow-on from D-004: "every public repo" is every in-scope repo except PS, which is 14 of 15 (HYG-9).
- **Question:** do public repos carry a security policy file, where, and which reporting channel does it name?
- **Options:** (a) `.github/SECURITY.md` from one template in every public repo, naming GitHub's private vulnerability reporting as the channel and the company contact as the fallback [0 of 15]; (b) the same file at the repo root; (c) none, as today [15 of 15].
- **Recommend:** (a), with private vulnerability reporting switched on in each public repo, and a template that promises fixes for the latest published version only unless a product says otherwise. Reason: the only channel today is a public issue, and `.github/` keeps the root as short as D-035 wants it.
- **Reversibility:** cheap. **Repos:** every public repo (D-004 settles the list). **Sources:** REC (SEC-15, HYG-9). **Waits on:** D-004, D-017.

### D-107 Security update deadline
`security` `platform`
- **Adopted (2026-11-16).**
- **Question:** how soon is a security fix in a runtime or dependency applied across the repos?
- **Options:** (a) within one week of the advisory, in every repo on that runtime or library, in one pass; (b) at the next scheduled pass (D-095); (c) no deadline, as today [the last runtime fix reached 3 of 6 Windows-building repos].
- **Recommend:** (a) for anything that ships inside an app or serves traffic, (b) for development-only tooling. Reason: the apps ship self-contained, so a runtime fix reaches users only when the maintainer rebuilds and releases.
- **Reversibility:** cheap. **Repos:** all 15. **Sources:** REC (SEC-12).

### D-084 Python toolchain for the Home Assistant integration
`code-style` `testing`
- **Adopted (2026-11-16).** Follow-on from D-015: the Python tool configuration and tests stay in PS and are not exported.
- **Question:** which Python tools check the integration, and are `pyproject.toml` and `tests/` exported to the public repo?
- **Options:** (a) ruff alone, matching Home Assistant core; (b) ruff plus mypy; no Python tooling [1 of 1: the HA source in PS today]. Export: keep `pyproject.toml` and `tests/` monorepo-only like `card-source/`, or export them.
- **Recommend:** (a), run with the existing `unittest` suite in PS CI as a gate before export; keep both monorepo-only until D-015 says otherwise. Reason: it matches the host ecosystem at the lowest upkeep.
- **Reversibility:** cheap. **Repos:** PS (for HA). **Sources:** PSA 4.2. **Waits on:** D-015.

## Source coverage

Every numbered divergence in the audit reports maps to an entry here or to `FIXES.md`.

| Report | Items | Mapping |
| --- | --- | --- |
| AI (`ai-docs.md`) | 13 | 4.1 D-004; 4.2 D-028; 4.3 D-029; 4.4 D-026; 4.5 D-027; 4.6 D-013; 4.7 FIX-005; 4.8 and 4.9 D-030; 4.10 D-031; 4.11 D-032; 4.12 D-033; 4.13 D-003 |
| AND (`android-and-misc.md`) | 14 | 4.1 D-011; 4.2 D-021; 4.3 D-059; 4.4 D-017; 4.5 D-009; 4.6 D-056; 4.7 D-055; 4.8 D-066; 4.9 D-058; 4.10 D-008; 4.11 D-034; 4.12 D-075; 4.13 D-003; 4.14 D-035 |
| DCT (`dev-ci-testing.md`) | 12 | 4.1 to 4.3 D-043; 4.4 D-046; 4.5 D-047; 4.6 D-024; 4.7 D-049; 4.8 D-050; 4.9 D-051; 4.10 D-052; 4.11 D-044; 4.12 D-048 |
| PSA (`printstream-addons.md`) | 6 | 4.1 D-012; 4.2 D-084; 4.3 D-058; 4.4 D-064; 4.5 and 4.6 D-015 |
| SET (`settings-crosscutting.md`) | 15 plus 8 | 4.1 D-076; 4.2 and 4.3 D-077; 4.4 D-073; 4.5 D-037; 4.6 D-007; 4.7 D-078; 4.8 D-038; 4.9 D-006; 4.10 D-009; 4.11 D-011; 4.12 D-017; 4.13 D-023; 4.14 D-008; 4.15 D-080; 10.1 D-081; 10.2 D-007, D-018, D-053; 10.3 D-057; 10.4 D-036; 10.5 D-082; 10.6 D-025; 10.7 D-083; 10.8 D-006, D-008 |
| SLA (`store-locales-assets.md`) | 12 | 4.1 and 4.2 D-066; 4.3 to 4.5 D-067; 4.6 D-010; 4.7 D-068; 4.8 and 4.9 D-069; 4.10 and 4.11 D-070; 4.12 D-012 |
| STR (`structure-releases.md`) | 14 | 4.1 and 4.3 D-040; 4.2 D-041; 4.4 D-058; 4.5 and 4.6 D-059; 4.7 D-060; 4.8 D-062; 4.9 D-061; 4.10 D-020; 4.11 D-005; 4.12 D-063; 4.13 D-042; 4.14 D-022 |
| WAP (`windows-apps-product.md`) | 16 | 4.1 D-019; 4.2 D-005; 4.3 to 4.5 D-071; 4.6 D-073; 4.7 and 4.8 D-072; 4.9 D-074; 4.10 D-014; 4.11 D-065; 4.12 D-058; 4.13 D-009, D-006; 4.14 D-018; 4.15 D-067; 4.16 D-005 |
| WAR (`windows-apps-repo.md`) | 16 | 4.1 to 4.3 D-005; 4.4 and 4.5 D-053; 4.6 D-026, D-035; 4.7 D-058; 4.8 D-034; 4.9 D-051; 4.10 D-054; 4.11 D-006, D-008; 4.12 D-014, D-065; 4.13 D-009; 4.14 D-012; 4.15 D-018; 4.16 D-067 |
| REC (`RECONCILE.md`) | 33 missing decisions, 32 once the support link raised twice is merged | new entries: D-085 (notices), D-086 (support link, raised twice), D-087 to D-089 (CLAUDE.md, paseo.json for Windows, generated HA repo), D-090 (branch names), D-091 to D-093 (warnings as errors, Kotlin formatter, TypeScript file names), D-094 (PhoneLock Gradle root), D-095 and D-096 (upgrade cadence, central NuGet versions), D-097 (bump subject, Release title, disclosure wording), D-098 (screenshot language), D-099 to D-101 (web key style, translation note, promo text), D-102 and D-103 (asset layout and README images, HA brand sizes), D-104 and D-105 (reset and export, storage keys), D-106 and D-107 (security policy file, update deadline). Merged into existing entries: public repos D-004; shared web UI package D-005; `deploy-over-ssh` D-045; `play-check.yml` naming D-048; lane notes file name and localized notes D-058; tray tooltip D-071; promo on web D-075 |
| WEB (`web-apps.md`) | 16 | 4.1 D-016; 4.2 and 4.3 D-007; 4.4 D-037; 4.5 and 4.6 D-079; 4.7 D-003; 4.8 D-039; 4.9 D-035; 4.10 D-078; 4.11 D-060; 4.12 D-005; 4.13 D-045; 4.14 D-008; 4.15 D-008 (licence), FIX-057 (runtime); 4.16 D-078 |
