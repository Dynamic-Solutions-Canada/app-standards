# Store listings

Where the text and images for each store listing live, what shape they take, and how the README, the listing and the website page are kept saying the same thing. The asset sizes themselves are in [assets.md](assets.md); the locale set is in [locales.md](locales.md); how a release reaches a store is in [distribution.md](distribution.md) and [releases.md](releases.md).

Counts are out of the 9 in-scope apps that have, or are being prepared for, a Microsoft Store or Google Play listing (hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, persistent, printstream, phonelock, sonarbridge) unless a rule says otherwise. The Home Assistant integration is the one HACS listing. game-is-up and store-sign-controller have no listing.

## Listing surfaces

| Surface | Source in the repo | Published by | Reference |
| --- | --- | --- | --- |
| Microsoft Store | `store-assets/microsoft/` | CI, from the listing source (LIST-9) | `persistent/store-assets/microsoft/`, `hub-display-switch/store-assets/microsoft/` |
| Google Play | `store-assets/play/` | CI, from the listing source (LIST-9) | `persistent/apps/mobile/store/listings.json`, `persistent/.github/workflows/play-listing.yml` |
| HACS | `hacs.json`, the README and `custom_components/<domain>/brand/`, where HACS requires them | The repo itself is the listing | `printstream-home-assistant/` |
| Website page and README | `store-assets/web/` | The website pulls from the app repo (see [assets.md](assets.md), ASSET-12) | `persistent/README.md` (embeds its Play graphics directly) |

## Proposed layout

One layout for every repo. A monorepo keeps it at the repo root, not under each app.

```
store-assets/
  README.md                        how to capture, check and publish
  manifest.json                    ordered screenshots per store: file, source, theme, caption, alt
  raw/                             untouched captures, kept so captions can change without recapturing
  microsoft/
    listings.json                  schemaVersion, locales{ title, shortDescription, fullDescription, features, keywords, screenshotCaptions }
    listings.md                    limits, claims to keep true, console-only fields
    screenshots/NN-subject-theme.png
  play/
    listings.json                  schemaVersion, locales{ title, shortDescription, fullDescription }
    listings.md                    limits, claims to keep true, data-safety answers with reasons, open questions, console-only fields
    icon.png                       512 x 512
    feature-graphic.png            1024 x 500
    screenshots/phone/NN-subject-theme.png
    screenshots/tablet/NN-subject-theme.png
  web/
    social-card.jpg                1200 x 630
    screenshots/                   the set the README and the website page use
release-notes/
  <version>.md                     the en-US notes; see LIST-10. The path grammar is set in releases.md (D-058)
  <version>.<identity>.md          one per other wording catalog, in a localized app only
```

HACS has no folder here: `hacs.json` and `brand/` stay where HACS requires them (LIST-11).

## Rules

### LIST-1: Listing text lives in the repo

- **Status:** Decided (D-066). Existing (5 of 9 keep listing text in the repo: hub-display-switch, persistent, printstream, phonelock, sonarbridge)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/apps/mobile/store/listings.json`
- **Why:** Text that exists only in a store console cannot be reviewed, translated in one pass, or checked against what the app does.

Every field a user can read on a listing (title, descriptions, feature bullets, keywords, screenshot captions) has its source in the repo. The store console is an output, never the place where wording is edited. An app prepares this source before its first submission, so the first listing is reviewed like any other change.

### LIST-2: One `store-assets/` folder at the repo root

- **Status:** Decided (D-066). Existing (2 of 9: hub-display-switch, persistent in part)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/store-assets/`
- **Why:** One place per repo means one validator, one capture guide, and no sync step between two copies.

Use the layout above: one subfolder per store, plus `web/` for the README and website set. In a monorepo the folder stays at the root, since the listing describes the product, not one build target. Persistent currently splits material between `store-assets/` and `apps/mobile/store/` and syncs between them; printstream keeps it under `apps/<target>/store/`. Both converge on the root folder. Folders named `play/`, `docs/store/` or `apps/<target>/store/` are not used for new work.

### LIST-3: `listings.json` shape

