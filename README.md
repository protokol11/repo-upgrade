# repo-upgrade

repo-upgrade is a pair of agent skills (setup-repo + upgrade-repo) that gives an AI-coding-agent project a standard skeleton: session rules, agent roles, docs conventions, and a compliance package — stamped from one principle catalog, so it never drifts.

## What you get

- **Scaffold a new repo in one pass** → `AGENTS.md`, `CONTEXT.md`, `.pi/agents/`, `.docs/` appear with role prompts injected, plus git init.
- **Upgrade an existing repo via dry-run first** → an audit, a risk-ranked plan, and a backup snapshot before any write happens.
- **Re-stamp the compliance package on every catalog bump** → `lint-rules.md` and `audit-log.md` render from the same source, so agents and the docs agree.

## Why

A new project without a skeleton grows by improvisation: session state scattered across chats, plan files in four formats, agents answering with stale context. The pain is the same for a repo that has already grown this way — every new agent has to reverse-engineer how things are done.

repo-upgrade gives both cases one entry point. `setup-repo` builds the scaffold for a blank folder. `upgrade-repo` walks an existing repo to the same end state, merging instead of overwriting.

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

- **setup-repo**: renders the catalog, runs `scaffold.ps1` to copy templates and inject role prompts, then drills you one question at a time to fill in project facts, then verifies against acceptance criteria.
- **upgrade-repo**: audits the repo against the same criteria, writes `upgrade-plan.md` with per-item risk classes, takes a git snapshot (or a `.archive/` copy), then re-runs the scaffold in `-OnlyMissing` mode. Verification diffs the result against a freshly rendered stamp and refuses to overwrite project-specific content without confirmation.

## Install

Copy this folder into your skills directory:

```powershell
# PowerShell, Windows-oriented — the scaffold script has no POSIX port
cp -Recurse repo-upgrade $env:USERPROFILE\.agents\skills\
```

Dependencies, stated openly:

- **`scaffold.ps1` is PowerShell.** Windows-first; on macOS/Linux you would need PowerShell Core and untested behavior.
- **The `principles` package is required.** Both skills call its renderer (`~/.agents/skills/principles-render/render.mjs`) to produce the compliance artifacts (role prompts, `lint-rules.md`, spec template). Without it, the compliance package cannot be stamped.

## Quick example

```powershell
node $env:USERPROFILE\.agents\skills\principles-render\render.mjs --out $env:TEMP\render
& $env:USERPROFILE\.agents\skills\setup-repo\scripts\scaffold.ps1 -ProjectPath .\my-new-project
```

Then answer the drill questions; the project ends with `AGENTS.md`, `CONTEXT.md`, `.pi/agents/`, `.docs/PLAN.md`, and `.docs/violations/` in place.

## Honest status

Status: **working**. Both skills are in daily use on the maintainer's projects. Known limits: scaffold script is Windows-only; merge behavior on heavily customized repos stops for manual review; no automated test suite ships with this pack.

## License

[MIT](LICENSE) © 2026 protokol11
