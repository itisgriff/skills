# Skills

A collection of agent skills following the [agentskills.io](https://agentskills.io) open specification. Works with Claude Code, OpenAI Codex CLI, Cursor, Gemini CLI, and any conforming agent platform.

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [sqlite-data](./sqlite-data) | SQLiteData — property wrappers, StructuredQueries DSL, GRDB, CloudKit sync | `npx skills add itisgriff/skills --skill sqlite-data` |

## Installation

```bash
npx skills add itisgriff/skills --skill <skill-name>
```

## Adding Skills

Each skill is a self-contained directory with a `SKILL.md` and optional `references/`, `agents/`, and `assets/` subdirectories:

```
skill-name/
├── SKILL.md              # Required — frontmatter + instructions
├── references/           # Optional — detailed reference docs
├── agents/
│   └── openai.yaml       # Optional — Codex CLI metadata
└── assets/               # Optional — icons, templates
```

See the [agentskills.io specification](https://agentskills.io/specification) for the full format.

## License

MIT
