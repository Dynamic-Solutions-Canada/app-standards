# Distribution

How each app is packaged, signed and delivered to users, and how an installed copy learns about updates. Versions, tags, notes and the release procedure are in [releases.md](releases.md); listing text and images are in [store-listings.md](store-listings.md); where build output lands and how artifacts are named are in [repo-layout.md](repo-layout.md).

No signing material, host name or credential appears in this file. Where a rule touches one, it cites the fix or decision ID, and storage rules are in [security.md](security.md) and the user-level agent rules (kept outside this repo).

## Channels

| Stack | Primary channel | Secondary | How installed copies update |
| --- | --- | --- | --- |
| Windows | Microsoft Store (MSIX) | Sideload or portable build for unpackaged use, where the product offers one (DIST-5) | The Store; the About page button (DIST-6) |
| Web | Hosted deploy from a container image (DIST-12) | Published images for self-hosted products | Deploy |
| Android | Google Play | None for a Play-only app; otherwise a GitHub Release with the Play-signed APK (DIST-9), or a `github` flavour for an app not on Play | Play; an in-app updater only in non-Play flavours (DIST-10) |
| HA | HACS custom repository (DIST-11) | Manual copy | HACS |
| Kit | Git: tag-pinned npm reference or submodule (see [releases.md](releases.md) REL-11) | none | Consumers bump the pin |

## Rules

### DIST-1: Windows apps ship as MSIX built from `<App>MSIX/`

- **Status:** Existing (6 of 6 WinUI apps). Sharing one packaging script is Decided (D-005)
- **Applies to:** Windows, and Windows desktop apps inside Web monorepos
- **Reference:** `wsl-monitor/WslMonitorMSIX/build-msix.ps1` (tool lookup without an installed SDK, standard switch names, the strictest language check); `wsl-monitor/windows-build.json` for the smallest manifest
- **Why:** One packaging path for local builds and CI means the package a developer installs is built the same way as the one the Store receives.

Each app has an `<App>MSIX/` folder with `Package.appxmanifest` and a `build-msix.ps1` that reads the version from `Directory.Build.props`, publishes self-contained, stamps the manifest, builds one `resources.pri` holding every declared language, packs, and signs unless told not to. The script takes the standard switches (`-Platform`, `-Configuration`, `-NoSign`, `-TrustedPfxPath`, `-TrustedPfxPassword`) so that `windows-build.json` needs no per-app translation keys. Local builds go through `scripts/windows-build`; see `windows-build-tools/README.md` for the tool itself, which these standards do not re-document. CI calls the same `build-msix.ps1`. No app uses a packaging project. Under D-005, the six diverged script copies become one shared script delivered with winui-app-kit and driven by `windows-build.json`.

### DIST-2: Signing depends on the context, and no key lives in a repo

- **Status:** Existing (shared dev certificate through `windows-build.json` in 6 of 6; unsigned Store packages in 4 of 4 Store apps; no CI code-signing secret in any of the 3 standalone apps with a build workflow, checked with `gh secret list` on 2026-11-16: little-launcher, repilot, immich-drive; CI packaging checks signed through the script fallback that FIX-060 removes). FIX-002 covers the one missing ignore rule
- **Applies to:** Windows
- **Reference:** `immich-drive/windows-build.json` and `immich-drive/.github/workflows/build-msix.yml`
- **Why:** Three contexts need three different signers, and mixing them is how a private key ends up in a working tree or a script.

| Context | Signer |
| --- | --- |
| Local dev build | The shared dev certificate supplied by the build tool (`"signing": "shared-dev"` in `windows-build.json`). The app repo holds no certificate, and the tool reads the certificate's password from the user-level credential store, never from code |
| CI sideload or portable build | A code-signing certificate held as repository secrets, passed to the packaging script, with a timestamp |
| CI packaging check (nothing published) | A throwaway self-signed certificate created per run with a random password, masked in the log, written only to the runner's temp folder, removed from the certificate store and deleted after the run, and passed to the packaging script like any other. It is never stored or reused. Building unsigned is not a substitute, because it skips the script's signing branch (publisher stamp from the certificate subject, dev name stamp, signing tool), which dev builds depend on |
| Store build | Unsigned (`-NoSign`); the Store signs on ingestion |

