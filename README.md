# pagerunner-skill

Agent Skill for [Pagerunner](https://github.com/Enreign/pagerunner) — real Chrome browser automation for Claude (and any MCP client) with authenticated sessions, PII anonymization, sealed secrets, site adapters, session checkpoints, and video recording.

## The skill lives in [`skills/pagerunner-skill/`](./skills/pagerunner-skill)

Start here: [`skills/pagerunner-skill/SKILL.md`](./skills/pagerunner-skill/SKILL.md) — ICP quick starts, gotchas, and the core workflow.

Full doc set:

| Doc | What it covers |
|---|---|
| [SKILL.md](./skills/pagerunner-skill/SKILL.md) | Main entry — 4 ICP quick starts, 7 rules for agents, gotchas |
| [REFERENCE.md](./skills/pagerunner-skill/REFERENCE.md) | All ~44 MCP tools with signatures + examples |
| [PATTERNS.md](./skills/pagerunner-skill/PATTERNS.md) | Workflow patterns — form filling, auth, scrolling, multi-step |
| [SECURITY.md](./skills/pagerunner-skill/SECURITY.md) | PII anonymization, sealed secrets, credential scrubbing, audit log |
| [RECORDING.md](./skills/pagerunner-skill/RECORDING.md) | Director's guide to making great videos (v0.8) |
| [ADVANCED.md](./skills/pagerunner-skill/ADVANCED.md) | Multi-agent coordination, session persistence, auto-recovery |
| [EXAMPLES.md](./skills/pagerunner-skill/EXAMPLES.md) | 4 full ICP workflows + multi-agent patterns |
| [DEBUGGING.md](./skills/pagerunner-skill/DEBUGGING.md) | Troubleshooting |
| [HALLUCINATION_PREVENTION.md](./skills/pagerunner-skill/HALLUCINATION_PREVENTION.md) | Why arrays cause hallucinations + how to avoid them |

## Install

### ClawHub (OpenClaw)

```bash
clawhub install pagerunner-skill
```

### GitHub CLI `gh skill` (Claude Code, Copilot, Cursor, Gemini)

```bash
gh skill install Enreign/pagerunner-skill
```

### Claude Code plugin marketplace

```
/plugin marketplace add Enreign/pagerunner-skill
/plugin install pagerunner-skill@pagerunner-skill
```

### Manual

Clone this repo and point your agent runtime at `skills/pagerunner-skill/`.

## Requires

- **Pagerunner** (the MCP server) — `brew tap enreign/pagerunner && brew install pagerunner`, then `pagerunner init`
- Chrome with one or more configured profiles
- Any MCP-compatible agent runtime (Claude Code, Cursor, Windsurf, Cline, Codex, Gemini CLI, etc.)

See [pagerunner upstream](https://github.com/Enreign/pagerunner) for setup.

## License

MIT. See [LICENSE](./LICENSE).

## Author

Stas — issues and skill feedback welcome in [this repo](https://github.com/Enreign/pagerunner-skill/issues). For bugs in Pagerunner itself, see [upstream issues](https://github.com/Enreign/pagerunner/issues).
