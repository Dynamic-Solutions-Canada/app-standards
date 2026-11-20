# README

How each repo's `README.md` is shaped. A README is the front door for users first and developers second; deep developer material lives in the docs set (see [docs.md](docs.md)).

Counts are out of the 11 end-user apps in scope (hub-display-switch, wsl-status, little-launcher, repilot, immich-drive, persistent, printstream, game-is-up, store-sign-controller, phonelock, sonarbridge) unless a rule says otherwise.

## Templates

There are three README shapes. Pick the one that matches the repo.

| Template | Use for | Reference |
| --- | --- | --- |
| Product | Any app with end users | `repilot/README.md` (short, every section present); `persistent/README.md` (several platforms and badges) |
| Add-on | A plugin or integration for another product, such as the Home Assistant integration | `printstream-home-assistant/README.md` |
| Tool | Shared kits and developer tooling: winui-app-kit, apps-promo, devkit | `wsl-status/external/winui-app-kit/README.md`; the out-of-scope `android-vm/README.md` is also a good model |

## Rules

### README-1: Product README section order

- **Status:** Decided (D-034). Existing (about 5 of 11 follow the order; section names vary in all of them)
- **Applies to:** Windows, Web, Android
- **Reference:** `repilot/README.md`, `persistent/README.md`
- **Why:** Users scan for what it is, where to get it, and what it does, in that order. Developers know to scroll to the end.

The sections, in order:

1. **Header** (see README-2).
2. **Lead paragraph:** one short paragraph saying who it is for and what problem it solves.
3. `## Features`: bullets in the form `**Label:** sentence`.
4. `## Get it`: store badges or download links, plus requirements (minimum OS version and so on).
5. Optional product sections, such as `## How it works` or `## Languages`.
6. `## Developers`: links only (see README-4).
7. `## License` (see README-5).

Use these exact headings, so the same section can be found in every repo. Today the same content appears as "Using it", "Overview", "What the key can do", "What makes it different", and "What it does".

### README-2: Centred header

- **Status:** Decided (D-034). Existing (5 of 11: little-launcher, repilot, immich-drive, persistent, store-sign-controller)
- **Applies to:** Windows, Web, Android
- **Reference:** `repilot/README.md` lines 1 to 31

A centred HTML block with: the app icon (one size, from the repo's asset folder; see [assets.md](assets.md)), `<h1>` with the product name, a one-sentence tagline, the store badges or primary call to action, and up to three screenshots. The product name (D-012) and tagline must match the store listing (see [store-listings.md](store-listings.md)).

### README-3: Official store badges

- **Status:** Existing (4 of the 6 apps with a store listing use an official badge image: little-launcher, repilot, immich-drive, persistent; printstream uses badges but a plain header)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/README.md` (Microsoft Store and Google Play together)

Use the official Microsoft Store and Google Play badge images, linked to the listing, at a consistent height. Do not use plain text links in place of badges, and do not hardcode a release tag (as `sonarbridge/README.md` does), because it goes stale on every release.

### README-4: Developers section links out

- **Status:** Decided (D-035). Existing (about 4 of 11: persistent, store-sign-controller, printstream, sonarbridge)
- **Applies to:** all
- **Reference:** `store-sign-controller/README.md` `## Developers`

The Developers section is a short list of links to the developer docs (see [docs.md](docs.md)): build and run, architecture, release process. Build commands, setup, deployment, environment variables, and architecture do not live in the README. game-is-up (377 lines) is the main repo that keeps a developer manual in its README.

Tool and devkit setup is linked, not copied. Do not include a "Remote development with Devkit" section; link to devkit's README instead. The section is currently copied into every devkit adopter in three drifted versions.

### README-5: License section last

- **Status:** Decided (D-008, D-006). Existing (5 of 11 have one: little-launcher, repilot, immich-drive, persistent, printstream)
- **Applies to:** all, including add-ons and tools
- **Reference:** `repilot/README.md` `## License`

`## License` is the last section. It names the licence, links the licence file, and states the copyright holder. Nothing comes after it; little-launcher has a section after its licence today. The spelling is `License` (American), matching the file name and GitHub's licence detection; winui-app-kit's `## Licence` is the outlier.

### README-6: No changelog or feature inventory

- **Status:** Proposed
- **Applies to:** all
- **Reference:** `repilot/README.md` (538 words)

Features are a short, benefit-led list, not every option the app has. Change history belongs in release notes (see [releases.md](releases.md)). little-launcher's roughly 40-bullet Overview (2,812 words) is the counter-example. As a guide, keep a product README under about 1,000 words.

### README-7: Add-on README

- **Status:** Existing (1 of 1)
- **Applies to:** HA
- **Reference:** `printstream-home-assistant/README.md` (source: `printstream/integrations/home-assistant/README.md`)

The header is `# <Product> for <Host>`. Then: one sentence linking the parent product and website, an image showing what the user gets inside the host (not the parent product's hero), `**Label:** sentence` features, `## Requirements`, `## Installation` (store or registry first, manual second), `## Setup`, a host-specific reference section, `## Issues`, and `## License`. If the repo is generated from another repo, say where development happens and link to it.

### README-8: Tool README

- **Status:** Existing (2 of 3: winui-app-kit, devkit)
- **Applies to:** Kit
- **Reference:** `wsl-status/external/winui-app-kit/README.md`

What it is and who uses it, how to add it to an app, the commands or API at a glance, and limits or gotchas, then `## License`. A tool README is the canonical usage doc for that tool; app READMEs link to it rather than repeating it.

### README-9: Images live in the repo's asset folder

- **Status:** Decided (D-102)
- **Applies to:** all

Where README images live is owned by ASSET-1 and ASSET-10 in [assets.md](assets.md) and is not restated here: the icon comes from `assets/generated/` and screenshots from `store-assets/web/screenshots/`, by relative path, so they render on GitHub and in clones. The README side of the rule is that every image it shows is one of those files, with an extension that matches its content; the `repilot/docs/*.png` files that are really JPEG data are the counter-example (FIX-019).

### README-10: Prose style

- **Status:** Existing (no hard-wrapping in most READMEs; em dashes in 6 of 15 in-scope READMEs)
- **Applies to:** all

README prose follows the writing section of the user-level agent rules (kept outside this repo) (no hard wrapping, no em dashes), which is not restated here. The rule exists so that README parity checks include it: little-launcher, repilot, immich-drive, persistent, game-is-up, and sonarbridge currently contain em dashes, and hub-display-switch and sonarbridge are hard-wrapped.
