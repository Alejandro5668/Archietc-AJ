---
name: aj-architecture-offboard
description: "Trigger: uninstall AJ architecture, remove AJ architecture, desinstalar arquitectura AJ, quitar arquitectura AJ, offboard AJ, /offboard-aj. Removes the aj-architecture-onboard skill and its cached kit checkout from this machine."
license: MIT
metadata:
  author: "Alejandro5668"
  version: "1.0"
---

## Activation Contract

Trigger only on explicit removal intent ("quitá/desinstalá la arquitectura
AJ", "uninstall AJ architecture", "no quiero más este skill",
"/offboard-aj"). Do not trigger on an ambiguous mention of "AJ" alone.

## Hard Rules

- Never delete anything without one explicit confirmation that names every
  path about to be removed.
- Only touch global, machine-level paths: `~/.claude/skills/aj-architecture-onboard/`
  and `~/.claude/cache/archietc-aj/`. Never touch files inside the current
  project (`CLAUDE.md`, `.claude/hooks/*`, `docs/architecture.md`,
  `scripts/*`) — those were written into that project's own git history by
  `aj-architecture-onboard` and are the user's files now, not this skill's
  to remove.
- If the user also wants a specific project's onboarded files stripped out,
  that is a separate, explicit request — point them at `git rm`/revert in
  that project instead of acting on it here.

## Execution Steps

1. Check what actually exists: list `~/.claude/skills/aj-architecture-onboard/`
   and `~/.claude/cache/archietc-aj/`. Don't assume both are present.
2. Show the exact paths found and ask one confirmation question before
   deleting anything. If neither path exists, say so and stop — nothing to
   remove.
3. On confirmation, remove only the confirmed paths:
   - Windows: `Remove-Item -Recurse -Force <path>`
   - macOS/Linux: `rm -rf <path>`
4. Report what was removed.

## Output Contract

- List of paths actually removed (or "nothing found to remove").
- One line confirming per-project onboarded files were left untouched.
- One line noting this skill's own trigger stops working immediately after
  removal (it deletes itself), and how to reinstall later if needed: clone
  `github.com/Alejandro5668/Archietc-AJ` and copy
  `.claude/skills/aj-architecture-onboard/` back into `~/.claude/skills/`.

## References

- `github.com/Alejandro5668/Archietc-AJ` — source repo; README's "Fastest
  path" section documents the install this skill reverses.
