# Standards

Shared conventions for the Dynamic Solutions family of apps: how they are built, documented, released, and presented, so that every app looks and works like part of one family.

> **Status: draft.** The rules here were written from an audit of the apps on 2026-11-15 and 2026-11-16 (see `audit/`). Every open decision was settled on 2026-11-16; they are recorded in [DECISIONS.md](DECISIONS.md), and the rules that rest on one are marked `Decided (D-xxx)`. Most are not followed by every repo yet, which is parity work.
>
> **Audit snapshot caveat:** the audit read local checkouts on 2026-11-15 and 2026-11-16. persistent and store-sign-controller were behind GitHub and were fast-forwarded afterwards, and the facts that changed (store-sign-controller is now web only) are corrected here. devkit's checkout is still behind GitHub, so findings about devkit may describe older code. printstream was one commit ahead of GitHub.

## Who it applies to

The apps and shared projects listed in [SCOPE.md](SCOPE.md). Each rule names the stacks it applies to: **Windows** (WinUI tray and desktop apps), **Web** (Node monorepos), **Android**, **HA** (Home Assistant integration), and **Kit** (shared projects such as winui-app-kit, apps-promo, and devkit).

## How an app uses these standards

- **Link:** every app's `AGENTS.md` links to this repo on GitHub and to the stack index that applies to it. A link works for people, local agents, containers, and cloud sessions alike.
- **Submodule (optional):** an app may also add this repo as a submodule at `external/standards` so agents can read the rules offline at the pinned version. The link stays the source of truth; the submodule is a convenience.
- **Cite rules by ID.** Commits, reviews, and parity checklists refer to rules by their ID (for example `README-3`) so a rule can be found and its status checked.

This repo is public, so the link resolves for everyone (D-004). `audit/` and the security fixes are private working material and are not published with it.

## What lives where

| Path | Purpose |
| --- | --- |
| `standards/` | One file per topic. These are the rules. |
| `stacks/` | One index per stack, listing the rule IDs that apply to it, so an app's agent loads only what it needs. |
| [DECISIONS.md](DECISIONS.md) | Every decision behind the rules: what was decided and what it changes, plus the adopted defaults with the options that were considered. New open questions are added there under "Open decisions". |
| [FIXES.md](FIXES.md) | Concrete defects found in the audit that need fixing regardless of any decision. Security fixes (FIX-001 to FIX-010) are tracked privately and cited by ID only. |
| [SCOPE.md](SCOPE.md) | Which repos are held to these standards. |
| `scripts/build-stacks.mjs` | Regenerates `stacks/` from the rules. Run it after changing any rule's ID, title, status, or "Applies to" line; `--check` fails if the indexes are stale. |
| `audit/` | The evidence the rules were written from, and the security fixes. Private: git-ignored and not published (D-004). Not a standard; it may go stale. |

## How a rule is written

Each rule in `standards/` has:

- **An ID:** a topic prefix and a number, such as `README-3`. IDs are never reused or renumbered; a retired rule stays in place, marked retired.
- **A status:**
  - `Existing (N of M)`: already followed by N of the M in-scope repos it applies to.
  - `Proposed`: no repo or only an excluded repo does it today; adopting it is new work.
  - `Pending D-xxx`: the rule's content depends on an open decision. The text shows the recommended option. No rule is pending as of 2026-11-16.
  - `Decided (D-xxx)`: it is settled; repos that do not follow it yet are parity work.
  - `Retired`: no longer applies; kept so the ID is not reused.
- **Applies to:** the stacks it covers.
- **Reference:** the best existing example to copy, as a repo path.
- **Why:** one or two sentences of reason, so a future reader can judge edge cases.

## What does not belong here

- **The user-level agent rules** (kept outside this repo) cover writing style, commits, dates and times, memory limits, and so on. They live in the maintainer's own agent instructions and apply everywhere already. These standards do not restate them, and apps must not copy them into their own `AGENTS.md`.
- **What a shared kit already documents.** winui-app-kit, apps-promo, and devkit own their own usage docs. The standards say *when* to use them and link there; they do not re-explain how.

## License

MIT. See [LICENSE](LICENSE).
