---
name: sqlite-data
description: Use this skill when the user is working with SQLiteData, StructuredQueries, or GRDB in a Swift project — including defining database models, writing queries, fetching data reactively, performing inserts/updates/deletes, setting up migrations, or configuring CloudKit synchronization and sharing. Also use when the user asks about Swift database patterns, reactive data observation, or SQLite-backed persistence, even if they don't mention SQLiteData by name.
license: MIT
metadata:
  author: itisgriff
  version: "1.1"
---

Review and write SQLiteData code for correctness, modern API usage, and adherence to project conventions. Report only genuine problems - do not nitpick or invent issues.

If the user asks you to write or improve database code, make the changes directly instead of returning a findings report.

## Review Process

1. Read `references/setup.md` if the code configures a database, calls `prepareDependencies`, or accesses `@Dependency(\.defaultDatabase)`.
1. Read `references/tables.md` if the code defines `@Table` structs, uses `@Column`, `@Selection`, or `Draft` types.
1. Read `references/queries.md` if the code builds queries with `where`, `order`, `limit`, `join`, or aggregates.
1. Read `references/fetching.md` if the code uses `@FetchAll`, `@FetchOne`, `@Fetch`, `FetchKeyRequest`, or dynamic `load()` calls.
1. Read `references/writes.md` if the code performs inserts, updates, deletes, or seeds data.
1. Read `references/migrations.md` if the code uses `DatabaseMigrator` or `#sql` for schema changes.
1. Read `references/cloudkit.md` if the code uses `SyncEngine`, `SyncMetadata`, `CloudSharingView`, or any CloudKit sharing APIs.
1. Read `references/observable.md` if the code uses `@Observable` classes with fetch wrappers, or integrates with SwiftUI/UIKit.
1. Read `references/testing.md` if the code sets up preview or test databases, or uses `SQLiteDataTestSupport`.
1. Read `references/advanced-topics.md` if the code uses raw SQL execution, custom database functions, or `QueryCursor`.

Load only the references relevant to the code you are reviewing or writing.


## Gotchas

These are non-obvious mistakes the agent will make without explicit guidance:

- **UUID encoding:** Pass `UUID` directly as GRDB raw SQL arguments — never `.uuidString`. `@Table` encodes UUIDs as BLOB; `.uuidString` produces TEXT, causing silent lookup failures.
- **@ObservationIgnored required:** `@FetchAll`, `@FetchOne`, `@Fetch`, and `@Dependency` must be marked `@ObservationIgnored` inside `@Observable` classes. Without it, the observation systems conflict and data won't update.
- **@State @FetchAll is wrong:** Do not combine `@State` with `@FetchAll`/`@FetchOne` — they are already stateful. `@State @FetchAll var items` will break.
- **prepareDependencies called once:** Calling it more than once produces runtime warnings. Call it in `App.init()` and nowhere else.
- **Don't import GRDB or StructuredQueries separately:** SQLiteData re-exports them. Adding separate imports causes ambiguous symbol errors.
- **CloudKit sharing is root-only:** Only records with no foreign keys to other synced tables can be shared. Child records inherit sharing from their parent.
- **CloudKit requires UUID primary keys:** All synced tables must use UUID primary keys conforming to `IdentifierStringConvertible`.
- **Reserved CloudKit column names:** `recordName`, `zoneName`, `ownerName`, `share`, `parent`, `lastKnownServerRecord`, `userModificationTime`, `recordPrimaryKey`, `recordType`, `parentRecordPrimaryKey`, `parentRecordType`, `isShared` — using these will collide with `SyncMetadata`.
- **nonisolated on @Table:** Mark `@Table` structs `nonisolated` for Swift 6 strict concurrency compliance.
- **Observation lifecycle:** Tie fetch observation to view lifecycle with `.task { try? await $wrapper.load(query).task }`. Without `.task`, the subscription leaks.


## Output Format

Organize findings by file. For each issue:

1. State the file and relevant line(s).
2. Name the rule being violated.
3. Show a brief before/after code fix.

Skip files with no issues. End with a prioritized summary.

### Self-Check

Before finalizing your review or written code, verify:
- [ ] No `@FetchAll`/`@FetchOne`/`@Dependency` without `@ObservationIgnored` in `@Observable` classes
- [ ] No `.uuidString` passed to GRDB query arguments
- [ ] No separate `import GRDB` or `import StructuredQueries`
- [ ] `prepareDependencies` called exactly once
- [ ] CloudKit-synced tables use UUID primary keys
- [ ] All `.task` subscriptions use the `FetchSubscription.task` pattern


## References

- `references/setup.md` — database configuration, `prepareDependencies`, `defaultDatabase`
- `references/tables.md` — `@Table`, `@Column`, `@Selection`, `Draft`, foreign keys
- `references/queries.md` — query DSL: where, order, limit, joins, aggregates
- `references/fetching.md` — `@FetchAll`, `@FetchOne`, `@Fetch`, `FetchKeyRequest`, dynamic loading
- `references/writes.md` — insert, update, delete, seeding, transactions
- `references/migrations.md` — `DatabaseMigrator`, `#sql` macro, schema evolution
- `references/cloudkit.md` — `SyncEngine`, sharing, `SyncMetadata`, schema requirements
- `references/observable.md` — `@Observable` patterns, `@ObservationIgnored`, SwiftUI/UIKit
- `references/testing.md` — preview/test setup, `SQLiteDataTestSupport`, dependency injection
- `references/advanced-topics.md` — custom functions, raw SQL execution, `QueryCursor`
