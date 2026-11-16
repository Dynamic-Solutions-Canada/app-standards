# Docs

Which documents a repo keeps besides its README, what they are called, and where they live. The README is covered in [readme.md](readme.md); files written for agents (`AGENTS.md`, guides, skills) are covered in [ai-docs.md](ai-docs.md); release notes are covered in [releases.md](releases.md).

Counts are out of the 15 in-scope repos unless a rule says otherwise.

## The docs set

| Document | Path | Needed when | Reference |
| --- | --- | --- | --- |
| README | `README.md` | Always | See [readme.md](readme.md) |
| Agent instructions | `AGENTS.md` | Always | See [ai-docs.md](ai-docs.md) |
| Licence | `LICENSE` | Always | See [repo-hygiene.md](repo-hygiene.md) |
| Developer guide | `docs/development.md` | Any app repo; a tool repo uses its README instead | `persistent/docs/development.md` |
| Architecture | `ARCHITECTURE.md` | The architecture no longer fits the table in `AGENTS.md` | `immich-drive/ARCHITECTURE.md` |
| Glossary | `GLOSSARY.md` | The product has its own vocabulary | `store-sign-controller/GLOSSARY.md` |
| Deployment, configuration | `docs/deployment.md`, `docs/configuration.md` | The product runs on a server | `printstream/docs/` |
| Topic and contract docs | `docs/<topic>.md` | A subject needs more than a section | `little-launcher/docs/versioning.md` |
| Docs index | `docs/README.md` | `docs/` holds more than about five files | `little-launcher/docs/README.md` |
| Privacy policy | Published page (DOCS-8) | The app is listed in a store or handles personal data | `phonelock/PRIVACY.md` for the outline |
| Release notes | `release-notes/<version>.md` | The repo releases | See [releases.md](releases.md) |

## Rules

### DOCS-1: Root files and the `docs/` folder

- **Status:** Decided (D-035). Existing (4 of the 7 repos with a standalone developer guide keep it under `docs/`: little-launcher, persistent, printstream, store-sign-controller)
- **Applies to:** all
- **Reference:** `printstream/` (root `ARCHITECTURE.md` and `GLOSSARY.md`, everything else in `docs/`)
- **Why:** A short root makes the few files everyone needs easy to find, and one naming scheme means a link written for one repo works in the next.

The only documents at the repo root are `README.md`, `LICENSE`, `AGENTS.md`, and, where they exist, `ARCHITECTURE.md` and `GLOSSARY.md`. Every other document lives under `docs/` with a lower-case, kebab-case name. Root files such as `DEVELOPING.md`, `DEVELOPMENT.md`, and a root protocol spec move to `docs/`.

This rule is about documents, not configuration files. D-035 also allows a root file that another rule owns: a `NOTICE` (HYG-3 in [repo-hygiene.md](repo-hygiene.md), D-085). There is never a `CLAUDE.md` (AI-5 in [ai-docs.md](ai-docs.md), D-108). `SECURITY.md` sits under `.github/`, not at the root (HYG-9).

`docs/` is for documents. README screenshots and store art live in the asset folder defined in [assets.md](assets.md), not in `docs/`.

### DOCS-2: Developer guide

