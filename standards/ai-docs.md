# AI docs

How a repo instructs coding agents: `AGENTS.md`, guides, skills, hooks, and `paseo.json`. Claude Code is the only agent tool in use (D-108). Files stay in tool-neutral locations, and a `.claude/` path is used only where Claude Code cannot read anything else. Documents written for people are in [docs.md](docs.md).

Counts are out of the 15 in-scope repos unless a rule says otherwise. Ten of them have a root `AGENTS.md` today.

## Layout

```
AGENTS.md                      always loaded; routes to everything below
.agents/
  guides/<topic>.md            agent-only guidance (AI-6)
  guides/ui-conventions.md     required for any app with a UI
  skills/<name>/SKILL.md       repeatable workflows (AI-7)
.claude/                       only what Claude Code cannot read from elsewhere
  skills -> ../.agents/skills  tracked symlink (AI-8)
  settings.json                hook registration and project permissions (AI-11)
  agents/<name>.md             subagents, only if the repo defines any (AI-10)
scripts/hooks/                 hook scripts, called from settings.json (AI-11)
docs/                          anything people also read (see docs.md)
paseo.json                     worktree setup and named scripts (AI-12)

No CLAUDE.md, no .codex/, no .claude/commands/.
```

## Where Claude Code looks

| Artefact | What Claude Code does | Where it lives here |
| --- | --- | --- |
| Root `AGENTS.md` | Loaded at session start, but only when no `CLAUDE.md`, `CLAUDE.local.md` or `.claude/CLAUDE.md` exists in the folder or its parents | Root `AGENTS.md`; never a `CLAUDE.md` (AI-5) |
| Nested `AGENTS.md` | Loaded when a file in that folder is read | Workspace folders (AI-3) |
| Skills | Discovered only under `.claude/skills/`; a symlinked folder is followed | `.agents/skills/`, reached through a tracked `.claude/skills` symlink (AI-8) |
| Hooks | Registered only in `.claude/settings.json`; the command can run any script | Registration in `.claude/settings.json`, scripts in `scripts/hooks/` (AI-11) |
| Subagents | Discovered only under `.claude/agents/` | `.claude/agents/`, no neutral option (AI-10) |
| Commands | Merged into skills | Skills only (AI-10) |
| `.agents/guides/`, `docs/` | Read only when `AGENTS.md` says so | Guidance map in `AGENTS.md` (AI-2) |
| Global rules | `~/.claude/CLAUDE.md`; it does not stop project `AGENTS.md` files loading | Not restated in projects (AI-14) |

Checked by test on 2026-11-16 with Claude Code 2.1.322: `AGENTS.md` loaded in `/development` and in a worktree under the home folder; a skill was found through both a whole-folder and a per-skill symlink; a `SessionStart` hook ran a script in `scripts/hooks/` through `$CLAUDE_PROJECT_DIR`. Re-check when Claude Code changes how it discovers files.

## Rules

### AI-1: Every repo has a root `AGENTS.md`

- **Status:** Existing (store-sign-controller, phonelock, devkit and apps-promo now have one, rechecked 2026-11-25; printstream-home-assistant is exempt; recount pending). The generated repo's exemption is Decided (D-089)
- **Applies to:** all
- **Reference:** `tray-control-for-wsl/AGENTS.md` (small app); `persistent/AGENTS.md` (monorepo); `tray-control-for-wsl/external/winui-app-kit/AGENTS.md` (kit)
- **Why:** It is the only file both tools load without being told, and kits need one most because their mistakes reach every consumer.

Every repo has an `AGENTS.md` at its root, including kits and tooling. A kit's file can be very short: the rules only an agent would get wrong. printstream-home-assistant is generated from printstream, whose export removes agent files. Under D-089 its instructions are a nested `AGENTS.md` in `printstream/integrations/home-assistant/` (none exists today), and the generated repo is exempt from this rule.

### AI-2: Skeleton, title, and section names

