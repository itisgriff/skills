# sqlite-data

Agent skill for [Point-Free's SQLiteData](https://github.com/pointfreeco/sqlite-data) — a fast, lightweight replacement for SwiftData powered by raw SQL via GRDB and StructuredQueries, with CloudKit synchronization and sharing support.

## What it does

Reviews, writes, and guides SQLiteData code covering the full stack:

- **Database setup** — `prepareDependencies`, `defaultDatabase`, dependency injection
- **Model definitions** — `@Table`, `@Column`, `@Selection`, `Draft` types
- **Query DSL** — where, order, limit, joins, aggregates via StructuredQueries
- **Reactive fetching** — `@FetchAll`, `@FetchOne`, `@Fetch`, dynamic queries
- **Data mutation** — insert, update, delete, seeding, transactions
- **Migrations** — `DatabaseMigrator`, `#sql` macro, schema evolution
- **CloudKit sync** — `SyncEngine`, sharing, `SyncMetadata`, schema requirements
- **Observable patterns** — `@Observable` integration, `@ObservationIgnored`, SwiftUI/UIKit
- **Testing** — preview/test database setup, `SQLiteDataTestSupport`
- **Advanced** — custom database functions, raw SQL execution, `QueryCursor`

## Installation

### Claude Code

```bash
/install itisgriff/skills --skill sqlite-data
```

### OpenAI Codex CLI

Copy or symlink `sqlite-data/` into `.agents/skills/` in your project or `~/.agents/skills/` globally.

### Universal (via skills.sh)

```bash
npx skills add itisgriff/skills --skill sqlite-data
```

## Structure

```
sqlite-data/
├── SKILL.md                     # Main skill — review process, gotchas, self-check
├── agents/
│   └── openai.yaml              # Codex CLI metadata
└── references/
    ├── setup.md                 # Database configuration
    ├── tables.md                # Model definitions
    ├── queries.md               # Query DSL
    ├── fetching.md              # Reactive data fetching
    ├── writes.md                # Data mutation
    ├── migrations.md            # Schema migrations
    ├── cloudkit.md              # CloudKit synchronization
    ├── observable.md            # Observable model patterns
    ├── testing.md               # Testing and previews
    └── advanced-topics.md       # Advanced patterns
```

## Compatibility

Built for SQLiteData **1.6.1**. Follows the [agentskills.io](https://agentskills.io) open specification — works with Claude Code, Codex CLI, Cursor, Gemini CLI, and any conforming agent platform.

## License

MIT