- **Status:** Decided (D-035). Existing (4 of 15 have `docs/development.md`; 2 more use a root file under another name)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/docs/development.md`; `repilot/DEVELOPING.md` for the right size in a small app
- **Why:** The README's Developers section links here (README-4), so the target must exist under one name.

`docs/development.md` covers prerequisites, build, run, test, debug, and the release procedure or a link to it. In a small app it is the only developer document and may include a short architecture section. Kit repos use their README as the developer guide (README-8) and do not need this file. hub-display-switch and wsl-monitor have no developer guide today; their build steps sit in the README.

Commands are named as they appear in [dev-scripts.md](dev-scripts.md). The guide explains what a command proves; it does not copy another repo's documentation (DOCS-7).

### DOCS-3: Architecture document

- **Status:** Decided (D-035). Existing (3 of the 6 repos with an architecture document keep it at the root: immich-drive, printstream, game-is-up)
- **Applies to:** all
- **Reference:** `immich-drive/ARCHITECTURE.md` (small app); `printstream/ARCHITECTURE.md` (monorepo)
- **Why:** People and agents both look for architecture at the root, and one copy stops `AGENTS.md` from re-explaining it.

Create `ARCHITECTURE.md` when the area-to-files table in `AGENTS.md` is no longer enough: more than one process, a server, a protocol, or a sync engine. It describes ownership boundaries, data flow, and the decisions that would surprise a newcomer. Where the app writes data outside its own folder, it lists every location (see DOCS-9).

A product with several large subsystems may split into `docs/<area>-architecture.md` files, with the root `ARCHITECTURE.md` as the overview that links them. `AGENTS.md` points to the architecture document and does not repeat it (see AI-2 in [ai-docs.md](ai-docs.md)). Decision records, where kept, go in `docs/adr/` as in little-launcher.

### DOCS-4: Glossary format

- **Status:** Decided (D-035). Existing (2 of 3 glossaries share the format: printstream, store-sign-controller)
- **Applies to:** all
- **Reference:** `store-sign-controller/GLOSSARY.md`
- **Why:** The `_Avoid_` line is what stops three words for one thing from spreading through code, UI text, and listings.

`GLOSSARY.md` has an `# <Product>` heading, a one-sentence scope, a `## Language` section, and then one entry per term: the term in bold, its definition, and an `_Avoid_:` line listing rejected synonyms. Product and feature names in the glossary match [branding.md](branding.md). `AGENTS.md` links to the glossary in its guidance map.

### DOCS-5: One copy per topic, with a scope header

- **Status:** Decided (D-026). Existing (1 of 15 opens topic docs with a scope header: little-launcher)
- **Applies to:** all
- **Reference:** `little-launcher/docs/versioning.md`, indexed from `little-launcher/docs/README.md`
- **Why:** A topic kept in two places drifts; printstream keeps most guides twice and already gives two locations for one of them (FIX-040).

Each topic exists once. If people read it, it lives in `docs/` and agents are routed to it from the guidance map in `AGENTS.md`. Material only an agent needs lives in `.agents/guides/` (see AI-6 in [ai-docs.md](ai-docs.md)). Nothing lives under a tool folder such as `.codex/docs/`.

A topic doc opens with two lines: what it covers ("Scope") and the paths it governs ("Governs"). When `docs/` holds more than about five files, add `docs/README.md` as an index with one line per document.

### DOCS-6: Reference docs kept apart from working notes

- **Status:** Proposed
- **Applies to:** all
- **Reference:** `printstream/docs/private/` (notes kept out of the reference set and stripped from the public export)
- **Why:** Dated validation records and session notes bury the documents a reader came for; store-sign-controller's `docs/` mixes both across 22 files.

`docs/` holds reference documents that are kept true. Dated investigations, validation records, and design-session notes go in `docs/notes/`, or are deleted once their conclusions are written into a reference doc. In a repo with a public export, material that must not be published goes in `docs/private/` and the export removes it.

### DOCS-7: Link to shared documentation; do not copy it

- **Status:** Decided (D-005). Existing (0 of 6: every Windows-building repo carries its own copy of the build guide)
- **Applies to:** all
- **Reference:** none yet; `wsl-monitor/.codex/docs/windows-development.md` is the only corrected copy
- **Why:** A fix made in one copy does not reach the others (FIX-037, FIX-038).

Documentation for something shared (the Windows build bridge, devkit, winui-app-kit, apps-promo) lives once, in the project that owns it. An app keeps only what is specific to it, such as its platform and its submodules, and links to the rest by URL. The link must resolve outside this machine: no sibling-checkout paths (see AI-13 in [ai-docs.md](ai-docs.md)). This is the documentation side of README-4, which says the same for the devkit section of the README.

### DOCS-8: Privacy policy

- **Status:** Decided (D-017). Existing (4 of the 11 end-user apps have a policy: persistent, printstream, phonelock, sonarphone-nmea-bridge; in three formats and three hosting places)
- **Applies to:** Windows, Web, Android
- **Reference:** `phonelock/PRIVACY.md` for the outline; Persistent's page on its own domain for hosting
- **Why:** Stores register the policy URL, so it must survive a repo move, and reviewers read it.

Every store-listed product has a policy published as a page on the company site or the product's own domain, written from one template, using the single company contact (see BRAND-9 in [branding.md](branding.md)). The app's About surface (BRAND-8 in [branding.md](branding.md)), the README and every store listing link to that page (see [store-listings.md](store-listings.md)). A GitHub blob URL or a GitHub Pages URL is not used as the registered address, because it changes when the repo moves (D-017). phonelock's GitHub file and sonarphone-nmea-bridge's GitHub Pages page move to website pages.