Packaging scripts contain no fallback that signs with a certificate file kept beside the script, and no password (FIX-060). Key custody in general is SEC-11 in [security.md](security.md). Every repo ignores the dev certificate path and certificate file types (see [repo-hygiene.md](repo-hygiene.md)). Per-app self-signed certificates from before the shared dev certificate are deleted, not kept as a fallback.

### DIST-3: Dev builds use their own package identity

- **Status:** Decided (D-065). Existing (2 of 4 apps with a Store identity: immich-drive, persistent)
- **Applies to:** Windows
- **Reference:** `immich-drive/windows-build.json` (`dev_identity`, `store_identity`, `store_identity_switch`)
- **Why:** A dev build that reuses the Store identity cannot sit beside the Store install, and an upgrade over the Store copy cannot be tested.

`windows-build.json` names a dev identity that differs from the Store identity, plus the Store identity and the switch that selects it. Dev builds use the dev identity by default; building with the Store identity is an explicit opt-in used to test an update over a Store install. The Store identity itself is a permanent identifier and is recorded in the identifier registry (BRAND-10 in [branding.md](branding.md)).

### DIST-4: The Microsoft Store is the primary Windows channel

- **Status:** Decided (D-014). Existing (4 of 6 WinUI apps have a Store-assigned identity: little-launcher, repilot, immich-drive, persistent; hub-display-switch and wsl-monitor carry an unreserved placeholder)
- **Applies to:** Windows
- **Reference:** `immich-drive/.github/workflows/store-publish.yml`
- **Why:** The Store handles signing, updates and payment, and a package identity cannot be changed once people have installed the app.

Every Windows app is delivered through the Store. The app's name is reserved in Partner Center before its first public build, so the manifest carries the Store-assigned identity from the start and never a placeholder. Submission runs from CI as described in [releases.md](releases.md) REL-7. hub-display-switch and wsl-monitor go to the Store too, and their names are reserved now (D-014). The publisher display name in the manifest is "Dynamic Solutions (CA)" (BRAND-2 in [branding.md](branding.md), D-006).

### DIST-5: What a GitHub Release carries

- **Status:** Decided (D-020, D-041, D-042). Existing (2 of 3 paid Store apps in public repos attach no binaries: repilot, immich-drive; 13 of 15 repos commit no build output)
- **Applies to:** all
- **Reference:** the header comment of `repilot/.github/workflows/build-msix.yml` (why nothing is attached); `printstream` server packages for `SHA256SUMS`
- **Why:** A free binary on a public repo undercuts a paid listing, and an asset whose name an installed app fetches cannot be renamed casually.

