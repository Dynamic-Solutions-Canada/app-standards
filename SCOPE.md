# Scope

Projects held to these standards, as approved on 2026-11-15.

## In scope

| Project | Kind |
| --- | --- |
| hub-display-switch | Windows (WinUI) |
| wsl-monitor | Windows (WinUI) |
| little-launcher | Windows (WinUI) |
| repilot | Windows (WinUI) |
| immich-drive | Windows (WinUI) |
| persistent | Web monorepo with Windows desktop app |
| printstream | Web monorepo |
| game-is-up | Web monorepo |
| store-sign-controller | Web monorepo; a private project that stays private, so rules that exist only because a repo is public (D-004, scrubbing history for public release, and CI-8 code scanning, which needs a paid plan on a private repo) do not apply, and every other rule does |
| phonelock | Android |
| sonarbridge (formerly sonarphone-nmea-bridge) | Android plus ESP32 |
| printstream-home-assistant | Home Assistant integration (PrintStream add-on); generated from `printstream/integrations/home-assistant/`, so fixes land in printstream |
| devkit | Shared dev tooling |
| winui-app-kit | Shared WinUI kit (submodule) |
| apps-promo | Shared promo catalog and page (submodule) |

Shared projects are both a source of conventions and held to them (READMEs, docs, AI docs).

## Out of scope

- Client work, which keeps its own standards
- shopify-form-filler, shopify-raw-material-tracker
- android-vm, paseo-shared-browser
- windows-build-tools, dynamic-solutions-website (used as references, not held to the standards)
- paseo-branch-tracker, esphome-lvgl, kotn_helper
- printstream-import-extension (possibly dead)
- Repos in other GitHub organisations and personal forks
- Upstream projects the maintainer contributes to, all `*.worktrees`, printstream-public (a dead clone; live public exports are produced under `printstream/data/exports/`)