The outline is phonelock's: the short version, what is handled, who can read it, services and servers, retention, children, your choices, contact. A policy names every network call the app makes on its own, including update checks and the promo catalog fetch (see [promo.md](promo.md)), even when no personal data is sent. No policy ships with a placeholder in it (FIX-012); the contact address fix is FIX-011.

This rule owns the privacy policy; LOG-10 in [logging.md](logging.md) was merged into it.

### DOCS-9: Data handling statements

- **Status:** Decided (D-023, D-025). Existing (1 of 11 end-user apps states that it has no telemetry: phonelock; no app has product telemetry or crash reporting)
- **Applies to:** Windows, Web, Android, HA
- **Reference:** `phonelock/PRIVACY.md` "The short version"
- **Why:** Users, store reviewers, and self-hosters each ask what is collected and what is left behind, and today the answer is in nobody's docs.

In three parts:

- **Telemetry:** the README and the privacy policy each carry one line stating that the app collects no telemetry or crash reports; the policy behind that line is LOG-9 in [logging.md](logging.md). If that ever stops being true, the line changes in the same release.
- **Where data lives:** `ARCHITECTURE.md`, or `docs/development.md` in a small app, lists every place the app writes (data folder, registry, startup entry, sync roots, server-side data) and what remains after uninstall. For a packaged (MSIX) Windows install, files under the app's AppData are redirected into the package's own folder, which Windows deletes on uninstall unless the manifest opts out of that virtualization. The redirection applies only to folders the packaged app creates: a folder that already existed (for example from an earlier unpackaged build) is written in place and survives uninstall (seen in hub-display-switch, 2026-11-16); what survives is what the app writes elsewhere (registry entries outside the package, sync roots, user documents) and everything an unpackaged dev build writes. State which case applies, and mark the list unconfirmed until a real uninstall has been checked.
- **Backup, export, deletion:** what a product must provide and state is SEC-13 in [security.md](security.md). It is written in `docs/deployment.md` for a product that runs on a server, and beside the data-location list otherwise.

### DOCS-10: Protocol and contract documents

- **Status:** Decided (D-035). Existing (2 of 2 Android repos keep a protocol document, both at the root under different names)
- **Applies to:** all
- **Reference:** `phonelock/PROTOCOL.md` for content; `printstream/docs/data-event-contract.md` for placement
- **Why:** A wire format or host contract is the one document where a silent drift from the code breaks installed copies.

A byte-level protocol, message format, or host-to-page contract gets its own document at `docs/protocol.md` or `docs/<name>-contract.md`. `AGENTS.md` names it as the source of truth and says it must change in the same commit as the code. Repos that share a contract use the same file name for it, as persistent and printstream do for `auth-architecture.md` and `data-event-contract.md`.

### DOCS-11: No changelog file; release notes are per version

- **Status:** Decided (D-058). Existing (15 of 15 have no `CHANGELOG.md`; 2 of 9 releasing repos commit a notes file per version)
- **Applies to:** all
- **Reference:** `immich-drive/release-notes/`
- **Why:** One user-facing text per version can be sent to GitHub and the store unchanged; a running changelog duplicates it.

Repos do not keep a `CHANGELOG.md` or a history section in the README (README-6). Release notes live in `release-notes/` at the repo root, outside `docs/`, because CI reads them. Their path grammar (version, lane and locale), format, wording, and the checks around them are owned by REL-6 in [releases.md](releases.md). A product that shows its changelog inside the app (printstream) keeps the structured file the app reads and treats it as that product's release notes.

### DOCS-12: Docs stay true and follow the writing rules

- **Status:** Existing (docs are generally current; em dashes or hard-wrapped prose remain in 8 of 15, see FIX-044 and FIX-045)
- **Applies to:** all
- **Reference:** `little-launcher/AGENTS.md` (the "what changed, update these docs" table)
- **Why:** A doc that names a file, version, or count that no longer exists is worse than no doc, because agents act on it.

A change that alters behaviour, a command, a path, or a requirement updates the affected docs in the same commit. `AGENTS.md` lists which docs each kind of change touches (see AI-2 in [ai-docs.md](ai-docs.md)). Docs avoid facts that go stale on their own: counts of sibling apps, release tags, and descriptions of one machine.

Prose follows the writing section of the user-level agent rules (kept outside this repo), which is not restated here.
