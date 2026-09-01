# AI Workflow Starter Kit

A portable "AI-first engineering workflow" for Claude Code: spec-driven
development (SDD) for ambiguous/high-stakes work, direct/delegated routing
for everything else, git-worktree isolation for concurrent sessions,
review tiers matched to blast radius, and GitHub Issues/Projects as the
shared task book. Extracted from a production repo so the same mechanisms
can be dropped into a new project instead of rebuilt from scratch.

This kit is deliberately thin: a handful of files to copy, a few
placeholders to fill, no framework to learn.

## What's in here

| File | Purpose |
|---|---|
| `TOOLS.md` | Which tools this architecture depends on, how to check/install each |
| `CLAUDE.md.template` | The team-workflow contract — decision ladder, worktree rule, review tiers, branch/PR conventions |
| `docs/architecture.md` | The full mechanism writeup (why, not just what) |
| `.claude/hooks/worktree-guard.mjs` | Auto-detects a concurrent session and blocks edits in the shared clone |
| `.claude/hooks/board-context.mjs` | Surfaces the GitHub Projects board at session start |
| `.claude/hooks/skill-reminder.mjs` | Reminds project conventions the first time a configured file category is edited |
| `.claude/skills/onboard/SKILL.md` | A Claude Code skill that runs this whole bootstrap for you |
| `.claude/settings.json` | Wires the three hooks above |
| `scripts/setup-worktree.sh` | Seeds a new worktree with gitignored local state |
| `scripts/check-branch-overlap.sh` | Checks for file overlap with the remote default branch before pushing |

## Bootstrap checklist

**Fast path:** copy this kit's `.claude/` folder and root files into your
new repo, then tell Claude Code something like "onboard this repo with
this architecture" — the `.claude/skills/onboard/SKILL.md` skill runs the
steps below for you (tool checks, Q&A, file wiring, gentle-ai/CodeGraph
init), asking before it installs or overwrites anything. The manual
steps below are what it does under the hood — still fine to do by hand.

1. **Verify/install tooling.** Open `TOOLS.md` and run each row's check
   command. Install anything missing (Claude Code and CodeGraph are
   load-bearing; Engram and ponytail are optional). Fill in gentle-ai's
   install command yourself — it's an intentional TODO in `TOOLS.md`, not
   an oversight.

2. **Copy the template files into your project root:**
   ```bash
   cp TOOLS.md CLAUDE.md.template /path/to/new-project/
   cp -r docs .claude scripts /path/to/new-project/
   ```
   (If the new project already has a `.claude/settings.json` or a
   `CLAUDE.md`, merge instead of overwrite — see steps 4-5.)

3. **Fill `CLAUDE.md.template` and save it as `CLAUDE.md`** in the new
   project root. Every `<PLACEHOLDER: ...>` marker needs a real answer —
   grep for `PLACEHOLDER` to find them all:
   ```bash
   grep -n "PLACEHOLDER" CLAUDE.md
   ```
   Key ones: project name, GitHub Projects board number/owner, module
   boundaries and dev-server ports, deploy pipeline, local pre-push gate,
   this project's Tier-1 (irreversible/high-blast-radius) surfaces.

4. **Configure the hooks:**
   - `.claude/hooks/board-context.mjs` — set `PROJECT_NUMBER`/`OWNER` at
     the top (or the `GH_PROJECT_NUMBER`/`GH_PROJECT_OWNER` env vars). If
     you don't use GitHub Projects, leave it unconfigured — it no-ops with
     a one-line notice instead of failing.
   - `.claude/hooks/skill-reminder.mjs` — fill the `CATEGORIES` array with
     your project's file-pattern → checklist mapping. Ships empty
     (no-op) until you do.
   - `.claude/hooks/worktree-guard.mjs` — no configuration needed, copy
     as-is.

5. **Wire `.claude/settings.json`.** If the new project doesn't have one
   yet, copy this kit's as-is. If it already has one, merge in the
   `hooks` block (`SessionStart`, `SessionEnd`, `PreToolUse`,
   `PostToolUse` entries) rather than overwriting the file.

6. **Fill in the scripts.**
   - `scripts/setup-worktree.sh` — uncomment and adapt the env-file and
     dependency-seeding sections for your stack (Node/Python/other);
     update the default dev port.
   - `scripts/check-branch-overlap.sh` — update `origin/main` if your
     default branch has a different name.

7. **Opt in to the local hooks path**, if you're using a pre-push git
   hook gate:
   ```bash
   git config core.hooksPath <path-to-your-hooks-dir>
   ```

8. **Verify it's wired correctly:**
   - Start a Claude Code session in the new repo → `board-context.mjs`
     should print board state (or its "not configured" notice) as
     session context.
   - Run `git worktree list` — should show only the main clone. Start a
     second session in the same clone and confirm the `SessionStart`
     warning appears.
   - Try a `Write`/`Edit` from that second session while the first is
     still open — should be denied with worktree instructions.
   - `./scripts/check-branch-overlap.sh` should run without error against
     your remote.
   - If gentle-ai is installed: `gentle-ai codegraph init --cwd
     $(pwd)` should succeed and create a `.codegraph/` index.

9. **Delete what you didn't fill in.** If you're not using GitHub
   Projects, SDD, or a local pre-push gate, say so explicitly in
   `CLAUDE.md` rather than leaving a half-filled placeholder — ambiguity
   here is worse than an explicit "not used."

## Design notes

- **Decision ladder first.** Every task question reduces to: does this
  need SDD, delegation, or just doing it? See `docs/architecture.md` §4
  before reaching for heavier machinery.
- **Worktree activation is orthogonal to task size.** A one-line fix made
  during a concurrent session still needs a worktree.
- **Local session memory is not shared state.** Anything a collaborator
  needs to know goes in a git-tracked file, never left in a
  memory/continuity tool alone (`docs/architecture.md` §6).
