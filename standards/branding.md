# Branding

The names and identifiers a product presents to the world: product name, publisher, GitHub owner, package ids, and the links an app shows or fetches. Visual identity (icons, screenshots) is in [assets.md](assets.md); listing text is in [store-listings.md](store-listings.md).

Most rules here rest on decisions made on 2026-11-16, because the values are registered with stores or shipped in installed apps. Counts are out of the 15 in-scope repos unless a rule says otherwise.

## Rules

### BRAND-1: One name per product

- **Status:** Decided (D-012). Existing (4 of 5 standalone Windows apps use one spaced name everywhere; ImmichDrive, PhoneLock, and the Home Assistant integration each have more than one)
- **Applies to:** all
- **Reference:** `little-launcher` ("Little Launcher" in the manifest, README, About page, store, and site)
- **Why:** A user who sees three names cannot tell whether they are three products, and search finds only one of them.

Each product has one display name, written the same way in the manifest or app label, the About page, the README title, `AGENTS.md`, the store listing, the promo catalog, and the website. The name is recorded in the registry (BRAND-10). The three products that had more than one name are settled by D-012:

| Product | Name | Retire |
| --- | --- | --- |
| ImmichDrive | "ImmichDrive" everywhere, including the Store listing and the site | "Drive for Immich" (Store listing and site today) and "Immich Drive" (README and prose today) |
| PhoneLock | "PhoneLock" | "Phone Lock" (FIX-023) |
| PrintStream's Home Assistant integration | "PrintStream" | "PrintStream Bridge", which collides with the bridge agent |

Display names are spaced words in title case, except the two that were settled as one word: ImmichDrive and PhoneLock. Elsewhere the unspaced PascalCase form is for code only: project names, namespaces, and data folders (see [repo-layout.md](repo-layout.md)). A former product name is not used in docs or new code (FIX-036); where it survives in a permanent identifier, the registry says so.

### BRAND-2: Publisher display name

