# Skills

A collection of agent skills following the [agentskills.io](https://agentskills.io) open specification. Works with Claude Code, OpenAI Codex CLI, Cursor, Gemini CLI, and any conforming agent platform.

## Available Skills

| Skill | Description |
|-------|-------------|
| [sqlite-data](./sqlite-data) | Reviews, writes, and guides SQLiteData code — property wrappers, StructuredQueries DSL, GRDB integration, and CloudKit sync |

## Installation

### Claude Code

```bash
/install itisgriff/skills --skill <skill-name>
```

### OpenAI Codex CLI

Copy or symlink the skill directory into `.agents/skills/` in your project or `~/.agents/skills/` globally.

### Universal (via skills.sh)

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
