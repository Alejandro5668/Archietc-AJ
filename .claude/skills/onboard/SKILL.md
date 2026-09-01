---
name: onboard
description: "Bootstrap this AI-first workflow starter kit (SDD routing, git-worktree isolation, review tiers, GitHub-issues task book) into a new or existing project. Trigger: 'onboard this repo', 'onboard this repo with the AJ architecture', 'set up the AI workflow starter kit', 'adopt the AJ architecture', '/onboard'."
license: MIT
metadata:
  version: "1.0"
---

## Purpose

One-shot guided bootstrap of this starter kit into the current repo: checks
tooling, fills `CLAUDE.md` placeholders via short Q&A, wires the hooks and
scripts, and runs gentle-ai/CodeGraph init for this project. Replaces
following `README.md`'s checklist by hand. This is a procedural runbook —
execute the numbered steps in order, don't skip the confirmation gates.

## Preconditions

- Run from the root of the target project (the repo being onboarded), git
  initialized.
- The starter kit's own files (`TOOLS.md`, `CLAUDE.md.template`,
  `docs/architecture.md`, `.claude/hooks/*.mjs`, `scripts/*.sh`) must be
  reachable — either already copied into this repo, or in a known
  starter-kit checkout to copy from.

## Runbook

### 0. Locate source files

- Check whether `CLAUDE.md.template` and `TOOLS.md` already exist at this
  repo's root.
  - Yes → the starter kit was already copied in; source root = this repo
    root. Go to step 1.
  - No → ask the user for the path to their `ai-workflow-starter-kit`
    checkout. Do not guess a path.

### 1. Tool check (read `TOOLS.md` for the authoritative table, run its checks)

- `claude --version`
- `gentle-ai --version`
- `npm list -g @colbymchenry/codegraph`
- Engram: `/plugin list`, or grep `enabledPlugins` in
  `~/.claude/settings.json` for `"engram@engram": true`
- ponytail: same pattern, `"ponytail@ponytail": true`

Classify each present/missing. Claude Code itself is a precondition
(you're running inside it) — skip re-verifying unless the version check
errors.

### 2. Confirm before installing anything (one batched question)

If anything is missing: list ALL missing tools with their install commands
from `TOOLS.md` in a single message, then ask ONE batched question — which
of these to install now (per-tool y/n, or "skip all"). Never silently run
a global npm install, a Go binary install, or a Claude Code plugin
marketplace add. gentle-ai has no captured install command in this kit
(`TOOLS.md` marks it TODO) — flag it as a manual step, never invent an
install command or URL for it.

Wait for the answer before installing anything. Tools the user skips are
noted for the final summary, not blockers — steps 5-6 degrade gracefully
if gentle-ai/CodeGraph end up absent.

### 3. Project Q&A — fill `CLAUDE.md.template` placeholders (one batched message)

Ask these together, skipping any whose answer is obvious from repo
inspection (e.g. no board usage, or a single-package repo with no
frontend/backend split):

1. Project name?
2. GitHub Projects board in use? If yes: board number + owner (org/user).
   If no: mark the board section unused.
3. Module boundaries / dev-server ports (e.g. "backend on 8000, frontend
   on 3000")?
4. Deploy pipeline notes (auto-deploy on push? manual script? which
   environments?) — or "none yet".
5. Any project-specific Tier-1 (irreversible/high-blast-radius) surfaces —
   auth, payments, tenant isolation, etc.?

### 4. Write `CLAUDE.md`

- If `CLAUDE.md` already exists at repo root: summarize what would change
  and ask before overwriting. Never overwrite silently.
- Otherwise: fill `CLAUDE.md.template`'s placeholders from the Q&A
  answers, write the result to `./CLAUDE.md`.

### 5. Copy/finalize project files

- `docs/architecture.md` → copy as-is to `./docs/architecture.md` (already
  generic, no edits needed).
- `.claude/hooks/board-context.mjs` → copy; fill `PROJECT_NUMBER`/`OWNER`
  from the Q&A answer, or leave the placeholder and note it's unused if no
  board.
- `.claude/hooks/worktree-guard.mjs` → copy verbatim, no config needed.
- `.claude/hooks/skill-reminder.mjs` → copy; leave `CATEGORIES` empty
  unless the user wants to define per-stack checklists right now (offer,
  don't force — fillable later).
- `.claude/settings.json` → if the target repo already has one, MERGE the
  four hook entries (`SessionStart`, `SessionEnd`, `PreToolUse`,
  `PostToolUse`) into its existing `hooks` block instead of overwriting
  the file; leave existing permissions/hooks untouched. If none exists,
  copy as-is.
- `scripts/setup-worktree.sh`, `scripts/check-branch-overlap.sh` → copy;
  leave their `PLACEHOLDER` customization sections for the user (don't
  guess dependency-manager specifics you weren't told).

### 6. gentle-ai / CodeGraph init (only for tools confirmed present in step 1/2)

- If gentle-ai is present: run `gentle-ai codegraph init --cwd
  <project-root>` to initialize CodeGraph for this project (this validates
  the root before delegating to CodeGraph's own init). Do not call
  `codegraph init` directly when gentle-ai is available, and never run
  `codegraph uninit`, `upgrade`, `install`, or `uninstall` — out of scope.
- If gentle-ai is absent but CodeGraph is present: run `codegraph init`
  directly on the project root.
- If the user plans to use SDD: tell them to run the `sdd-init` skill next
  (it detects stack and bootstraps the artifact-store backend) — don't
  replicate its logic here.
- If gentle-ai isn't installed on this machine: skip both sub-steps, note
  it in the summary.

### 7. Summary

Report, in this order:
- Tools installed vs. skipped (from step 2).
- Files written/merged vs. left alone because they already existed
  (CLAUDE.md, hooks, settings.json, scripts).
- What still needs manual attention: unfilled `PLACEHOLDER` markers (e.g.
  deploy pipeline notes, empty `skill-reminder` `CATEGORIES`).
- Suggested next steps: commit these files; run `sdd-init` if adopting
  SDD; `git config core.hooksPath <path>` if adopting local pre-push
  checks.

## Guardrails

- Never install anything global-machine-scoped without the batched
  confirmation in step 2.
- Never overwrite an existing `CLAUDE.md` or `.claude/settings.json`
  without showing the change and getting explicit go-ahead.
- Never invent a gentle-ai install command or URL.
- Never run `codegraph uninit`, `codegraph install/uninstall`, or
  `codegraph upgrade` — out of scope for onboarding.
- Keep each Q&A a single batched message, not one question per item.
