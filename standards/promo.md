# Promo

The "Our other apps" page: which apps show it, how they include it, and how the catalog behind it is kept complete.

The page and its catalog live in the apps-promo repo, which owns its own usage docs: see [its README](https://github.com/Dynamic-Solutions-Canada/apps-promo) (local copy: `wsl-monitor/external/promo/README.md`) for how to add the submodule, how `scripts/refresh-catalog.ps1` regenerates the catalog, and how web consumers read it. This file says when and where an app uses it. Counts are out of the 6 WinUI apps (hub-display-switch, wsl-monitor, little-launcher, repilot, immich-drive, and persistent's desktop app) unless a rule says otherwise.

## Overview

| Stack | Shows sibling apps | How |
| --- | --- | --- |
| Windows | Yes, every WinUI app (PROMO-1) | Submodule plus shared-items import; footer navigation item |
| Web | Optional; persistent's web app does (PROMO-8, D-075) | `catalog.ts` and the bundled manifest from the same submodule |
| Android | No (PROMO-9, D-075) | Apps are listed in the catalog only |

## Rules

### PROMO-1: Every WinUI app includes the promo page

- **Status:** Existing (6 of 6)
- **Applies to:** Windows
- **Reference:** `hub-display-switch/HubDisplaySwitch/` (newest pin, uses the page's own localization hook)
- **Why:** Cross-promotion only works if every app does it, and the page hides the app it is running in, so there is no reason for an app to opt out.

Any WinUI app with a settings window includes the apps-promo page, from its first release. It is not copied into the app and not reimplemented. A web-view host for a client-branded product (store-sign-controller's desktop host) is not part of the family and does not include it.

### PROMO-2: Submodule path and canonical URL

- **Status:** Existing (6 of 6 for the path and import; 3 of 6 for the URL, see FIX-046)
- **Applies to:** Windows
- **Reference:** `wsl-monitor/.gitmodules`, `wsl-monitor/WslMonitor/WslMonitor.csproj`
- **Why:** One path and one URL mean the same commands, CI steps and ignore rules work in every repo.

| Item | Value |
| --- | --- |
| Submodule path | `external/promo`, beside the app project. In a monorepo it sits inside the host app's folder (`apps/desktop/external/promo` in persistent) |
| Submodule URL | `https://github.com/Dynamic-Solutions-Canada/apps-promo.git` |
| Project wiring | One `<Import Project="..\external\promo\PromotedApps.projitems" Label="Shared" />` line in the app's `.csproj`, and nothing else |
| CI | Every checkout that builds the app fetches submodules |

The pre-rename URL (`technicallyreal-promo` under the personal account) is not used anywhere new. The delivery mechanism for shared code in general is KIT-5 in [shared-kits.md](shared-kits.md).

### PROMO-3: Pin policy

- **Status:** Decided (D-005). Existing (0 of 6 pin a tag; four different commits are pinned across the six consumers)
- **Applies to:** Windows, Kit
- **Reference:** none yet; devkit's tag pins are the model (see KIT-6)
- **Why:** The catalog reaches shipped apps at runtime without a bump, so pins only matter for page code. Left alone they scatter, and a fix such as the localization hook then exists in one app and is re-implemented in three.

Pin policy for every kit is KIT-6 in [shared-kits.md](shared-kits.md) (tagged releases, every consumer on the same tag, the wording of the bump commit) and is not restated here. Two points are specific to apps-promo: a consumer bumps its pin when the page's code changes (layout, localization, fetch logic), in its next release at the latest, and a catalog-only change needs no bump, because the catalog reaches shipped apps at runtime.

### PROMO-4: Placement in navigation

- **Status:** Existing (6 of 6 for position, label and glyph; 4 of 6 for the tag)
- **Applies to:** Windows
- **Reference:** `repilot/Repilot/SettingsWindow.xaml`
- **Why:** The page should be in the same place in every app, out of the way of the app's own pages.

The page is a footer item of the settings window's `NavigationView`, directly above "About" (UI-2 in [ui.md](ui.md)): label "Our other apps", glyph E719, tag `PromotedAppsPage`, navigating to the page type the submodule provides. It is not a top-level menu item, not a tray menu item, and never opens by itself.

### PROMO-5: Every shipped app has a catalog entry

- **Status:** Decided (D-075, D-014). Existing (5 of the 11 end-user apps are in the catalog: printstream, persistent, little-launcher, immich-drive, repilot)
- **Applies to:** Windows, Web, Android (as catalog entries), Kit
- **Reference:** `ids.json` and `Assets/PromotedApps/apps.json` in apps-promo
- **Why:** An app missing from the catalog is not promoted by any of its siblings, and nobody notices because the page still looks full.

- Every app with a public listing (Microsoft Store, Google Play, or a public product site) has an entry, added when the listing goes live. hub-display-switch and wsl-monitor join when their Store names are reserved (D-014); phonelock and sonarbridge join when they have listings (D-075). Internal or client-specific apps are not listed.
- An entry has a stable `appId` that never changes when a product is renamed, one link per platform the product ships on, and appears exactly once in `appOrder`.
- The display name is the product's canonical name (D-012, see [branding.md](branding.md)); the catalog still shows "ImmichDrive".
- Icons are 256 px, supplied through `iconUrl`, not the Store's 50 px logo (D-070, see [assets.md](assets.md)).
- Adding or updating the entry and running `scripts/refresh-catalog.ps1` is a step of every release (D-063, see [releases.md](releases.md)). No release procedure includes it today.

### PROMO-6: The manifest URL is a permanent identifier

- **Status:** Proposed (the fetch-with-fallback behaviour is Existing in 6 of 6; the URL still uses the pre-rename owner and repo name, FIX-017)
- **Applies to:** Kit, Windows
- **Reference:** `Views/PromotedAppsPage.xaml.cs` in apps-promo
- **Why:** Installed copies fetch the catalog from a fixed public URL. If that URL stops resolving, every shipped app's page silently freezes at its last cached catalog.

- The public manifest URL is recorded in the identifier registry (BRAND-10 in [branding.md](branding.md)) and is checked before any rename or transfer of the apps-promo repo. When the URL changes, the old one must keep serving the catalog for installed copies.
- At runtime the page reads only the committed public manifest and its icons, with the last successful copy cached and the bundled copy as the offline fallback. Nothing but `scripts/refresh-catalog.ps1` talks to a store backend.
- The fetch is a network call made by every app that shows the page, so each product's privacy policy names it (DOCS-8 in [docs.md](docs.md)).

### PROMO-7: Localization goes through the page's own hook

- **Status:** Decided (D-101). Existing (1 of 6 uses the hook: hub-display-switch; 3 of 6 carry a copied adapter; 2 of 6 show the page in English only)
- **Applies to:** Windows
- **Reference:** `hub-display-switch/HubDisplaySwitch/MainWindow.xaml.cs` (sets `PromoLocalization.LocaleProvider`)
- **Why:** Three apps patch the page's text from outside by walking its visual tree. The adapters are near copies and break whenever the page changes.

A localized app sets the promo page's `LocaleProvider` once at startup, to the same language source the rest of its interface uses, and calls its refresh when the language changes. No app carries a `PromotedAppsPage.Localization.cs` adapter; the three that exist are deleted when the pin is bumped (PROMO-3). An English-only app pins the provider to `en-US`, because the page otherwise follows Windows' languages and would be the only translated page in the app. The locale set is in [locales.md](locales.md), and the app descriptions in the catalog, which stay English for now, are covered by LOC-13 there.

### PROMO-8: The page matches its host

- **Status:** Decided (D-019, D-075). Existing (1 of 6: the page uses the oldest page styling, which matches only little-launcher)
- **Applies to:** Kit, Windows, Web
- **Reference:** UI-3 in [ui.md](ui.md)
- **Why:** A shared page that looks different from the pages around it reads as an advert pasted into the app.

The WinUI page uses the family page standard (UI-3): the same padding, max width, card style and section headers as the host's own pages. It is the first page restyled under D-019. Under D-075, a web product may show the catalog and none is required to. A web app that does builds its page from its own components and follows the "Web and Android consumers" section of the apps-promo README: bundled manifest for the first offline visit, refresh when the page opens, hide the current product by `appId`, show a platform label on every app. The web page takes the catalog from the one submodule already in the repo; it does not add a second copy.

### PROMO-9: No promo surface on Android

- **Status:** Decided (D-075). Existing (2 of 2 native Android apps have none)
- **Applies to:** Android
- **Reference:** none
- **Why:** phonelock is a parental-control tool and sonarbridge is a single-purpose utility; a sibling-apps page fits neither and would need new UI in two toolkits.

The native Android apps do not show sibling apps. They still appear in the catalog once listed (PROMO-5), so the Windows apps and the website can show them. A Capacitor shell shows whatever its web app shows.