- **Status:** Decided (D-066). Existing (2 of 9: persistent, hub-display-switch)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/apps/mobile/store/listings.json` (Play), `hub-display-switch/store-assets/microsoft/listings.json` (Microsoft)
- **Why:** The same shape for both stores lets one validator and one translation pass serve both, and it holds any number of locales without a schema change.

Each store has one `listings.json`:

- `schemaVersion` (currently `1`) and a `locales` object keyed by locale identity, in the order given in [locales.md](locales.md).
- Each locale record uses semantic field names: `title`, `shortDescription`, `fullDescription` for both stores; the Microsoft file adds `features`, `keywords` and `screenshotCaptions`.
- An alias locale is `{ "copyFrom": "<identity>" }`, not a duplicated record (`en-CA` copies `en-US`, `fr-CA` copies `fr-FR`). The publisher expands aliases, because both stores need a full record per listing locale.
- Field names are not the store's own import IDs. Mapping to Partner Center or Play fields is the publisher's job.
- Release notes are not a field in this file (LIST-10).

Ordered screenshot paths belong in `manifest.json` (LIST-8) and console-only settings in `listings.md` (LIST-4), so a single-locale file that also carries assets and console fields, as printstream's `listing.json` does, moves to this shape.

### LIST-4: `listings.md` beside each `listings.json`

- **Status:** Existing (2 of 9 have the companion file: hub-display-switch, persistent; phonelock has the same content as separate drafts)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `hub-display-switch/store-assets/microsoft/listings.md` (limits and claims), `phonelock/play/data-safety.md` (answers with reasons)
- **Why:** A listing makes promises. Writing down which ones must stay true, and why each console answer was given, is what lets a later release check them.

The companion file holds what the JSON cannot:

- **Limits:** the character limits in force and the longest current value per field.
- **Claims to keep true:** each factual promise the listing makes (privacy, offline behaviour, supported versions, languages), so a release that changes behaviour knows which wording to revisit.
- **Console-only fields:** category, pricing, age rating, privacy policy URL, and for Play the data-safety answers with the reason for each.
- **Open questions** before the next submission.

It does not hold a second copy of the listing text, and it never holds reviewer credentials. How reviewer and demo access is provided is covered in [security.md](security.md) (D-083). The privacy policy URL and contact address follow D-017; FIX-011 tracks the one listing draft that differs.

### LIST-5: Required fields and limits

- **Status:** Existing (2 of 9 enforce limits in a validator: persistent, printstream; hub-display-switch records them)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/apps/mobile/scripts/play-metadata.mjs` (Play limits in code), `hub-display-switch/store-assets/microsoft/listings.md` (Microsoft limits)
- **Why:** A translated string that is over the limit fails at upload time, in one locale, after the release is tagged. Checking in the repo moves that failure to review.

Every locale record carries every required field, and the validator rejects over-length text instead of truncating it.

| Store | Field | Limit |
| --- | --- | --- |
| Microsoft Store | `shortDescription` | 270 characters |
| Microsoft Store | `fullDescription` | 4,000 characters |
| Microsoft Store | `features` | up to 20, each up to 200 characters |
| Microsoft Store | `keywords` | up to 7, each up to 40 characters, 21 words combined |
| Microsoft Store | `screenshotCaptions` | one per screenshot, up to 200 characters |
| Microsoft Store | release notes | 1,500 characters |
| Google Play | `title` | 30 characters |
| Google Play | `shortDescription` | 80 characters |
| Google Play | `fullDescription` | 4,000 characters |
| Google Play | release notes | 500 characters |

The Microsoft `title` is the name reserved in Partner Center and must match it exactly (see [branding.md](branding.md)). These limits are the stores' and can change; when a store changes one, update this table and the validator together. French and German run longest, so write the English text with room to spare.

### LIST-6: Tool-specific trees are generated, never a second hand-kept copy

