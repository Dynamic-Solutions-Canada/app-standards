# Assets

Icons and imagery: where the master of each mark lives, which sizes each platform needs, how they are generated, and how the website and promo catalog get them. Which mark, colours and name an app uses is in [branding.md](branding.md); how a tray icon behaves is in [tray.md](tray.md); listing text and screenshot naming are in [store-listings.md](store-listings.md).

Counts are out of the 11 end-user apps in scope unless a rule says otherwise. "Windows apps" means the six with a WinUI package (displayswitchr, tray-control-for-wsl, little-launcher, repilot, immich-drive, and Persistent's desktop app).

## Proposed layout

```
assets/
  brand/
    icon.svg                   master: square, full-bleed art on a transparent canvas
    icon-maskable.svg          the same art inside the 80 percent safe zone on a solid background
    icon-mono.svg              single-colour silhouette (Android themed icon, notification, tray)
    tray/<state>.svg           only when the app ships tray state icons as files (for example offline)
    logo-horizontal.svg        only if the app has a wordmark
  generated/
    icon-256.png               cross-surface exports: README header, website, promo catalog
    icon-512.png
scripts/generate-icons.*       the one generator entry point
store-assets/                  listing text, store graphics and screenshots (see store-listings.md)
```

Platform outputs are written where the platform requires them, not under `assets/`:

| Platform | Generated into |
| --- | --- |
| Windows | `<App>/Resources/<App>.ico`, `<App>/Resources/<App>.png`, `<App>MSIX/Images/` |
| Android | `res/drawable/`, `res/mipmap-anydpi-v26/` (and `res/mipmap-*` only when legacy rasters are needed) |
| Web and PWA | `apps/web/public/` |
| Play and Microsoft Store graphics | `store-assets/<store>/` |
| HA | `custom_components/<domain>/brand/` |

## Rules

### ASSET-1: One asset folder layout

- **Status:** Decided (D-102). Proposed (0 of 11 have this layout; masters sit in `Resources/`, `apps/web/public/`, `apps/mobile/assets/` or only inside a drawing script)
- **Applies to:** Windows, Web, Android, HA
- **Reference:** `game-is-up/scripts/dev/generate-brand-icons.mjs` with `game-is-up/apps/web/public/` (closest: one script, every output)
- **Why:** When the master has a fixed address, the README, the website, the promo catalog and every build target can all be pointed at one file.

This rule owns the asset folder layout; README-9 in [readme.md](readme.md) and the trees in [repo-layout.md](repo-layout.md) follow it. Use the layout above. `assets/brand/` holds hand-authored masters only. `assets/generated/` holds the small cross-surface exports. `store-assets/` holds everything made for a listing. The README takes its icon from `assets/generated/icon-256.png` and its screenshots from `store-assets/web/screenshots/` (see [readme.md](readme.md), README-2 and README-9), by relative path. `docs/` is for documents, not images; README screenshots kept in `docs/*.png` today (little-launcher, repilot, immich-drive, sonarbridge) move. In a monorepo both folders are at the repo root.

### ASSET-2: One vector master per mark

- **Status:** Decided (D-069). Existing (1 of 11 generates everything from one source: game-is-up; printstream derives its Android icons from the web SVG)
- **Applies to:** Windows, Web, Android
- **Reference:** `game-is-up/scripts/dev/generate-brand-icons.mjs`; `printstream/apps/mobile/scripts/android-branding.mjs`
- **Why:** A mark drawn separately for each platform drifts. Persistent keeps one mark in three sources and asks, in a comment, that they be kept in step by hand.

- The master is an SVG, committed, with text converted to outlines so every rasterizer gives the same result.
- A mark has at most three master variants: full (`icon.svg`), maskable (`icon-maskable.svg`) and monochrome (`icon-mono.svg`). Colour variants an app offers its users (little-launcher's six) are parameters of the generator or extra masters in `assets/brand/`, each stored once in the repo.
- A mark that exists only as drawing code in a script (the remaining `generate-msix-images.ps1` copies) or only as rasters (little-launcher) is redrawn as an SVG. This is the main cost of D-069 and is done one app at a time.
- No app ships a platform's default or system icon as its own (phonelock's launcher icons today).

### ASSET-3: One generator, sizing and packing only

- **Status:** Decided (D-069). Proposed (3 on the shared generator, devkit-icons: chatmixr and displayswitchr since 2026-11-22, tray-control-for-wsl since 2026-11-23; little-launcher, repilot, immich-drive and persistent are moving, 2026-11-23)
- **Applies to:** Windows, Web, Android, Kit
- **Reference:** `game-is-up/scripts/dev/generate-brand-icons.mjs` (single entry point, resvg); `displayswitchr/assets/brand/icons.json` (a Windows target through devkit-icons, which writes the complete Windows size list)
- **Why:** The size lists are identical for every app; only the art differs. Five copies of the list have already produced three different output sets.

- One shared generator takes the masters and emits every size in the tables below. It contains no drawing code and no app-specific sizes. D-069 places it in devkit; see [shared-kits.md](shared-kits.md) and link devkit's README for usage once it exists.
- Each repo exposes it as one package script and documents nothing more than that; the script name follows [dev-scripts.md](dev-scripts.md) (D-043).
- Rasterize with resvg or rsvg-convert, the two already in use. ImageMagick's SVG renderer is not used.
- The generator has a check mode that regenerates to a temporary folder and fails if any committed output differs from what the masters produce. It runs in `validate` ([ci.md](ci.md)).
- Until the shared generator exists, an app does not fork another copy of a per-app script; it extends the size list of the one it has to match ASSET-5.

### ASSET-4: Generated files are committed where the platform reads them, and never edited

- **Status:** Decided (D-069). Existing (5 of 6 Windows apps commit their MSIX images; persistent ignores them and regenerates at build)
- **Applies to:** Windows, Web, Android, HA
- **Reference:** `tray-control-for-wsl/WslMonitorMSIX/Images/`
- **Why:** Committed outputs keep builds independent of the generator and its rasterizer, and make an icon change visible in review. The check mode in ASSET-3 removes the usual risk of committed outputs going stale.

- Outputs a build or a platform reads from the tree (`.ico`, MSIX images, Android resources, web icons, HA brand images, store graphics, `assets/generated/`) are committed.
- A generated file is never edited by hand and never has a second copy elsewhere in the repo. If two targets need the same file, the generator writes both, or one target references the other's path. Counter-examples: little-launcher's six 1024 px PNGs tracked twice, sonarbridge's store images tracked two and three times.
- Intermediate files (raw rasters, temporary renders) are ignored; see [repo-hygiene.md](repo-hygiene.md).
- Build outputs in general follow D-042 in [repo-layout.md](repo-layout.md); icons are source-like inputs to the build, which is why they are the exception.

### ASSET-5: Windows size set

- **Status:** Existing (2 of 6 committed `.ico` files carry the full frame list, and 2 of 5 tracked MSIX sets are complete: displayswitchr, tray-control-for-wsl)
- **Applies to:** Windows
- **Reference:** `tray-control-for-wsl/WslMonitorMSIX/Images/`, `tray-control-for-wsl/WslMonitor/Resources/WslMonitor.ico`
- **Why:** Windows picks the nearest frame and scales it. A missing frame shows as a blurred icon at 125 and 150 percent, in the taskbar, or in the Store.

| Output | File | Sizes |
| --- | --- | --- |
| App icon | `<App>.ico` | frames at 16, 20, 24, 30, 32, 36, 40, 48, 60, 64, 72, 80, 96, 128, 256 px |
| In-app image (About page) | `<App>.png` | 256 px |
| MSIX logos | `Square44x44Logo`, `Square150x150Logo`, `StoreLogo`, `SplashScreen` | each at scale 100 (unqualified file name), 125, 150, 200, 400 |
| MSIX taskbar and shell icons | `Square44x44Logo.targetsize-N`, in three forms: plain, `_altform-unplated`, `_altform-lightunplated` | N = 16, 20, 24, 30, 32, 36, 40, 48, 60, 64, 72, 80, 96, 256 |
| MSIX tiles, only if the app declares them | `Wide310x150Logo`, `Square71x71Logo`, `Square310x310Logo` | scales 100 to 400 |
| Tray state icons, only if shipped as files | `<App>-<State>.ico` | the same frame list as the app icon |

- File names are the manifest's logo names (`Square44x44Logo.scale-200.png`), not the app's name; little-launcher's `LittleLauncher.scale-N.png` scheme is the outlier.
- The `lightunplated` form is drawn for a light taskbar, not a copy of the dark one.
- A desktop app that is not MSIX (printstream's Electron host) still ships the full `.ico` frame list; its current file has three frames.
- Whether the tray icon is drawn in code or shipped as `.ico` files is a [tray.md](tray.md) question (D-071 allows either). This table applies only when files are shipped.

### ASSET-6: Android size set

- **Status:** Decided (D-069). Existing (2 of 5 repos with an Android app have a layered vector adaptive icon: printstream, sonarbridge; 1 of 5 generates it from the master: printstream)
- **Applies to:** Android (native apps and the Android hosts of web products)
- **Reference:** `printstream/apps/mobile/scripts/android-branding.mjs`; `sonarbridge/android/app/src/main/res/mipmap-anydpi-v26/`
- **Why:** The adaptive icon is what every current launcher masks, animates and themes. A raster-only or system icon looks wrong on all of them.

| Output | File | Sizes |
| --- | --- | --- |
| Launcher icon | `res/mipmap-anydpi-v26/ic_launcher.xml` and `ic_launcher_round.xml`, with `res/drawable/ic_launcher_foreground.xml`, `ic_launcher_background.xml` and `ic_launcher_monochrome.xml` | vector; 108 dp canvas with the art inside the central 66 dp |
| Legacy launcher rasters, only if `minSdk` is below 26 | `res/mipmap-{mdpi,hdpi,xhdpi,xxhdpi,xxxhdpi}/ic_launcher.png` and `ic_launcher_round.png` | 48, 72, 96, 144, 192 px |
| Notification and foreground-service icon | `res/drawable/ic_stat.xml` | vector, 24 dp, single colour on transparent |
| Play icon | `store-assets/play/graphics/icon.png` (devkit 0.21.0 default; kept out of the screenshots folder) | 512 x 512, 32-bit PNG, no transparency |
| Play feature graphic | `store-assets/play/feature-graphic.png` | 1024 x 500, PNG or JPEG, no transparency |

The monochrome layer is required (it is what themed icons use) and comes from `icon-mono.svg`. Emoji and system drawables are not used as app or notification icons. In-app iconography is a [ui.md](ui.md) topic.

### ASSET-7: Web and PWA size set

- **Status:** Existing (2 of 4 web apps: printstream, game-is-up; the other two are FIX-027)
- **Applies to:** Web
- **Reference:** `printstream/apps/web/vite.config.ts`, `printstream/apps/web/public/`
- **Why:** One SVG declared as both `any` and `maskable` has no safe-zone padding when masked and gives installers that need a raster nothing to use.

| Output | File | Sizes |
| --- | --- | --- |
| Favicon and scalable icon | `icon.svg` | vector |
| Manifest icons, `purpose: any` | `icon-192.png`, `icon-512.png` | 192, 512 px |
| Manifest icons, `purpose: maskable` | `maskable-icon-192.png`, `maskable-icon-512.png` | 192, 512 px, art inside the central 80 percent |
| iOS home screen | `apple-touch-icon.png` | 180 px, opaque background |
| Social card | `store-assets/web/social-card.jpg`, served by the web app | 1200 x 630 |

`any` and `maskable` are always separate files and separate manifest entries. Icon URLs in the manifest carry a version query string that is bumped when the art changes, so installed copies refresh. The social card has one source; a second, different card on the website (printstream today) is drift under ASSET-12.

### ASSET-8: Naming

- **Status:** Proposed (the 512 px Play icon has three names and the feature graphic three across 3 repos; the app icon is `<App>.ico` in 5 of 6 repos that commit one)
- **Applies to:** all
- **Reference:** the tables in ASSET-5 to ASSET-7
- **Why:** The same asset under three names cannot be found by a script, a sync step or a person moving between repos.

- Use exactly the file names in the size tables. Where a platform fixes a name (MSIX logo names, Android resource names, fastlane's `featureGraphic.png`), the platform's name applies inside that platform's tree only.
- Everything else is lower-case kebab-case with the size as a suffix when several sizes exist (`icon-192.png`). The Windows app icon keeps the project name in PascalCase (`<App>.ico`), matching the project it sits in.
- The extension matches the content. PNG for interface screenshots and icons, JPEG for photographs and the social card, SVG for masters. FIX-019 tracks the one known mismatch.
- Screenshot names are defined in [store-listings.md](store-listings.md), LIST-8.

### ASSET-9: Screenshot sizes and checks

- **Status:** Existing (2 of 9 apps with a listing validate listing images before upload: persistent, printstream)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent` `shots:check` (`scripts/dev/store-assets.ts`), `printstream/apps/desktop/scripts/windows-store-listing.mjs`
- **Why:** Stores reject or quietly demote images that miss their size rules, and the failure arrives after the release is tagged.

| Use | Location | Size and format | Count |
| --- | --- | --- | --- |
| Microsoft Store, desktop | `store-assets/microsoft/screenshots/` | PNG, 1366 x 768 or larger; one consistent size per listing | 1 to 10; aim for 4 or more |
| Play, phone | `store-assets/play/screenshots/phone/` | PNG, 1080 x 1920 or larger, 9:16 or 16:9 | 2 to 8 |
| Play, tablet | `store-assets/play/screenshots/tablet/` | PNG, 1200 x 1920 or larger | only if the app supports tablets |
| README and website | `store-assets/web/screenshots/` | PNG; the same files as a store set where one exists, not a recapture | up to three in the README header |

The check (part of the listing validation in LIST-9) confirms dimensions, count, no alpha channel where the store forbids it, no duplicate files, that the extension matches the content, and that every file is named in `store-assets/manifest.json`. Play accepts smaller images than 1080 px but does not promote listings that use them; printstream's 390 x 844 set is below that line.

### ASSET-10: README images

- **Status:** Decided (D-102). Existing (1 of 11 embeds listing graphics directly: persistent; 4 of 11 keep a separate set in `docs/`)
- **Applies to:** all
- **Reference:** `persistent/README.md`
- **Why:** A README set kept apart from the listing set is a third collection to recapture on every UI change, and it is the one that goes stale.

The README header icon is `assets/generated/icon-256.png`, shown at one size. README screenshots are files from `store-assets/web/screenshots/`, by relative path. The README does not hot-link an image from a hosted site, does not point at a platform output such as an MSIX logo, and does not carry images that exist nowhere else in the layout. Kits and tools need no images.

When the only screenshots a product has are a store's own set (an Android app with only Play images), the README points at that store's folder (`store-assets/play/screenshots/phone/...`) rather than copying them into `store-assets/web/screenshots/`, since ASSET-4 allows one copy of each image. The same holds for the icon until an SVG master and the shared generator produce `assets/generated/` (reference: sonarbridge).

### ASSET-11: Home Assistant brand images

- **Status:** Existing (1 of 1). The sizes are Decided (D-103)
- **Applies to:** HA
- **Reference:** `printstream/integrations/home-assistant/custom_components/printstream/brand/`
- **Why:** HACS and Home Assistant read the brand images from a fixed path and reject sizes outside their rules.

`brand/icon.png` and `brand/logo.png` are generated from the parent product's masters into the integration folder in printstream, and reach the public repo through the export. Under D-103, sizes follow the Home Assistant brands rules in force at release time. Today's files are 192 and 512 px and have not been checked against those rules; that check is made when the generator is written. The integration has no masters of its own.

### ASSET-12: The website and promo catalog take images from the app repo

- **Status:** Decided (D-070). Existing (1 promo entry of 5 is pulled from its app repo: printstream through `iconUrl`; the website hand-copies for 5 of 5 products with a page)
- **Applies to:** all, Kit (apps-promo)
- **Reference:** `apps-promo/ids.json` (`externalApps[].iconUrl`)
- **Why:** Hand copies are in step today only by care. A pull from a named path makes drift a failed check instead of something a visitor notices.

What an app repo owes the other surfaces is a stable set of paths:

| Consumer | Takes | From |
| --- | --- | --- |
| Website page | app icon | `assets/generated/icon-512.png` |
| Website page | screenshots, social card | `store-assets/web/` |
| Promo catalog | app icon, 256 px | `assets/generated/icon-256.png` |
| Home Assistant integration | brand images | generated (ASSET-11) |

- The website is out of scope for these standards, so the rule is on the app side: publish at these paths, do not rename them, and do not hand a resized or re-encoded copy to the site. D-070 calls for a sync script in the website repo that pulls these files at a pinned commit.
- The promo catalog uses a 256 px icon from the app repo for every entry, not the 50 px logo the Store returns. How the catalog is refreshed is documented in the apps-promo README; see [promo.md](promo.md).
- Refreshing both is a post-release step, owned by the shared post-release skill (D-063; REL-9 in [releases.md](releases.md)).
- The product name used for the files' slug on each surface follows D-012 ([branding.md](branding.md)).
