# Tools this architecture depends on

Check each row before adopting this kit — only install what you're
missing. **Claude Code, gentle-ai, and CodeGraph are load-bearing**: the
SDD lifecycle, review orchestration, and structural code-intelligence
lookups described in `docs/architecture.md` assume all three. **Engram and
ponytail are optional** — useful, but the worktree/board/review-tier
mechanics work without them.

| Tool | What it does | Why it matters here | Check installed | Install |
|---|---|---|---|---|
| **Claude Code** | The agent runtime itself — sessions, sub-agents, hooks, skills | Everything in this kit (hooks, skills, `CLAUDE.md` loading) runs on it | `claude --version` | https://docs.claude.com/claude-code |
| **gentle-ai** | Orchestrates SDD phases (proposal → spec → design → tasks → apply → verify → archive) + receipt-driven adversarial review + CodeGraph init glue | The engine behind the SDD lifecycle in `docs/architecture.md` §5 | `gentle-ai --version` (this kit was built against v2.2.2) | **TODO — fill in your own install source for gentle-ai here** (`go install <module>@latest` or equivalent). Not captured in this session; don't guess a URL. |
| **CodeGraph** (`@colbymchenry/codegraph`) | Structural code-intelligence index — symbols, call graph, blast radius | Preferred over grep/manual reading for "how does X work" / impact-analysis questions | `npm list -g @colbymchenry/codegraph` | `npm install -g @colbymchenry/codegraph` |
| **Engram** (Claude Code plugin, `Gentleman-Programming/engram`) | Persistent cross-session memory, local to the machine — does not sync between collaborators | Personal continuity across compactions/restarts; not a substitute for git-tracked shared decisions (`docs/architecture.md` §6) | `/plugin list` inside Claude Code, or check for `"engram@engram": true` under `enabledPlugins` in `~/.claude/settings.json` | `/plugin marketplace add Gentleman-Programming/engram` then `/plugin install engram@engram` |
| **ponytail** (Claude Code plugin, `DietrichGebert/ponytail`) | Output style/skill enforcing YAGNI / lazy-minimal-code discipline | Optional workflow-style preference, not structural | Same pattern as Engram | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` |

## Notes

- gentle-ai's install source is a real gap in this kit — whoever adopts it
  needs to supply their own working install command before the SDD phase
  commands in `docs/architecture.md` will work. Don't skip filling this in
  silently; it will look like it works until the first `sdd-*` command
  fails.
- If you don't use gentle-ai at all, the decision ladder and review tiers
  in `CLAUDE.md.template` and `docs/architecture.md` §4 still apply — SDD
  ceremony is optional there regardless of tooling. You'd just be running
  the SDD phases manually (writing proposal/spec/design/tasks by hand)
  instead of through gentle-ai's phase orchestration.
- CodeGraph without gentle-ai still works standalone as an MCP-exposed
  code-intelligence tool; it isn't coupled to gentle-ai's SDD orchestration.