- **Status:** Decided (D-066). Proposed (no repo generates a tool tree; 2 of 9 keep the text twice by hand: persistent, sonarbridge)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/.github/workflows/play-listing.yml` (publishes straight from the JSON, with no intermediate tree)
- **Why:** Two hand-kept copies of the same sentence drift, and the copy that drifts is the one nobody publishes from.

Where a tool demands its own layout (fastlane's `metadata/android/<locale>/` text files, a Partner Center import file), that tree is generated from `listings.json`, or the publisher reads the JSON directly. A generated tree is never edited by hand. Markdown mirrors of the listing text (persistent's `listing.md`, sonarphone's `docs/PLAY_LISTING.md`) are removed once nothing reads them. The same applies to images: one committed copy of each store image, not one in `docs/` and one in the tool's tree.

### LIST-7: Listing locales equal UI locales

- **Status:** Decided (D-010, D-098). Existing (2 of the 7 localized apps: persistent, sonarbridge)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `sonarbridge/fastlane/metadata/android/` (nine listing locales matching nine UI locales)
- **Why:** A listing in English for an app that runs in Japanese hides the translation from the people it was made for, and a Japanese listing for an English-only app misleads them.

An app's listing is published in exactly the locales its interface supports ([locales.md](locales.md), LOC-1 and LOC-3): all nine identities for a localized app, one for an English-only app. Listing wording uses the same terms as the in-app catalogs. A release is not complete until listing text exists for every wording catalog. Under D-098, screenshots are captured once, in English, and serve every listing locale, as persistent and sonarbridge do today. Captions and alt text are translated.

### LIST-8: Screenshots are named, ordered by a manifest, and captured from the real app

- **Status:** Existing (1 of 9 has a manifest and a scripted pipeline: persistent)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/store-assets/manifest.json`, `persistent/store-assets/README.md`
- **Why:** Order, caption and alt text are part of the listing. Keeping them beside the files means a reorder or a caption fix is a one-line change, not a recapture.

- **Names:** `NN-subject-theme.png`, two-digit order first, lower-case kebab subject, `light` or `dark` last (`00-current-dark.png`). Drop the theme only when the app has one theme.
- **Manifest:** `store-assets/manifest.json` lists, per store, each file with its raw source, theme, caption and alt text. The captions in `listings.json` and the manifest order must agree in count.
- **Raw captures** stay in `store-assets/raw/` so framing and captions can change without recapturing.
- **Content:** captured from the real installed app with synthetic data. No personal names, device names, network names, account details or addresses.
- **Look (decided 2026-11-17):** every screenshot shows the app in dark mode. Windows screenshots are framed centred on the Windows 11 default dark wallpaper (the dark "bloom", `C:\Windows\Web\Wallpaper\Windows\img19.jpg`), read from a Windows machine at render time and never committed to a repo. Android screenshots are dark mode with no added frame.
- **Approval:** new or changed screenshots, in a store listing, a README or the website, are shown to the maintainer and approved before they are committed as listing assets or published.
- **Sizes and counts** are in [assets.md](assets.md), ASSET-9.
- **Scripts:** capture, render, check and sync are package scripts, documented in `store-assets/README.md`. Persistent's `shots:*` set is the model; script naming follows [dev-scripts.md](dev-scripts.md).

### LIST-9: CI is the only publisher, and it validates first

- **Status:** Existing (2 of the 4 apps with a Play listing source publish from CI with a check mode: persistent, printstream)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `persistent/.github/workflows/play-listing.yml`, `printstream/apps/desktop/scripts/windows-store-listing.mjs`
- **Why:** A listing edited by hand in a console is unreviewed and silently replaced by the next automated publish.

- The listing is published by a workflow, from the committed source, on demand or as part of a release (see [ci.md](ci.md) and [distribution.md](distribution.md)).
- The publisher has a **check mode** that compares the source with the live listing and writes nothing. Run it before a release and after any manual console change.
- Before any upload it **validates**: every locale present, every required field present, limits (LIST-5), `copyFrom` targets exist, every manifest file exists, and image dimensions, count, alpha and duplicates (ASSET-9).
- A listing source that no publisher reads, as hub-display-switch's is today, is a draft. It still follows this layout so that wiring the publisher is the only remaining step.

### LIST-10: Store release notes come from the release-notes source