- A paid Store app in a public repo attaches no binaries to its Releases; the Release points to the Store. little-launcher stops attaching its portable zips (D-020). The rule covers Windows builds, where the paid listing is: a Linux build with no paid channel (little-launcher's AppImage) may still be attached, with `SHA256SUMS`.
- A free app, or one with an unpackaged audience, may attach a signed sideload or portable build (DIST-2).
- Assets are named by the convention in [repo-layout.md](repo-layout.md) (D-041). Before renaming any asset, check whether an installed updater fetches it by name (DIST-10).
- A Release with binaries carries one `SHA256SUMS` file.
- No build output is committed to a repo (LAYOUT-13 in [repo-layout.md](repo-layout.md), D-042); firmware images and editor extensions ship as Release assets.

### DIST-6: Windows update check is a button on the About page

- **Status:** Decided (D-014). Existing (an update path in 4 of 6 WinUI apps; 3 of 6 open the Store's updates page: little-launcher, repilot, immich-drive; 1 of 6 downloads and installs in-app: persistent; none checks in the background)
- **Applies to:** Windows
- **Reference:** `repilot/Repilot/Services/UpdateService.cs` (the small service shared in shape by three apps); `persistent/apps/desktop/Persistent.Desktop/Services/UpdateService.cs` for the in-app install, whose header documents the traps
- **Why:** The Store already updates packaged apps; the app only needs to tell the user where to look, and one small shared service is cheaper to keep right than six.

- Every app has a "Check for updates" action on its About page (see [ui.md](ui.md)). No app checks in the background. Exception (decided 2026-11-17): little-launcher's retired portable (unpackaged) copies keep a startup check with a notice, because it is how those users learn that updates moved to the Store; packaged copies never check in the background.
- Packaged build: the action opens the Store's updates page. In-app download and install is the exception, used only where the product needs it (persistent).
- Unpackaged build: the action queries the repo's GitHub Releases (filtered by the lane's tag prefix in a monorepo) and links to the Release page.
- The service comes from winui-app-kit once it is moved there (D-005); apps do not keep private copies. The repo owner string it queries must be the repo's current owner (FIX-014, FIX-015, FIX-016).

### DIST-7: Android flavours are named for the channel

- **Status:** Existing (1 of 2 native Android apps uses `github` and `play` flavours on one dimension: phonelock; sonarbridge is Play-only with no flavours (D-112); persistent, the Capacitor app checked, uses a single channel flavour)
- **Applies to:** Android
- **Reference:** `phonelock` for two flavours; `sonarbridge/android/app/build.gradle.kts` for one channel with none
- **Why:** Play forbids things a direct download allows (self-update, some permissions), and a flavour is the place to keep that difference out of shared code.

An app distributed through more than one channel has one flavour per channel, named for it (`play`, `github`), on a single distribution dimension. Code and manifest entries that only one channel may have (an updater, a permission Play would reject) live in that flavour's source set, not in `main`. An app with one channel does not need flavours. Application ids are BRAND-7 in [branding.md](branding.md) (D-011).

### DIST-8: One signer per Android application id

- **Status:** Decided (D-021). Existing (1 of 2 native apps uses Play App Signing with its upload key held only in CI secrets: sonarbridge; phonelock is not on Play yet)
- **Applies to:** Android
- **Reference:** `sonarbridge/.github/workflows/play-publish.yml`; the "Get it" section of `sonarbridge/README.md` for what a signer split costs users
- **Why:** Two builds with the same application id and different signers cannot replace each other, so users must uninstall and lose settings to move between them.

Every build of an application id that users can install is signed by the same key. For a Play app that key is Google's app signing key, and the upload key lives only in CI secrets. Release signing credentials are never in the repo; a debug keystore may be committed so reinstalls work across build containers, and it signs debug builds only. An app that must ship outside Play with a different signer takes a distinct application id (a flavour suffix). For phonelock (D-021): the existing key is enrolled as Play's app signing key so both channels keep one signer; if Google refuses it, the two channels take separate application ids. Debug builds get their own key. All of this is done before the first Play upload.

### DIST-9: A Play app attaches no APK to GitHub, or only the Play-signed one

- **Status:** Existing (3 repos publish to Play: persistent attaches the Play-signed APK with a `play-apk.yml` workflow; sonarbridge (D-112) and printstream attach none). Decided (D-112) that a Play app with no need to ship outside Play attaches none
- **Applies to:** Android
- **Reference:** `persistent` for attaching; `sonarbridge/.github/workflows/play-apk.yml` and `scripts/play-apk.mjs` for the verified fetch kept as a private artifact
- **Why:** It is the direct consequence of DIST-8: the only APK that can update, or be updated by, the Play install is the one Google signed. Even that one opens the Play Store when sideloaded if Play's license check is in it, so a Play-only app is simpler with no APK at all.

A Play app that has no reason to ship outside Play attaches no APK: its GitHub Release carries the notes and a link to the Play listing. Where an app does offer a direct download, CI uploads the bundle to Play, then downloads Google's signed universal APK, verifies its certificate fingerprint, and attaches it to the GitHub Release with a `play` channel suffix in its name. Either way CI may keep the verified APK as a private workflow artifact (for screenshots or testing), never as a release asset of a Play-only app. No locally signed APK of a Play app is published anywhere. Version name and version code come from the tag (see [releases.md](releases.md) REL-1), and listing metadata and "What's new" text are pushed from the repo (see [store-listings.md](store-listings.md)).

### DIST-10: Self-updaters outside the stores

- **Status:** Existing (3 repos ship an updater outside a store: phonelock, and the Linux AppImage builds of little-launcher and persistent). The artifact-name constraint is Decided (D-041)
- **Applies to:** Android flavours not on Play; Linux desktop builds; any direct-download build
- **Reference:** `persistent/apps/linux-desktop/` (update manifest published with the Release, install only with consent); `phonelock/tools/publish-release.sh` (version file beside the assets)
- **Why:** Outside a store the app is the only thing that can tell the user about an update, and whatever it fetches becomes a permanent interface.

- A store build has no self-updater. Play forbids one, so it exists only in the non-Play flavour (DIST-7).
- A direct-download build may update itself. It reads a small version manifest published with the Release, verifies what it downloads (same signer on Android, checksum elsewhere), and installs only with the user's consent.
- Every URL and asset name an installed copy fetches is recorded in the identifier registry (BRAND-10 in [branding.md](branding.md)) and treated as permanent: a repo move or an asset rename keeps the old name working. FIX-013 to FIX-017 are the cases where this was missed.

### DIST-11: The Home Assistant integration ships through HACS

- **Status:** Decided (D-015). Existing (1 of 1: a HACS custom repository holding the generated bundle)
- **Applies to:** HA
- **Reference:** `printstream/integrations/home-assistant/hacs.json` and the exported `printstream-home-assistant` repo
- **Why:** HACS installs from a tagged public repo, so the repo's layout and tags are the distribution format.

The public repo follows the HACS layout (`custom_components/<domain>/`, `hacs.json` with the minimum Home Assistant version, the README rendered in HACS) and is installed as a HACS custom repository, with manual copy as the documented alternative. HACS takes the version from the tag, so releases follow [releases.md](releases.md) REL-10. It stays a custom repository holding the generated card bundle only (D-015); default-store listing and publishing card sources are revisited after the licence work under D-008. The `NOTICE` shipped in the public repo describes that repo, not the parent (FIX-024).

### DIST-12: Web products deploy as one container image

- **Status:** Decided (D-079). Existing (single image in 3 of 4 Web monorepos: printstream, persistent, store-sign-controller; a tracked `compose.server.yml` in 2 of 4: persistent, store-sign-controller; a scripted deploy in 3 of 4). FIX-006 and FIX-008 are the known gaps
- **Applies to:** Web; server components of other stacks (phonelock's hub)
- **Reference:** `persistent/compose.server.yml` for structure; `store-sign-controller/Dockerfile` for the smallest correct multi-stage image; `persistent/scripts/deploy/deploy-over-ssh.mjs` for the deploy script
- **Why:** The same image shape and the same deploy command across products make the server predictable and a deploy reviewable.

- One multi-stage root `Dockerfile` producing a single image that serves the API and the built web app. This rule owns the single-image shape.
- Compose file names, and which products track the production file, are CONF-11 in [settings-config.md](settings-config.md). A product that other people self-host (printstream) also publishes versioned images to a registry.
- Hardening of the image and the compose files (non-root user, health check, named volumes, required secrets with no default, placeholder host names) is SEC-5 and SEC-7 in [security.md](security.md) and is not restated here.
- Deploy is `npm run deploy:prod` (and `deploy:staging` where a staging environment exists), which updates the server checkout and rebuilds. What the script must refuse and check first is SCRIPT-10 in [dev-scripts.md](dev-scripts.md); env files and variable names are CONF-10 in [settings-config.md](settings-config.md).
- Migrations run on container start through `start:prod` (seen in persistent and store-sign-controller; not checked in the other two).
- The API serves the web app's shell (`index.html`) for any GET that matches no file, so client routes load directly, except a path whose last segment has a file extension, which gets a 404 with `Cache-Control: no-store` (Cloudflare caches 404s for static extensions otherwise). A missing image then fails as an image instead of returning HTML with a 200. App routes keep their last segment free of a dot. Reference: `game-is-up/apps/api/src/lib/web-static.ts`.

Container conventions for development belong to devkit; see `devkit/README.md`.
