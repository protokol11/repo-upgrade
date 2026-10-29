# repo-upgrade

Agent skills that stamp a standard repo skeleton from one principle catalog, so agentic projects never drift.

Two skills cover both entry states: `setup-repo` scaffolds a blank folder, `upgrade-repo` walks an existing repo to the same end state, merging instead of overwriting.

## Why

A new project without a skeleton grows by improvisation. Session state scatters across chats, plan files appear in four formats, every new agent reverse-engineers how things are done — and answers with stale context. The cost is identical for a repo that has already grown this way.

The root cause is a missing single source of truth. When each file is written by hand, the lint rules, the role prompts and the docs each drift on their own schedule. repo-upgrade renders both skills' artifacts from one principle catalog (`principles-render`), so the compliance package is a stamp of a single source: bump the catalog, re-stamp, and agents and docs agree again.

Scope: scaffold and upgrade of agent-coding repos on this toolchain. Non-scope: general project templating, CI setup, dependency management — none of those are touched.

## Install

Prerequisites:

- **PowerShell.** `scaffold.ps1` is Windows-first and has no POSIX port. On macOS/Linux it needs PowerShell Core, and that combination is untested.
- **The `principles` package.** Both skills call its renderer (`~/.agents/skills/principles-render/render.mjs`) to produce the compliance artifacts (role prompts, `lint-rules.md`, spec template). Without it, the compliance package cannot be stamped.
- **Node.js**, to run the renderer.

Command:

```powershell
cp -Recurse repo-upgrade $env:USERPROFILE\.agents\skills\
```

Expected result: `setup-repo` and `upgrade-repo` appear next to your other skills, and `setup-repo\scripts\scaffold.ps1` resolves:

```powershell
Test-Path $env:USERPROFILE\.agents\skills\repo-upgrade\setup-repo\scripts\scaffold.ps1
# True
```

## Usage

One path to one worked result — scaffold a new project:

```powershell
node $env:USERPROFILE\.agents\skills\principles-render\render.mjs --out $env:TEMP\render
& $env:USERPROFILE\.agents\skills\setup-repo\scripts\scaffold.ps1 -ProjectPath .\my-new-project
```

Then answer the drill questions one at a time as the skill asks them. When it finishes, the project holds the skeleton:

```powershell
Get-ChildItem .\my-new-project -Name
# AGENTS.md
# CONTEXT.md
# .pi
# .docs
```

For an existing repo, run `upgrade-repo` instead: it audits first, writes `upgrade-plan.md` with per-item risk classes, takes a snapshot, then re-runs the scaffold in `-OnlyMissing` mode and diffs the result against a freshly rendered stamp before writing anything.

## How it works

Two skills, one renderer between them:

```
                 ┌─────────────────────────────┐
                 │  principles-render (package) │
                 │  render.mjs → role-prompts/  │
                 │  lint-rules.md, spec-template│
                 └──────────┬──────────────────┘
                            │ rendered once
            ┌───────────────┴────────────────┐
            ▼                                ▼
  ┌───────────────────┐            ┌───────────────────┐
  │    setup-repo     │            │   upgrade-repo    │
  │ scaffold.ps1      │            │                   │
  │ copies templates: │  applies   │ 1. explore+audit  │
  │ AGENTS.md,        │ ──────────▶│    → AUDIT.md     │
  │ CONTEXT.md,       │  missing   │ 2. render catalog │
  │ .pi/agents/*,     │  files +   │ 3. plan (risks)   │
  │ .docs/PLAN.md,    │  re-stamp  │ 4. backup snapshot│
  │ .docs/violations/ │            │ 5. scaffold.ps1   │
  └───────────────────┘            │    -OnlyMissing   │
                                   │ 6. verify + status│
                                   └───────────────────┘
```

- **setup-repo**: renders the catalog, copies templates and injects role prompts, then drills you one question at a time to fill in project facts, then verifies against acceptance criteria.
- **upgrade-repo**: audits the repo against the same criteria, writes `upgrade-plan.md`, takes a git snapshot (or a `.archive/` copy), re-runs the scaffold in `-OnlyMissing` mode, and refuses to overwrite project-specific content without confirmation.

## Skills

Each skill is driven through its own `SKILL.md`; the README is the map, the skill files are the interface.

- `setup-repo` — scaffold a blank folder. Invocation: run the skill, it calls `scripts\scaffold.ps1 -ProjectPath <dir>` and drills for project facts.
- `upgrade-repo` — upgrade an existing repo. Invocation: run the skill; it audits, plans, snapshots, then scaffolds with `-OnlyMissing` and verifies against a fresh render.

## Contributing

Both skills are in daily use on the maintainer's projects. Merge behavior on heavily customized repos stops for manual review; no automated test suite ships with this pack — report behavior against the steps above as an issue in the parent repository.

## License

[MIT](LICENSE) © 2026 protokol11