- **Status:** Decided (D-006). Existing (5 of 6 WinUI manifests use "Dynamic Solutions (CA)"; Persistent's desktop app uses "Dynamic Solutions Canada")
- **Applies to:** all
- **Reference:** `repilot/RepilotMSIX/Package.appxmanifest`
- **Why:** The Microsoft Store rejects a package whose publisher display name does not match the account exactly.

The publisher display name is "Dynamic Solutions (CA)", used unchanged in every manifest; it must match what Partner Center shows for the account. Persistent's desktop app is corrected to it. The same name is used in store listings and on the promo page.

### BRAND-3: Copyright holder

- **Status:** Decided (D-006). Existing (0 of 7 licence files name the company; all seven name the individual owner)
- **Applies to:** all
- **Reference:** `repilot/LICENSE.md`
- **Why:** The licence, the README, and the About page must name the same holder for the grant to be clear.

The copyright holder is the company, written "Dynamic Solutions", for every repo except the two personal projects, game-is-up and store-sign-controller, whose holder is the individual owner (the repo owner's legal name). printstream and its Home Assistant integration keep their own licence (D-008), and the holder named in it was not part of the decision. The same holder appears on every About page's copyright line (BRAND-8). The copyright line reads `Copyright (c) <first year>-<current year> <holder>`, or a single year for a new repo, and is identical in the licence file (HYG-1 in [repo-hygiene.md](repo-hygiene.md)), the README's License section (README-5), and any About page that shows it.

### BRAND-4: GitHub owner and repo names

- **Status:** Decided (D-009). Existing (9 of 15 are in the `Dynamic-Solutions-Canada` organisation; 4 are under the personal account; 2 are under `PrintStreamApp`)
- **Applies to:** all
- **Reference:** `wsl-monitor` (organisation owner, kebab-case name, current remote)
- **Why:** A known owner per repo makes every link predictable, and nine repos have already moved.

There are three owners, and each repo has exactly one:

| Owner | Repos |
| --- | --- |
| `Dynamic-Solutions-Canada` organisation | Every repo not named below. sonarphone-nmea-bridge and devkit move there from the personal account |
| `PrintStreamApp` | printstream and printstream-home-assistant |
| The maintainer's personal account | game-is-up and store-sign-controller, which are personal projects |

Repo names are lower-case kebab-case and match the product name (`little-launcher`, not `LittleLauncher`). devkit's package is renamed from its personal-account scope to `@dynamic-solutions-canada/devkit` when the repo moves, and its consumers update their dependency in the same change (D-009).

A move is planned against the registry (BRAND-10) and follows BRAND-11. sonarphone-nmea-bridge moves only together with its privacy page address and in-app links.

### BRAND-5: Links in shipped apps name the current owner

- **Status:** Existing (1 of the 4 Windows apps with an update checker names the current owner: little-launcher; defects are FIX-013 to FIX-018 and FIX-048)
- **Applies to:** Windows, Web, Android, HA, Kit
- **Reference:** `little-launcher/LittleLauncher/Services/UpdateService.cs` (owner and repo as two constants)
- **Why:** An installed app keeps the URL it shipped with, so a stale owner string turns a GitHub redirect into a dependency.

Every URL compiled into an app (About-page source link, update check, promo catalog, privacy link, package `homepage`, documentation URL) names the current owner and repo from the registry. The owner and repo are defined once per app, as constants or configuration, and every link is built from them; they are not retyped per page.

### BRAND-6: Windows package identity

- **Status:** Decided (D-014). Existing (4 of 6 WinUI apps use the Store-assigned identity)
- **Applies to:** Windows
- **Reference:** `immich-drive/ImmichDriveMSIX/Package.appxmanifest`
- **Why:** The identity is permanent once an app is published, and settings and updates are tied to it.

A Store app's identity name and publisher id are the values Partner Center assigns when the name is reserved. They are never invented, and never changed after the first submission. Under D-014, hub-display-switch and wsl-monitor go to the Store, and their names are reserved now, before any install base exists on a placeholder identity.

A permanent identity may carry an old product name; Repilot's does. That is recorded in the registry and left alone. Identities for development builds are covered in [distribution.md](distribution.md).

### BRAND-7: Android application ids

- **Status:** Decided (D-011). Existing (1 of 4 Android ids uses the company namespace: sonarphone-nmea-bridge; 2 of 4 use the product's own domain: persistent, printstream)
- **Applies to:** Android
- **Reference:** `sonarphone-nmea-bridge/android/app/build.gradle.kts`
- **Why:** An application id cannot change after the first Play upload, and it should show who published the app.

A new Android app uses `ca.dynamicsolutions.<app>`, or the reversed product domain when the product has its own (`app.printstream`, `ca.persistent.app`). No other namespace is used for new apps. phonelock's Play-bound parent app is renamed to `ca.dynamicsolutions.phonelock.parent` before its first Play upload. Published ids are permanent and are recorded in the registry.

### BRAND-8: About block and support links

- **Status:** Proposed (0 of 6 WinUI apps have every item; Persistent's desktop app is closest with 5 of 8). The support link target is Decided (D-086)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/apps/desktop/Persistent.Desktop/Pages/AboutPage.xaml`; `printstream/apps/web/src/components/HelpFeedbackDialog.tsx` for a web product
- **Why:** With no telemetry (D-023), a user who can find the logs and a place to send them is the only diagnostic channel there is.

Every app has an About surface with these items, in this order. This rule owns the list and the target of each link; the Windows page layout is UI-5 in [ui.md](ui.md), and what "Open logs folder" does on each stack is LOG-8 in [logging.md](logging.md).

1. Product name and version.
2. One-line description, matching the store tagline.
3. Check for updates (see [distribution.md](distribution.md)).
4. Source code link, where the repo is public.
5. Privacy policy link (DOCS-8 in [docs.md](docs.md)).
6. Support or feedback link.
7. Licence, and third-party notices where they ship (HYG-3 in [repo-hygiene.md](repo-hygiene.md)).
8. Open logs folder, on platforms where the user can reach it (see [logging.md](logging.md)).

Labels use the same wording in every app, on every stack. The wording is winui-app-kit's `kit.about.*` labels, which already have the family's translations; Android and web apps copy the text (and, when localized, the translations) from the kit's catalogs rather than writing their own:

| Item | Label (en-US) |
| --- | --- |
| Section and entry | About |
| Version | Version {version} |
| Updates | Updates; button: Check for updates |
| Links section | Links: Source code, Privacy policy, Support, License, Third-party notices |
| Diagnostics section | Diagnostics; action: Open logs folder |

Where each stack shows them:

- **Windows:** winui-app-kit's shared About page (UI-5 in [ui.md](ui.md)).
- **Android:** an "About" screen reached from the last row of Settings, with the same sections in the same order as Material list items (product name and version as the header, then Updates, Links, Diagnostics where the user can share logs). Play-distributed builds omit "Check for updates" when Play handles updates.
- **Web:** an "About" dialog or page reached from the main menu, with the same sections in the same order, plus a privacy link in the footer. "Check for updates" becomes the app's own update notice where it is a PWA, and is omitted otherwise.

The support link goes to the repo's GitHub issues page, which has an issue template (HYG-9 in [repo-hygiene.md](repo-hygiene.md)). printstream, whose repo is private, links to its help channel or the company contact (BRAND-9) instead (D-086).

### BRAND-9: One contact

- **Status:** Decided (D-017). Existing (2 contact addresses are in use across the 2 Android apps, one of them personal; FIX-011)
- **Applies to:** all
- **Reference:** `phonelock/PRIVACY.md` (uses the company address)
- **Why:** A personal address in a public policy or listing is hard to withdraw once stores have indexed it.

Privacy policies, store listings, `SECURITY.md`, and support links all give the single company contact address. The exception is the two personal projects, game-is-up and store-sign-controller, which give the individual owner's contact (D-017). No personal address is written into a tracked file; repos link to the policy or contact page instead. The address itself is recorded outside the standards, in the store accounts and the company site.

### BRAND-10: Identifier and URL registry

- **Status:** Decided (D-036). Existing (0 of 15)
- **Applies to:** all
- **Reference:** the table below
- **Why:** Three GitHub owners, four Android id schemes, and a promo URL on a pre-rename name mean a move can silently break update checks in installed copies.

The standards hold one registry of each product's permanent identifiers and every URL that shipped binaries fetch. It lives here, under this rule, and nowhere else; other topic files link to BRAND-10 instead of keeping their own list. It is updated in the same change that ships a new identifier or URL, and is checked before any rename or transfer (BRAND-11).

Values below are as found in the audit on 2026-11-16, with the decisions of the same day applied; a cell that names a decision describes a change still to be made. Visibility is the target set in D-004 ("public" means public now or to be made public), not a check of each repo's current setting. Store identities must be confirmed against Partner Center and Play Console before anyone relies on this table.

| Product | Display name | Repo today | Visibility (D-004) | Permanent identifier | URLs fetched or shown by shipped copies |
| --- | --- | --- | --- | --- | --- |
| Hub Display Switch | Hub Display Switch | `Dynamic-Solutions-Canada/hub-display-switch` | Public | Store ID `9P1S2QJFXZFW`, package identity `DynamicSolutionsCA.HubDisplaySwitch` (reserved 2026-11-16; the dev build keeps `TechnicallyReal.HubDisplaySwitch`, DIST-3); tray GUID `5b0f1c7e-3c1a-4e8e-9a35-6f1d2a7c4b11`; host window "Hub Display Switch Host"; single-instance mutex `HubDisplaySwitch`; startup task `HubDisplaySwitchStartup` | Promo catalog |
| WSL Monitor | WSL Monitor | `Dynamic-Solutions-Canada/wsl-monitor` | Public | Store ID `9MTQWTX87B8J`, package identity `DynamicSolutionsCA.WSLMonitor` (reserved 2026-11-16; manifest still on the placeholder) | Promo catalog |
| Little Launcher | Little Launcher | `Dynamic-Solutions-Canada/little-launcher` | Public | Store identity `27766TechnicallyReal.LittleLauncher` | Release check on the repo; About source link (FIX-013); promo catalog |
| Repilot | Repilot | Resolves to `Dynamic-Solutions-Canada/repilot`; app strings use the old owner (FIX-015) | Public | Store identity `27766TechnicallyReal.CopilotKeyRemapper`, an old product name kept permanently | Release check and About link on the old owner; promo catalog |
| ImmichDrive | ImmichDrive (D-012; the English Store listing was corrected 2026-11-16; the site still says "Drive for Immich") | `Dynamic-Solutions-Canada/immich-drive` (app strings fixed, FIX-016) | Public | Store identity `27766TechnicallyReal.ImmichDrive`; tray GUID `48633E3A-477E-41D2-8596-F1851C4D2045`; host window "ImmichDrive Host"; single-instance mutex `ImmichDrive.Instance`; startup task `ImmichDriveAutoStart` | Release check and About link on `Dynamic-Solutions-Canada/immich-drive`; promo catalog |
| Persistent | Persistent | `Dynamic-Solutions-Canada/persistent` | Public | Store identity `27766TechnicallyReal.Persistent`; Android id `ca.persistent.app` | Release check and About link on the old owner (FIX-014); privacy page on the product domain; promo catalog |
| PrintStream | PrintStream | `PrintStreamApp/printstream` (the published snapshot; stays under this owner, D-009) | Private, with a public snapshot | Android id `app.printstream` | Product domain; package `homepage` (FIX-018) |
| PrintStream for Home Assistant | PrintStream (D-012; the manifest and strings still say "PrintStream Bridge") | `PrintStreamApp/printstream-home-assistant` (stays, D-009) | Public | Integration domain `printstream` | Documentation URL in the cards (FIX-018) |
| Game's Up | Game's Up | Personal account (stays; a personal project, D-009) | Public | None recorded | None recorded |
| store-sign-controller | "Store Sign" in its README; not otherwise checked | Personal account (stays; a personal project, D-009) | Public | None recorded | None recorded |
| PhoneLock | PhoneLock (D-012; the parent app's label is FIX-023) | `Dynamic-Solutions-Canada/phonelock` | Public | Android ids `home.phonelock` and `home.phonelock.parent`; the parent app becomes `ca.dynamicsolutions.phonelock.parent` before the first Play upload (D-011), and is permanent after it | Release check on the repo, fetching assets by fixed file name; privacy policy as a GitHub file link, which moves to a website page (D-017) |
| sonarphone-nmea-bridge | SonarBridge | Personal account; moves to `Dynamic-Solutions-Canada` (D-009) | Public | Android id `ca.dynamicsolutions.sonarbridge` | Releases link on the personal account; privacy page on GitHub Pages, which moves to a website page (D-017) |
| apps-promo | n/a | `Dynamic-Solutions-Canada/apps-promo` | Public | The catalog URL itself | Every WinUI app fetches the catalog from a raw URL under the pre-rename owner and repo name (FIX-017) |
| devkit | n/a | Personal account; moves to `Dynamic-Solutions-Canada` (D-009) | Public | Package name `@dynamic-solutions-canada/devkit` (renamed from its personal-account scope), installed by git tag | None |
| winui-app-kit | n/a | `Dynamic-Solutions-Canada/winui-app-kit` | Public | None | None |

Shared values, one row each:

| Item | Value |
| --- | --- |
| Publisher display name | "Dynamic Solutions (CA)" (BRAND-2, D-006) |
| Copyright holder | "Dynamic Solutions"; the individual owner (the repo owner's legal name) for game-is-up and store-sign-controller (BRAND-3, D-006) |
| GitHub owner | `Dynamic-Solutions-Canada`, except printstream and its Home Assistant integration (`PrintStreamApp`) and the two personal projects (BRAND-4, D-009) |
| Android namespace for new apps | `ca.dynamicsolutions.<app>`, or the product's own reversed domain (BRAND-7, D-011) |
| Contact | The company contact address, recorded outside the standards (BRAND-9, D-017) |
| Product page | A page per product on the company site, or the product's own domain where it has one (see D-017) |

Not yet collected, and added as columns when the registry is next filled in: the per-app identifiers that must survive a kit migration (tray GUID, mutex name, startup task id and data folder name; see TRAY-3, TRAY-9 and TRAY-10 in [tray.md](tray.md) and CONF-1 in [settings-config.md](settings-config.md)). The standards repo itself is public, with its `audit/` working material kept private (D-004).

App privacy policies live at `https://www.dynamic-solutions.ca/privacy/<slug>`, where the slug is the display name lowercased with spaces as hyphens (decided 2026-11-16): `/privacy/hub-display-switch`, `/privacy/wsl-monitor`, `/privacy/little-launcher`, `/privacy/repilot`, `/privacy/immichdrive`, `/privacy/phonelock`, `/privacy/sonarbridge`. These URLs are permanent once a store listing or a shipped app links them (BRAND-11). `/privacy-policy` stays the website's own policy and indexes the app policies; Persistent and PrintStream keep their policies on their own domains.

### BRAND-11: A rename keeps old URLs working

- **Status:** Proposed
- **Applies to:** all
- **Reference:** FIX-017 states the requirement for the promo catalog
- **Why:** Installed copies cannot be patched; phonelock's move already showed that old hardcoded URLs live on in the field.

Before a repo, product, or published file is renamed or moved:

1. List every registry entry it touches.
2. Keep every URL that an installed copy fetches working for as long as those copies exist: leave the old repo name unused so the redirect holds, or keep serving the file at the old path.
3. Update the app's constants, `.gitmodules`, remotes, and scripts to the new name in the same piece of work (BRAND-5; HYG-11 in [repo-hygiene.md](repo-hygiene.md)).
4. Update the registry.

Release assets that an updater fetches by file name are covered the same way; see [releases.md](releases.md) before renaming one.
