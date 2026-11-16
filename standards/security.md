# Security baseline

The controls every app must have, per stack: where credentials may live, what must never reach a repo, how containers and web APIs are hardened, how CI and signing keys are handled, how dependencies stay current, and what each product owes users for backup and deletion.

This file states the required control and nothing more. Known gaps are tracked by ID and are cited here, not described: the security fixes (FIX-001 to FIX-010) are kept privately and are not published with this repo (D-004), and the rest are in [FIXES.md](../FIXES.md). Do not add hostnames, key locations, or reproduction detail to this file.

The user-level agent rules (kept outside this repo) already cover how a credential is created, stored under a private `.secrets` directory, indexed, and backed up, and how new dependency versions are chosen. Those rules are not restated; this file covers what the repos and shipped apps must do.

Counts are against the in-scope repos in [SCOPE.md](../SCOPE.md). "Web products" means the four monorepos (persistent, printstream, game-is-up, store-sign-controller).

## Baseline at a glance

| Stack | Rules that apply |
| --- | --- |
| Windows | SEC-1, SEC-4, SEC-10, SEC-11, SEC-12, SEC-13, SEC-15 |
| Web | SEC-3 to SEC-10, SEC-12 to SEC-15 |
| Android | SEC-2, SEC-4, SEC-10 to SEC-13, SEC-15 |
| HA and Kit | SEC-4, SEC-10, SEC-12, SEC-15 |

## Rules

### SEC-1: Credentials at rest on Windows

- **Status:** Decided (D-081). Existing (1 of 2 Windows apps that hold a credential; see FIX-001 for the other)
- **Applies to:** Windows
- **Reference:** `little-launcher/LittleLauncher/Services/ProtectedStore.cs`
- **Why:** The settings file is plain text that users copy, export, sync, and attach to support requests.

No credential, token, or API key is written to `settings.json` or to a log. A Windows app that must keep one stores it with Windows data protection scoped to the current user, in a separate file in the app's data folder, and excludes it from any settings export. A Windows host for a web product keeps only the server origin and leaves the session to the WebView2 profile. When a second app needs the protected store, it moves into winui-app-kit (see [shared-kits.md](shared-kits.md)).

### SEC-2: Credentials and app data on Android and other native hosts

- **Status:** Decided (D-081). Existing (1 of 2 native Android apps declares its backup setting: phonelock; 3 of 3 Capacitor shells do). Encrypted token storage on Android is a proposed norm; the audit did not check how the Android apps store tokens
- **Applies to:** Android, Web (Capacitor, Electron and Linux hosts)
- **Reference:** `phonelock` manifests (explicit backup setting); `printstream/apps/desktop/electron/src/state-store.ts` (owner-only state files)
- **Why:** A token copied into a cloud backup or readable by another local account has left the app's control without anyone deciding it should.

- **Android:** tokens and keys are stored encrypted with a key held by the platform keystore, never in plain preferences. The manifest always states its backup setting explicitly; the default is off, and turning it on needs a written reason and backup rules that exclude credentials. Cleartext traffic is off unless the product has a documented need (for example a self-hosted server on a local network), and then it is stated in the privacy policy.
- **Electron and Linux hosts:** state and log files are readable only by the owning user, and credentials use the operating system's credential store where one exists.
- **Capacitor shells:** follow the Android rules; the session stays in the web view.

### SEC-3: Credentials and secrets in web products

- **Status:** Decided (D-081). Existing (1 of 4 web products encrypts the third-party secrets it stores: printstream; whether the other three store any was not checked). The browser-storage and hashing points are proposed norms; the audit did not check them
- **Applies to:** Web
- **Reference:** `printstream/apps/api/src/lib/secret-encryption.ts`
- **Why:** A database dump or a script running in the page should not be enough to act as a user or as the product.

- The browser holds a session, not a long-lived secret. Nothing that grants access is kept in browser storage that page scripts can read.
- Third-party credentials a user gives the product (API keys, access tokens for other services) are encrypted before they are written to the database, with the key supplied through the environment.
- Passwords and one-time codes are stored only as hashes.
- Passkey support uses one major version of the same library across products, moved together under SEC-12.

