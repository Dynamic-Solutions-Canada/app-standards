# Locales

Which languages the apps ship in, how that set is named, and how strings are stored, keyed, resolved and checked on each stack. Listing text is in [store-listings.md](store-listings.md); the language setting's place on the Settings page is UI-4 in [ui.md](ui.md).

Counts are out of the 11 end-user apps in scope unless a rule says otherwise. Seven of them are localized (hub-display-switch, wsl-status, little-launcher, repilot, persistent, phonelock, sonarbridge); "of 7" counts those. "Of 5" counts the localized Windows apps (the first four plus Persistent's desktop app).

## The nine-locale set

| Order | Identity | Native name shown in the app | Wording catalog |
| --- | --- | --- | --- |
| 1 | `en-US` | English (United States) | `en-US` |
| 2 | `en-CA` | English (Canada) | reads `en-US` |
| 3 | `es-ES` | Español (España) | `es-ES` |
| 4 | `fr-FR` | Français (France) | `fr-FR` |
| 5 | `fr-CA` | Français (Canada) | reads `fr-FR` |
| 6 | `de-DE` | Deutsch (Deutschland) | `de-DE` |
| 7 | `zh-CN` | 简体中文（中国） | `zh-CN` |
| 8 | `pt-BR` | Português (Brasil) | `pt-BR` |
| 9 | `ja-JP` | 日本語（日本） | `ja-JP` |

Nine identities, seven wording catalogs. This table is the single statement of the set; app docs link here instead of restating it.

## Rules

### LOC-1: The supported set is the nine-locale set

- **Status:** Existing (7 of 11)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/packages/shared/src/locales.ts`
- **Why:** One set across the family means one translation pass per string, one listing matrix, and a user who changes app keeps their language.

A localized app supports exactly the nine identities above, in that order, with those native names, wherever the list is shown or declared (language picker, package manifest, listing). The two Canadian identities are real choices a user can pick and that the OS and stores see, even though they share wording with `en-US` and `fr-FR` today. An app does not add a tenth locale or drop one on its own; a change to the set is a change to this file first, then to every localized app. Known drift: Repilot's native names for `zh-CN` and `ja-JP` differ from the table.

### LOC-2: How the set is named

- **Status:** Proposed (4 of 7 define their languages by pointing at another app, such as "the same languages as Persistent": wsl-status, little-launcher, persistent, phonelock)
- **Applies to:** all
- **Reference:** `phonelock/README.md` `## Languages` (wording of the user-facing sentence)
- **Why:** Naming the set after one app makes that app the standard by accident, and the reference goes stale when it changes.

- In standards, developer docs, code comments and agent guides, the set is called **the nine-locale set** and linked to this file. Do not call it "the Persistent locales" or define it by pointing at another app's source file.
- In code, the list is one named constant per app (`SupportedLanguages` in the kit for Windows; `supportedLocales` in a shared package for Web), never a literal list repeated per screen.
- For users (README and listing), name the languages, not the identities: "English (United States and Canada), Spanish, French (France and Canada), German, Simplified Chinese, Brazilian Portuguese and Japanese". In the README this sentence goes in the optional `## Languages` section (see [readme.md](readme.md), README-1).
- "Locale identity" means one of the nine tags; "wording catalog" means one of the seven sets of strings. Use those two terms, so "nine" and "seven" are never confused.

### LOC-3: Fully localized or English only, with named exemptions

- **Status:** Decided (D-010). Existing (11 of 11 are either all nine or English only in the interface; 2 of 7 localized apps also carry all nine in their listing)
- **Applies to:** all
- **Reference:** `persistent/docs/localization.md`
- **Why:** A partial set is worse than either extreme: the user cannot predict which screens, listings or notes will be in their language.

An app is in one of two states, and is in the same state everywhere (interface, package manifest, listing, release notes):

- **Localized:** the full nine-locale set.
- **English only:** one language, `en-US`, declared as the only language everywhere, and stated in the README.

The exemptions, decided in D-010: printstream is exempt until it has an i18n layer; game-is-up, store-sign-controller and the Home Assistant integration are exempt; immich-drive is brought into the nine-locale set because it is a paid Store app sold in the same markets as the localized ones. Kits and developer tooling (devkit's CLI, agent docs, logs) are English and are not user-facing in this sense. A new app with a store listing is localized from its first release unless an exemption is recorded in D-010.

### LOC-4: Default, fallback and resolution

- **Status:** Existing (7 of 7 default to `en-US`; the full resolution order is written down in 1 of 7: persistent)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/packages/shared/src/locales.ts` (`resolveLocale`)
- **Why:** The edge cases (a Traditional Chinese user, a French-speaking Swiss user, a key missing from one catalog) should behave the same in every app.

- **Default** is `en-US`. It is also the catalog every other catalog is checked against.
- **Choice:** the app follows the system language unless the user picks one in Settings. The picker shows the nine identities by native name in the standard order; its place on the Settings page and the wording of the follow-the-system option are UI-4 in [ui.md](ui.md) (D-072).
- **Resolution:** the user's ordered language preferences are tried in turn. For each, an exact identity match wins; otherwise the same language and script family (`fr-CH` gets `fr-FR`, `pt-PT` gets `pt-BR`). If nothing matches, `en-US`.
- **Script safety:** Traditional Chinese (`zh-Hant`, or `zh` with region TW, HK or MO) never resolves to the Simplified catalog. It falls through to the next preference, then English.
- **Per-key fallback:** a key missing from a catalog shows the `en-US` text, never the key or an empty string. The catalog check (LOC-9) exists so this never happens in a release.
- **Wording only:** the language choice changes wording. Dates, numbers and clock format follow the device region unless the app offers a separate override. User content and device names are never translated.

On Android and with `.resw`, the platform's own resource resolution provides most of this; the app must still meet the script-safety and wording-only points.

### LOC-5: Aliases are a mapping, not duplicated files

- **Status:** Decided (D-068). Existing (6 of 7; sonarbridge ships three duplicate catalogs)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/packages/shared/src/locales.ts` (`localeCatalogs`); `phonelock/companion/src/main/res/` for Android
- **Why:** A duplicated catalog is hundreds of strings to keep in step by hand for no visible difference.

`en-CA` reads the `en-US` catalog and `fr-CA` reads `fr-FR` through a mapping in code (Windows, Web) or the platform's own fallback (Android: no `values-en`, `values-en-rCA` or `values-fr-rCA` folders, as D-068 decides). The one place a full record per alias is unavoidable is a store listing; there it is expressed as `copyFrom` and expanded by the publisher ([store-listings.md](store-listings.md), LIST-3).

### LOC-6: Resource mechanism per stack

- **Status:** Decided (D-067, D-005). Existing (2 of 5 localized Windows apps use the kit classes: hub-display-switch, wsl-status; 2 of 2 native Android apps use `strings.xml`; 1 of 1 localized web product uses shared JSON catalogs)
- **Applies to:** Windows, Web, Android, HA
- **Reference:** see table
- **Why:** Five mechanisms across seven apps means five checkers and no shared tooling. One per stack is the minimum the platforms allow.

| Stack | Mechanism | Reference |
| --- | --- | --- |
| Windows | JSON catalogs embedded as resources, read through winui-app-kit's `UiText` and `SupportedLanguages`. See the kit README for usage. | `wsl-status/WslMonitor/Classes/AppLanguages.cs` |
| Windows, accepted second mechanism | `.resw` with `x:Uid`, for an app with no resident UI that does not need live language switching | `repilot/Repilot/Strings/<locale>/Resources.resw` |
| Windows manifest strings | `.resw`, since the package manifest requires it; generated from the catalogs at package time, not hand-kept | `persistent/apps/desktop/tools/localization-resources` |
| Web, and native hosts of a web product | One set of JSON catalogs in the shared package, loaded by `i18next` on web and read directly by each native host | `persistent/packages/shared/src/messages/` |
| Android (native) | `strings.xml` resources, split by area once a file grows past a screenful | `phonelock/companion/src/main/res/values/` |
| HA | Home Assistant's `strings.json` with `translations/en.json` generated from it, not hand-kept as a pair | `printstream/integrations/home-assistant/custom_components/printstream/` |

Under D-067, new Windows work and the two kit apps use the kit; little-launcher and Persistent's desktop app move from their private copies of the same design to the kit's classes when they adopt the kit; Repilot's `.resw` stays. An app does not write its own `UiText`. A product with clients on several platforms keeps one catalog set shared by all of them, as persistent and little-launcher do, not one per client.

No user-facing string is a literal in code or markup in a localized app. Symbols and formatted values are the only exceptions.

### LOC-7: Key style and placeholder syntax

- **Status:** Decided (D-067, D-099). Existing (dotted keys 2 of 5 Windows apps: wsl-status, little-launcher; `{{name}}` 3 of 5 plus the kit: hub-display-switch, wsl-status, persistent; snake_case and positional placeholders 2 of 2 native Android apps; Web 1 of 1 as shown)
- **Applies to:** Windows, Web, Android
- **Reference:** `wsl-status/WslMonitor/Strings/en-US.json`
- **Why:** A dotted key survives a wording fix; an English sentence used as the key changes in every catalog each time the English is corrected. Named placeholders let a translator reorder a sentence.

| Stack | Key style | Example | Placeholder |
| --- | --- | --- | --- |
| Windows (kit catalogs) | Dotted, area first, lower-case kebab segments | `distro.files-tip` | `{{name}}` |
| Web | Today: the English sentence as the key, with dotted codes for errors (persistent, the one localized web product). Under D-099, dotted ids for new web products and new areas; persistent's existing keys stay until a planned string pass. | `error.announcement-not-found` | `{{name}}`, `<v0/>` for rich-text slots |
| Windows (`.resw`) | `Name.Property`, as `x:Uid` requires | `NavHome.Content` | `{0}` |
| Android | snake_case with an area prefix | `main_add_device` | `%1$s` (always positional) |
| HA | Home Assistant's nested keys | per the platform | `{name}` |

- The area is the page or feature (`settings.`, `tray.`, `error.`). Error and status codes shared with a server use the same dotted form (`error.announcement-not-found`).
- A placeholder name says what it holds (`{{count}}`, `{{deviceName}}`), and the same names appear in every catalog.
- Plural or gendered sentences are separate keys or the library's plural form, never string concatenation.
- Existing English-sentence keys in hub-display-switch are not rewritten for their own sake; under D-067 they move to dotted keys at the app's next string pass. `{name}` in little-launcher changes when it adopts the kit.

Shared labels that every app shows (D-072: Settings, About and tray wording such as "Start with Windows", "Our other apps", "Open logs folder") are supplied by winui-app-kit in its own catalogs under the `kit.` key prefix, in the same wording catalogs as the apps. Text lookup checks the app's catalog, then the kit's, then `en-US`. Apps do not define `kit.` keys: a different wording for a shared label is a change to the kit, so the label reads the same in every app. The catalog check (LOC-9) treats `kit.` keys as owned by the kit.

### LOC-8: Catalog location and file names

- **Status:** Decided (D-067). Existing (1 of 5 Windows apps uses the decided path: hub-display-switch; four different paths are in use)
- **Applies to:** Windows, Web, Android
- **Reference:** `hub-display-switch/HubDisplaySwitch/Localization/Catalogs/`
- **Why:** The folder is where every agent and translator looks first; four names for it across five apps costs a search each time.

| Stack | Location | File name |
| --- | --- | --- |
| Windows (kit catalogs) | `<App>/Localization/Catalogs/` | `<identity>.json`, full identity (`fr-FR.json`) |
| Windows with a shared non-Windows client | `Shared/Localization/` at the repo root | `<identity>/<area>.json` or `<identity>.json` |
| Web | `packages/shared/src/messages/` | `<identity>.json` |
| Android | `res/values/` (English) and `res/values-<qualifier>/` | the platform's qualifiers: `values-es`, `values-fr`, `values-de`, `values-pt`, `values-ja`, `values-b+zh+Hans` |

Seven files or folders, one per wording catalog; none for the two aliases (LOC-5). Files are named by full identity even where only the language differs, so a later regional catalog needs no rename. See [repo-layout.md](repo-layout.md) for the surrounding project layout.

### LOC-9: A catalog check runs in the normal validate step

- **Status:** Decided (D-067). Existing (7 of 7 have a check, in seven implementations; 6 of 7 fail the build, and sonarbridge's `android/scripts/check-localization.py` is run by hand: neither Gradle nor a workflow calls it)
- **Applies to:** Windows, Web, Android
- **Reference:** `wsl-status/scripts/check-strings.py` (most complete single script); `phonelock/companion/build.gradle` `lint` block (Android)
- **Why:** A missing key is invisible in English testing and shows as English text, or a raw key, to exactly the users the translation was for.

The check fails the build, locally and in CI ([ci.md](ci.md), [testing.md](testing.md)), when any of these is false:

- Every wording catalog has exactly the key set of `en-US`: none missing, none extra.
- Placeholders in each translation match the `en-US` entry by name or position.
- No value is empty.
- Every key used in code or markup exists, and no catalog key is unused.
- The identities declared to the OS (LOC-10) equal the supported list, and the catalog files on disk equal the seven wording catalogs.
- Text follows the house writing rules the script can test (wsl-status's rejects em dashes).

The script is exposed as `check:l10n` (D-067; wsl-status's `check:strings` is renamed) and is part of `validate`; see [dev-scripts.md](dev-scripts.md). On Android the same guarantees come from lint configured to fail release builds on missing, extra or mismatched translations, as phonelock does, or from a script wired into the build. A shared checker for the kit catalogs belongs in winui-app-kit so the two kit apps stop carrying one each; see [shared-kits.md](shared-kits.md).

### LOC-10: The supported list is declared to the OS

- **Status:** Existing (6 of 7; little-launcher declares one language, FIX-020)
- **Applies to:** Windows, Android
- **Reference:** `wsl-status/WslMonitorMSIX/Package.appxmanifest`, `phonelock/companion/src/main/res/xml/locales_config.xml`
- **Why:** Windows, Android and both stores decide what to offer a user from the declaration, not from the files the app happens to ship.

- **MSIX:** one `<Resource Language>` entry per identity, nine in the standard order with `en-US` first, or exactly one for an English-only app.
- **Android:** `res/xml/locales_config.xml` listing the nine identities, referenced by `android:localeConfig`, so the system per-app language setting works; `resourceConfigurations` limited to the shipped catalogs.
- **Web:** `<html lang>` is set to the resolved identity at runtime, not left at a hardcoded `en`, in a localized app.

The declaration is checked against the supported list by LOC-9, so adding a catalog without declaring it fails the build.

### LOC-11: Listing and release-note locales track the UI locales

- **Status:** Decided (D-010). Existing (2 of 7: persistent, sonarbridge)
- **Applies to:** Windows, Web, Android apps with a store listing
- **Reference:** `sonarbridge/fastlane/metadata/android/`
- **Why:** The set is one promise to the user, made in the store before the app is installed and kept inside it.

The locale set of the interface, the package declaration, the store listing and the store release notes is the same set. The listing-side rules are [store-listings.md](store-listings.md) LIST-7 and LIST-10, and how localized notes are stored is REL-6 in [releases.md](releases.md). Moving an app from English only to localized is one piece of work that lands together: catalogs, OS declaration, language picker, `listings.json` for every store, and the README `## Languages` section. It is not released in parts.

### LOC-12: Translations are marked as unreviewed until a native speaker reviews them

- **Status:** Decided (D-100). Existing (4 of 7 say so in writing: sonarbridge, hub-display-switch, persistent, phonelock)
- **Applies to:** Windows, Web, Android
- **Reference:** `persistent/docs/localization.md`
- **Why:** Every catalog so far is AI-drafted. Recording that honestly tells a future reviewer what has and has not been checked, and stops a draft being mistaken for approved copy.

Every localized app states in writing which catalogs are AI-drafted and which have had a native-speaker review, and updates the note when a review happens. The four apps keep the note in four different places: a localization doc, a listing companion file, a comment in a catalog file and the README. Under D-100, the app's localization doc (see [docs.md](docs.md)), as one row per wording catalog, beside the list of product terms that must be translated consistently so listing text and interface text agree.

### LOC-13: Text that stays English is listed, not accidental

- **Status:** Decided (D-101). Proposed (3 of 5 localized Windows apps each translate the shared promo page's strings by exact English match: wsl-status, repilot, persistent)
- **Applies to:** Windows, Web, Android, Kit
- **Reference:** `persistent/docs/localization.md` (states what stays English and why)
- **Why:** An untranslated screen in a localized app looks like a bug unless it is a stated choice, and three private copies of the same translation table is a shape that should be extracted.

- A localized app's localization doc lists what is deliberately English: legal pages, log files, diagnostic exports, content fetched from a remote English source, and developer-only surfaces. On Windows it also notes that WinUI's own built-in control text (for example a text box's Cut, Copy, Paste and Select All menu) follows the Windows display language, not the in-app language choice: the kit does not set `PrimaryLanguageOverride`, because doing so makes "Use system setting" resolve to the last chosen language (winui-app-kit README).
- The listing's "claims to keep true" repeats any of these a user would notice ([store-listings.md](store-listings.md), LIST-4).
- Shared UI shipped by a kit is localized in the kit, through the same catalogs mechanism, not re-translated in each host. The promo page is the current case: apps-promo translates its own page text at its newest pin (PROMO-7 in [promo.md](promo.md)), three hosts on older pins still carry an exact-match translation table, and, from apps-promo v2.2.0, the app descriptions are translated in the catalog's own source (D-101, amended 2026-11-18). The host tables are deleted with the pin bump, including any host table that translated descriptions.