- **Status:** Proposed (most audited repos now use the title form, rechecked 2026-11-25; section names still differ; recount pending)
- **Applies to:** all
- **Reference:** `tray-control-for-wsl/AGENTS.md` for shape; `little-launcher/AGENTS.md` for the guidance map and "Keeping docs current" tables; `persistent/AGENTS.md` for "Shared helpers"
- **Why:** Fixed headings let an agent, and the maintainer, find the same thing in the same place in every repo.

Use this skeleton. Sections marked optional are omitted, not left empty.

```markdown
# <Product name>: Agent Instructions

<One paragraph: what it is, who uses it, how it ships, and which shared kits it builds on.>

## Standards

This repo follows the shared standards at https://github.com/Dynamic-Solutions-Canada/app-standards. Read the index for this stack, https://github.com/Dynamic-Solutions-Canada/app-standards/blob/main/stacks/<stack>.md, then the topic file for the area you are changing. If `external/standards/` exists, it is the same content pinned for offline use. Where this file differs from the standards, this file wins and says why.

## Architecture

<Small repo: an area-to-files table. Larger repo: three to six bullets on ownership boundaries, then a link to `ARCHITECTURE.md`.>

## Behaviour

<Optional. Product rules an agent could break without noticing.>

## Conventions and gotchas

<Project-specific only. One line each, with the reason. Nothing the standards or the global rules already say.>

## Shared helpers (do not duplicate)

<Optional. "For X, use `helper` from `path`.">

## Commands

<Build, verify, test, run. The exact command, what it proves, and what it does not prove.>

## Release

<Optional. One paragraph and the name of the release skill.>

## Guidance map

| Guide, doc, or skill | Read or use when | Governs |
| --- | --- | --- |

## Keeping docs current

| What changed | Update |
| --- | --- |
```

Rules that go with it:

- The title is `# <Product name>: Agent Instructions`, with the product name from [branding.md](branding.md).
- Architecture is not explained twice. Where `ARCHITECTURE.md` exists, this file links to it (see DOCS-3 in [docs.md](docs.md)).
- The guidance map lists every guide, reference doc, glossary, and skill an agent should know about, with the paths each governs.
- Monorepos add one line under Architecture: "Nested `AGENTS.md` files hold area rules. List them with `git ls-files '*AGENTS.md'` and read the ones for the folders you touch."

### AI-3: Length and nesting

- **Status:** Decided (D-031). Existing (11 root files are over about 1,500 words on 2026-11-25: displayswitchr 2,151, chatmixr 1,938, tray-control-for-wsl 1,834, phonelock 1,797, immich-drive 1,751, sonarbridge 1,708, little-launcher 1,707, persistent 1,615, repilot 1,604, printstream 1,560, winui-app-kit 1,535)
- **Applies to:** all
- **Reference:** `persistent/AGENTS.md` (1,313 words, detail pushed to nested files and one guide)
- **Why:** The root file is loaded into every session, so every extra word is paid for on every task.

The root `AGENTS.md` stays under about 1,500 words. Anything longer moves to a guide, a reference doc, or a nested `AGENTS.md`. Monorepos keep nested files at workspace level only (`apps/*`, `packages/*`), with finer rules in guides. Nested files are never listed by hand in the root file; they are enumerated with `git ls-files`.

### AI-4: Link to these standards