### SEC-4: Nothing secret or personal in a repo

- **Status:** Decided (D-038). Existing (3 of 4 repos with a project secrets folder use `.secrets/`: persistent, printstream, store-sign-controller)
- **Applies to:** all
- **Reference:** `persistent/.gitignore` (commented ignore rules with backstop patterns)
- **Why:** A pushed secret must be treated as leaked for good, and several of these repos are public.

- Project-local secrets live only in `.secrets/` (D-038) or in the user-level location named by the global credential rule. One folder name, in every repo.
- Every repo carries the shared ignore block that HYG-6 in [repo-hygiene.md](repo-hygiene.md) defines, which covers env files, the secrets folder, cloud credential folders, certificates, keystores, and service-account key patterns. A new place a secret can sit gets an ignore rule before the file exists (FIX-002, FIX-003).
- Tracked configuration refers to secrets by reference (an environment variable, a CI secret, the tool's own secret file), never by value. This includes device and firmware configuration (FIX-004).
- No personal detail is tracked: no personal email address, home network name, or home directory path in code, docs, skills, or listings (FIX-005, FIX-011). Use the company contact address chosen under D-017.
- History is checked before a repo is first pushed or made public. If any commit contains a written secret or personal detail, it is removed from history before publishing: a fresh root commit when the history is short and unpublished, or a history scrub otherwise (as for game-is-up and store-sign-controller). A secret that was ever pushed to a public remote is rotated, not just removed.
- Agents never open secret files to verify these rules; check ignore rules with `git check-ignore`.

### SEC-5: Example and template files use placeholders

- **Status:** Existing (2 of 4 web products meet it; the known gaps are FIX-006 and FIX-007)
- **Applies to:** Web
- **Reference:** `persistent/compose.server.yml` (required variables); `printstream/.env.server.example` (every variable commented)
- **Why:** Example files are copied to production unchanged more often than they are edited.

A tracked compose, nginx, or env example contains no real hostname, address, account, or password. Server names use an obviously fake placeholder domain. Every secret is a required variable that stops startup with a clear message when it is missing; an example never supplies a working default for a password or key (FIX-006, FIX-007). File naming for examples is in [settings-config.md](settings-config.md).

### SEC-6: Environment is validated, and production refuses unsafe settings

- **Status:** Existing (4 of 4 web APIs validate the environment through one module; 1 of 4 adds production guards: store-sign-controller)
- **Applies to:** Web
- **Reference:** `store-sign-controller/apps/api/src/lib/env.ts` (production guards); `persistent/apps/api/src/lib/env.ts` (helpers)
- **Why:** A check at startup is the only one that cannot be forgotten at deploy time.

The API reads its environment only through one schema-validated module. When the mode is production, the schema itself rejects insecure combinations: a client origin that is not HTTPS, any development or simulator switch, and any demo or reviewer access that SEC-14 does not allow. The process refuses to start; it does not log a warning and continue.

### SEC-7: Containers run as a non-root user

- **Status:** Existing (3 of 4 production API images; see FIX-008)
- **Applies to:** Web
- **Reference:** `store-sign-controller/Dockerfile` (the smallest correct multi-stage image)
- **Why:** A flaw in the app should not hand over root inside the container, or write access to its mounted volumes as root.

The runtime stage of every production image switches to an unprivileged user before the start command. Production compose files publish no host port (the reverse proxy reaches the app over a shared network), define a health check, and keep data in named volumes. Development images are exempt.

### SEC-8: Web API hardening: security headers and rate limiting

- **Status:** Decided (D-081). Existing (security headers middleware in 4 of 4 web APIs; rate limiting through a standard middleware library in 1 of 4: store-sign-controller; route coverage in the other three was not audited)
- **Applies to:** Web
- **Reference:** `store-sign-controller/apps/api` (rate limiting); any of the four for `helmet`
- **Why:** Sign-in and code-entry routes are guessable at machine speed unless something slows the caller down.

Every API applies the standard security-headers middleware before its routes. Every API rate-limits, at minimum, routes that accept a credential or code (sign-in, registration, password or code verification, token exchange, reviewer login) and routes that send email or notifications. Limits are counted per client with the proxy configured so the real client address is used. A rejected request gets the product's normal error shape (PLAT-15 in [platform.md](platform.md)).

### SEC-9: Origin policy

- **Status:** Decided (D-081, D-078, D-057). Existing (an allowlist exists in each of the 4 native hosts that load a web app, written four separate times)
- **Applies to:** Web, Windows (hosts of web products), Android (Capacitor shells)
- **Reference:** `printstream/apps/desktop/electron/src/origin-policy.ts`
- **Why:** A native shell that will load any page, or an API that answers any site, turns one bad link into access to the user's session.

- **API:** cross-origin requests are accepted only from an explicit list read from the environment (the web client origin and the Android app origins, using the shared variable names in [settings-config.md](settings-config.md)). No wildcard origin when credentials are allowed.
- **Native hosts:** a host loads and navigates only to its configured origin. A user-entered server address is normalised and validated before it is saved, other origins open in the system browser, and the bridge between host and page is exposed only to the allowed origin.
- The four implementations converge on one rule set per host technology as the host bridge contract is written (D-057); until then a change to one is checked against the other three.

### SEC-10: CI tokens, action pinning and secrets

- **Status:** Decided (D-046). Existing (42 of 45 workflow files set `permissions` somewhere, 32 of them at the top level as CI-5 requires; see FIX-010 for the three with none and FIX-009 for floating action references)
- **Applies to:** all repos with workflows
- **Reference:** `printstream/.github/workflows/android-ci.yml`
- **Why:** Release workflows hold signing and store credentials, so the workflow token and every third-party action are part of the trust boundary.

The workflow rules are owned by [ci.md](ci.md) and are not restated here:

| Control | Owning rule |
| --- | --- |
| Token permissions, starting from read-only | CI-5 |
| Action pinning, and the Dependabot entry for actions | CI-7 (D-046) |
| Code scanning | CI-8 |
| Shared store secret names and the credential-expiry reminder | CI-11 |

This rule adds the one requirement that is about the secrets themselves: a secret reaches a job only through repository or environment secrets, and is never echoed, written to a log or an artifact, or made available to a pull request from a fork.

### SEC-11: Signing key custody

- **Status:** Proposed as a whole; Decided (D-021) for phonelock. Existing in part (no certificate or production keystore is tracked in any in-scope repo; which key signs debug builds was not verified for every app)
- **Applies to:** Windows, Android
- **Reference:** `sonarphone-nmea-bridge/android/app/build.gradle.kts` and its release workflow
- **Why:** A lost signing key ends an app's update path, and a leaked one lets someone else ship as the publisher. Neither can be patched afterwards.

- A production signing key exists in exactly two kinds of place: the CI secret store, and the maintainer's private credential storage with its backup, as the credential rule in the user-level agent rules describes. Never in a repo, a build script, a container image, or an artifact.
- No signing password is written in a script or config file, including fallback paths for old certificates (FIX-060). Superseded per-app certificates are removed from working trees once the shared path works.
- Production keys never sign debug builds. Debug and local development builds use a separate development key; the shared Windows development certificate is supplied by the build tooling and is git-ignored in every Windows-building repo (FIX-002).
- One application id has one signer across every channel it ships on; DIST-8 in [distribution.md](distribution.md) owns that rule (D-021 for phonelock). Where a store holds the app signing key, the upload key is the only one the maintainer keeps, and the published signing certificate fingerprint is recorded so releases can be checked against it.
- Which signer each build context uses is DIST-2 (Windows) and DIST-8 (Android) in [distribution.md](distribution.md).
- A throwaway certificate created for a single CI packaging check (DIST-2) is outside these custody rules: it is never a production or development signing key, is never stored, and signs nothing that is published.

### SEC-12: Dependency update cadence

- **Status:** Decided (D-107, D-037, D-046). Existing (1 of 15 repos has update automation for every ecosystem it uses: printstream; little-launcher covers one project)
- **Applies to:** all
- **Reference:** `printstream/.github/dependabot.yml`
- **Why:** A security fix in a runtime or library has had to be applied repo by repo, by hand, and some repos were missed.

- Every repo has update automation for each ecosystem it contains. The Dependabot setup (weekly, grouped, every directory watched) is HYG-8 in [repo-hygiene.md](repo-hygiene.md) and is not restated here.
- Security updates are not grouped or delayed (D-107). A fix in anything that ships inside an app or serves traffic is applied within a week of the advisory, and a runtime security bump is applied to every repo on that runtime in the same pass, tracked against the versions table in [platform.md](platform.md). Development-only tooling waits for the next scheduled pass (PLAT-1).
- Shared projects consumed by pin (winui-app-kit, apps-promo, devkit) are bumped in every consumer when they release a security-relevant change.
- Version choice for new dependencies follows the dependency rule in the user-level agent rules; pinning style is PLAT-10 in [platform.md](platform.md).

### SEC-13: Backup, export and deletion

- **Status:** Decided (D-025). Existing (server backup in 1 of 4 web products: printstream; account controls in persistent)
- **Applies to:** Web, Windows, Android
- **Reference:** printstream's server backups (self-hosted backup); persistent's account settings (account controls)
- **Why:** Google Play requires an account-deletion path, self-hosters expect a stated answer, and a lost database cannot be recovered.

Every store-listed product documents and implements three things (D-025), with printstream as the reference; game-is-up and store-sign-controller document the answer even where it is a restore from the host's database backup. Which document they are written in is DOCS-9 in [docs.md](docs.md).

| Topic | Required |
| --- | --- |
| Backup | How server data is backed up and restored, and who is responsible. "Restore from the host's database backup" is an acceptable answer if it is written down and has been tried once. |
| Export | How a user gets their own data out, or a statement that there is none to export. |
| Deletion | How a user deletes their account and data, reachable from inside the app and from a public URL for store listings, and what is retained afterwards. |

Client apps also document every place they write outside their own data folder and what remains after uninstall (the list is part of DOCS-9). Backups hold the same secrets as the live system and get the same protection.

### SEC-14: Demo mode and store-review access

- **Status:** Decided (D-083). Existing (4 of 4 web products have a mechanism, each different; 1 of 4 refuses its development switch in production: store-sign-controller)
- **Applies to:** Web, and any app whose store listing needs reviewer credentials
- **Reference:** `store-sign-controller/apps/api/src/lib/env.ts`
- **Why:** Reviewer logins are a recurring store requirement and a recurring way to leave a back door open.

One pattern (D-083):

- Demo data and reviewer access are switched on only by environment variables that follow the shared naming (CONF-10 in [settings-config.md](settings-config.md)), and are off by default.
- The environment schema (SEC-6) refuses demo and simulator switches in production. Reviewer access in production is a named, ordinary account with no extra privileges, limited to demo data, and covered by rate limiting (SEC-8).
- Reviewer credentials are created per submission, stored under the global credential rule, entered only in the store's private reviewer-notes field (see [store-listings.md](store-listings.md)), and rotated or disabled once the review completes.
- Nothing in a tracked file gives a working login.

### SEC-15: Reporting a vulnerability

- **Status:** Decided (D-106). Proposed (0 of 15 repos have a security policy file)
- **Applies to:** all public repos
- **Why:** Someone who finds a problem needs a private way to report it; the only channel today is a public issue.

Every public repo gives a private way to report a vulnerability. The policy file, where it sits, its template and the channel it names are owned by HYG-9 in [repo-hygiene.md](repo-hygiene.md) and are not restated here. This rule adds the handling side: a fix for a reported problem is given a FIX ID and its detail is recorded privately, in the same way as the security findings of the audit; public files cite the ID only.