- **Status:** Decided (D-058, D-010). Existing (2 of 9 feed one committed file to both the store and GitHub and fail when it is missing: immich-drive, printstream; 2 of 9 publish localized notes: persistent, sonarbridge)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `immich-drive/release-notes/` with `.github/scripts/set-store-release-notes.ps1`; `persistent/apps/mobile/store/releases/<version>.json` for localized notes as they are kept today (JSON, which D-058 replaces with one Markdown file per wording catalog)
- **Why:** Notes kept as a field in the listing file, a separate text file and a GitHub body are three texts for one release, and only one of them is what CI sends.

Release notes are per version, not per store. Their home, path grammar and format are owned by REL-6 in [releases.md](releases.md); under D-058 it is `release-notes/<version>.md` for the `en-US` text, plus one `release-notes/<version>.<identity>.md` per other wording catalog in a localized app, committed before the tag, with the same text sent to every store and the GitHub release. This file adds three listing-side constraints:

- The notes fit the shortest limit they are sent to (500 characters for Play, 1,500 for the Microsoft Store), and the publisher rejects longer text.
- `listings.json` has no `releaseNotes` field. Persistent and hub-display-switch carry one today; it is removed when the per-version source exists.
- For a localized app the notes ship in the listing locales (LIST-7, D-010). The publisher reads one file per wording catalog and fills the two Canadian identities through the alias mapping, as it does for `copyFrom` in the listing.

Text meant only for a GitHub post, such as the AI-disclosure line, is added by the step that posts to GitHub and is not part of the shared source.

### LIST-11: HACS listing

- **Status:** Decided (D-012). Existing (1 of 1 for the layout)
- **Applies to:** HA
- **Reference:** `printstream/integrations/home-assistant/hacs.json` (exported to `printstream-home-assistant/`)
- **Why:** HACS builds the listing from files at fixed paths, so the repo layout is the listing format.

The HACS listing is `hacs.json` (display name, minimum Home Assistant version, `render_readme`), the README as the listing body (see [readme.md](readme.md), README-7), and `custom_components/<domain>/brand/` for the icon and logo (ASSET-11). They stay at the paths HACS requires and are not mirrored under `store-assets/`. The display name in `hacs.json`, `manifest.json` and `strings.json` is the same name, "PrintStream" (D-012). The README shows an image of the integration inside Home Assistant, not the parent product's hero. Release notes follow LIST-10.

### LIST-12: README, listing, website page and promo catalog say the same thing

- **Status:** Decided (D-012, D-006, D-017). Proposed (no repo checks it; immich-drive has three spellings of its name across repo, site and catalog)
- **Applies to:** all
- **Reference:** `persistent/README.md` (icon and screenshots embedded straight from the listing graphics, so the two cannot differ)
- **Why:** A user who arrives from the Store, the website or GitHub should recognise the same product. Each small difference in name, tagline or claim reads as a different app.

The listing source is the reference text. These must match it:

| Item | README | Store listing | Website page | Promo catalog |
| --- | --- | --- | --- | --- |
| Product name | `<h1>` | `title` | page title | entry name |
| Tagline | header tagline | first sentence of `shortDescription` | page subtitle | entry description |
| Feature claims | `## Features` | `features` or `fullDescription` | feature list | n/a |
| Icon | from `assets/` | store icon | from `assets/` | from `assets/` |
| Screenshots | subset of `store-assets/web/screenshots/` | `store-assets/<store>/screenshots/` | `store-assets/web/screenshots/` | n/a |
| Publisher, privacy policy URL, minimum OS version, languages | where stated | console fields in `listings.md` | where stated | n/a |

- Wording need not be identical in length (a README can say more), but it must not contradict the "claims to keep true" in `listings.md` (LIST-4).
- The README uses relative paths into `store-assets/` and `assets/`; it does not hot-link a hosted copy, and it does not keep a separate screenshot set in `docs/`.
- Store badges in the README follow [readme.md](readme.md), README-3.
- Updating the website page and the promo catalog is a release step, owned by the shared post-release skill (D-063); see [promo.md](promo.md) and [releases.md](releases.md).