- **Status:** Decided (D-002, D-004). Existing (essentially every audited repo links the standards, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** the `## Standards` section of the skeleton in AI-2
- **Why:** A link works for people, local agents, containers, and cloud sessions; a sibling path works on one machine.

Every `AGENTS.md` has a `## Standards` section, directly after the intro, with the GitHub URL of the standards repo and of the stack index that applies. That section is a pointer, not a summary: it does not restate any rule. The standards repo is public (D-004), so the link resolves for outside readers, CI and cloud sessions as well as for the maintainer; the URL is filled into the skeleton once the repo has been pushed.

A repo may also add the standards as a submodule at `external/standards` for offline reading. The link remains the source of truth, and a repo that adds the submodule initialises it in its worktree setup (AI-12). The repo never refers to the standards by a host path.

### AI-5: One tool-neutral instruction file

- **Status:** Decided (D-108). Existing (no audited repo has a `CLAUDE.md` or addresses Codex by name, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `tray-control-for-wsl/AGENTS.md`
- **Why:** Two instruction files drift, and guidance written for one tool reads as optional to the other.

`AGENTS.md` is the single instruction file. A repo never has a `CLAUDE.md`, `CLAUDE.local.md`, or `.claude/CLAUDE.md`: Claude Code stops reading `AGENTS.md` when any of them exists, so adding one silently drops the real instructions. No other tool-specific instruction file (`.cursorrules`, `.github/copilot-instructions.md`) is kept.

Titles, headings, and skill descriptions do not name a tool: write "the agent", not "Codex" or "Claude". Skills are referred to by name in backticks. Instructions addressed to Codex are rewritten or removed.

### AI-6: Guides

- **Status:** Decided (D-026). Existing (most repos with guides use `.agents/guides/`, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `little-launcher/docs/` with its index table in `little-launcher/AGENTS.md`; `printstream/.agents/guides/ui-conventions.md` for depth of content
- **Why:** A tool-neutral home means both agents find the guide, and one copy per topic means it cannot contradict itself.

A topic people also read lives in `docs/` (see DOCS-5 in [docs.md](docs.md)); material only an agent needs lives in `.agents/guides/<topic>.md`. Nothing lives under `.codex/` (AI-10). A topic is never kept in both places (FIX-040).

A guide is one topic per file, named in kebab-case, and aims for under about 5 KB. It opens with two lines, "Read when" and "Governs" (path globs), matching the guidance map columns. A guide over about 10 KB starts with a section index. Guides have no frontmatter, since neither tool reads it.

Every app with a UI has `.agents/guides/ui-conventions.md`, the path the user-level agent rules (kept outside this repo) already name (1 of the 11 end-user apps has one today: printstream). It records the app's own conventions and links to [ui.md](ui.md) for the shared ones.

### AI-7: Skills

- **Status:** Existing (9 of 15 keep workflows as skills under `.agents/skills/`; all 9 use `name` and `description` frontmatter)
- **Applies to:** all
- **Reference:** `sonarbridge/.agents/skills/` (small, single-purpose, trigger-style descriptions); `persistent/.agents/skills/verify/SKILL.md`
- **Why:** A skill is found by its description, so a vague one never runs and an overlapping one runs at the wrong time.

A repeatable workflow is a skill at `.agents/skills/<name>/SKILL.md`.

- **Name:** kebab-case, verb first, equal to the folder name. The standard names are `commit`, `release`, `deploy`, `verify`, `audit-docs`, and `desktop`; use them when the skill does that job.
- **Frontmatter:** `name` and `description` only, unquoted unless YAML requires it. The description says what the skill does, then "Use when..." and, where it helps, "Do not use for...".
- **Body:** a one-line goal, numbered steps, and a final "Report" line saying what evidence to give. A skill links to a guide or doc for detail and does not repeat it.
- **No tool-specific metadata:** remove `agents/openai.yaml` files; nothing reads them (D-108).

A skill does not hold configuration, such as a commit author (FIX-005).

### AI-8: Claude Code reaches project skills through a symlink

- **Status:** Decided (D-108, D-028). Existing (essentially every audited repo with skills has the symlink, rechecked 2026-11-25; recount pending); the mechanism was tested on 2026-11-16
- **Applies to:** all repos with skills
- **Reference:** none yet
- **Why:** Claude Code reads only `.claude/skills/`, so every tracked project skill is invisible to it today; the symlink makes them visible without moving them into a Claude-only folder.

Each repo with skills tracks one symlink, `.claude/skills -> ../.agents/skills`, created with `ln -s ../.agents/skills .claude/skills` and committed as a symlink. `.agents/skills/` stays the one real location; never put a skill directly in `.claude/skills/`.

The Windows build tooling (windows-build-tools) skips `.claude/` and `.agents/` at any depth when it stages a source tree, so the symlink does not break `verify:desktop` or packaging; it still rejects any other symlink. A Windows app adds the symlink, or bumps to a winui-app-kit tag that contains it, only once it uses a tooling version with that exclusion.

Repos are worked on from WSL and Linux, where git checks symlinks out correctly. A checkout made by Windows git with `core.symlinks` off turns the link into a small text file and Claude Code there sees no skills; that is acceptable because no agent works from a Windows checkout. If that changes, switch to per-skill symlinks or revisit this rule.

### AI-9: Shared skills

- **Status:** Decided (D-029). Existing (the generic `commit` skill is installed at user level since 2026-11-18; repo copies are being trimmed to what they add; the `desktop` skill is still copied into 6 repos)
- **Applies to:** all
- **Reference:** the maintainer's user-level `commit` skill (`~/.agents/skills/commit/`, linked into `~/.claude/skills/`)
- **Why:** A copied skill drifts the same way a copied doc does.

A skill that is the same in every repo (`desktop`, the generic `commit`, the post-release steps in D-063) is installed once at user level, in `~/.agents/skills/` with `~/.claude/skills` linked to it the same way as AI-8. A repo that must work in cloud sessions keeps a copy, checked against the shared original for drift. A repo that opts into the `external/standards` submodule may symlink from it instead.

A project skill with the same name as a shared one replaces it, and says in its first line what it adds, for example persistent's data-isolation review in `commit`.

### AI-10: No `.codex/`; `.claude/` holds only what Claude Code requires

- **Status:** Decided (D-108). Existing (no audited repo tracks `.codex/`, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** none yet
- **Why:** Codex is not used, and files under `.codex/commands/` were never commands in any tool; they duplicate the `desktop` skill beside them.

`.codex/` is removed from every repo. Its content moves first: workflows become skills (AI-7; immich-drive's five real workflows included), guidance becomes a guide or a doc (AI-6), and Codex hooks that still matter become git hooks or Claude Code hooks (AI-11).

`.claude/` holds only `skills` (the AI-8 symlink), `settings.json`, and `agents/` when the repo defines subagents, because Claude Code reads those from nowhere else. Anything that can live elsewhere does: hook scripts in `scripts/hooks/`, guidance in `.agents/guides/` or `docs/`. There is no `.claude/commands/`; commands are skills. `.claude/settings.local.json` is personal and git-ignored.

### AI-11: Tool hooks

- **Status:** Decided (D-030, D-108). Existing (no audited repo has Codex hooks left, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `persistent/.codex/hooks.json` and `persistent/.codex/README.md` for what the hooks do (to be ported)
- **Why:** A guard that runs in no tool in use is not a guard, and a hook nobody can explain gets disabled.

Anything that must never be committed (generated files, secrets) is enforced by a git hook, which applies to every tool and to people; see HYG-10 in [repo-hygiene.md](repo-hygiene.md). Tool hooks are for guidance during a session only: reminders by path, targeted validation, worktree preparation.

When a repo has Claude Code hooks:

- They are registered in `.claude/settings.json`, the only place Claude Code reads project hooks. The registration only names the script: `"$CLAUDE_PROJECT_DIR"/scripts/hooks/<name>.mjs`.
- The script lives in `scripts/hooks/`, so it is ordinary repo code that is reviewed, tested and reusable outside Claude Code.
- `scripts/hooks/README.md` says what each hook blocks or runs and what to do instead.
- Hook scripts resolve paths from the git root, not from a host path.
- A hook script used by more than one repo ships from devkit, not as a copy.
- The Codex hooks in persistent, printstream and game-is-up are ported or dropped when `.codex/` is removed (AI-10).

### AI-12: `paseo.json`

- **Status:** Existing (persistent, printstream, game-is-up, store-sign-controller and the three newer WinUI apps, rechecked 2026-11-25; recount pending). Decided (D-045) for the shared template and (D-088) for the standalone Windows apps
- **Applies to:** Web; Windows (proposed)
- **Reference:** `persistent/paseo.json`
- **Why:** A new worktree that is missing its submodules or dev environment fails in ways that look like code bugs.

A repo that agents open worktrees in has a `paseo.json` at its root with a worktree `setup` and `teardown` and one named script per canonical command in [dev-scripts.md](dev-scripts.md). Script keys are the command names with colons replaced by hyphens (`dev-doctor`, `validate`, `test`, `build`).

Devkit repos use devkit's setup and teardown pair as documented in devkit's README; do not re-document it. Repos with submodules run `git submodule deinit --all --force` in teardown, because `git worktree remove` refuses a worktree that still has submodules (reference: immich-drive's `paseo.json`). Under D-045, devkit generates the file from a template. Under D-088, the Windows repos' file initialises submodules on setup and exposes the four `*:desktop` scripts.

Where worktrees are created is not stated in a project file (D-032, FIX-041): the maintainer's global rules put them inside the main checkout at `.worktrees/<name>/`, and each repo ignores that folder in git, its Docker build context, and its lint, format, test and watcher settings.

### AI-13: No host paths, sibling links, or personal details

- **Status:** Proposed (displayswitchr, winui-app-kit, printstream, game-is-up and sonarbridge are now clean, rechecked 2026-11-25; recount pending)
- **Applies to:** all
- **Reference:** `immich-drive/AGENTS.md` (refers to a sibling only "if available and permitted by the current environment")
- **Why:** Tracked instructions are read in containers, CI, cloud sessions, and public repos, where a path on the maintainer's machine is wrong and a personal detail is a leak.

Tracked agent files use repo-relative paths. They contain no absolute host path, no `../sibling` link, no description of one machine (its CPU architecture, its folder layout), and no hostname, device name, or personal address. A reference to another project is a URL. Machine and device facts stay in the maintainer's user-level files, outside the repo.

A cross-repo convention ("do it the way Little Launcher does") is replaced by a link to the rule in these standards once the rule exists.

Tracked files name roles, not people: a skill or guide says "the maintainer" (or "the release approver") where a person must act or approve, never a personal name.

### AI-14: No restated global rules; one commit and push rule

- **Status:** Decided (D-013) for commit, push, release and deploy authority. Proposed for restatement (FIX-043)
- **Applies to:** all
- **Reference:** none yet
- **Why:** A copied rule goes stale when the original changes, and five wordings of the push rule mean an agent picks whichever it read last.

Project files do not restate the user-level agent rules, these standards, or what a kit's README documents. A project file may apply a user-level rule to a named thing in the project ("use `AppDialog` for every dialog"), which is a project rule, not a restatement. Generic lines such as "avoid god files", "comments explain why", the ban on attribution trailers, and "leave changes uncommitted" are removed.

One rule, stated once in the shared `commit` skill and not per repo (D-013): an agent commits only when asked and pushes only when explicitly requested. Invoking a `release` or `deploy` skill is that request for the skill's own work: it authorises the commit, the tag and the push the procedure makes, end to end, without a second confirmation. It authorises nothing else, so changes that are not part of the release or deploy are not committed along with it. Whether the stops a release procedure defines for itself (approval of the notes, a dirty tree) still apply under that grant is open; see REL-8 in [releases.md](releases.md).

### AI-15: Agent files stay true

- **Status:** Existing in most repos; writing-rule violations are tracked as FIX-044 and FIX-045; the stale references in FIX-034, FIX-036, and FIX-039 are resolved
- **Applies to:** all
- **Reference:** `little-launcher/AGENTS.md` "Keeping docs current" table; the `audit-docs` skill in persistent, printstream, game-is-up, and little-launcher
- **Why:** An agent follows a stale instruction exactly, so a deleted file named in a skill becomes a recreated file.

A change that renames or removes a file, command, or setting updates every `AGENTS.md`, guide, and skill that names it, in the same commit. The "Keeping docs current" table in `AGENTS.md` is the checklist, and skills are part of it. Repos with more than a handful of guides keep an `audit-docs` skill that checks paths and commands named in agent files against the tree.

Agent files follow the writing section of the user-level agent rules, which is not restated here.
