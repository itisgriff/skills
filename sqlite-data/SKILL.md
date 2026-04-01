---
name: sqlite-data
description: Reviews, writes, and guides SQLiteData code covering the full stack — property wrappers, StructuredQueries DSL, GRDB integration, and CloudKit sync. Use when reading, writing, or reviewing projects that use SQLiteData.
license: MIT
metadata:
  author: Custom
  version: "1.0"
---

Review and write SQLiteData code for correctness, modern API usage, and adherence to project conventions. Report only genuine problems - do not nitpick or invent issues.

Review process:

1. Validate database setup and configuration using `references/setup.md`.
1. Check model definitions and table macros using `references/tables.md`.
1. Validate query construction and DSL usage using `references/queries.md`.
1. Check reactive data fetching patterns using `references/fetching.md`.
1. Validate data mutation operations using `references/writes.md`.
1. Ensure migrations are correct and complete using `references/migrations.md`.
1. If CloudKit sync is used, validate sync engine setup and sharing using `references/cloudkit.md`.
1. Check Observable model patterns and SwiftUI integration using `references/observable.md`.
1. Validate testing and preview setup using `references/testing.md`.
1. Check for advanced patterns and raw SQL usage using `references/advanced-topics.md`.

If doing a partial review, load only the relevant reference files.


## Core Instructions

- SQLiteData re-exports `Dependencies`, `StructuredQueriesSQLite`, and key GRDB types — do not import them separately.
- Target Swift 6.2 or later, using modern Swift concurrency. Mark `@Table` structs as `nonisolated` for Swift 6 compliance.
- Always use `STRICT` mode when creating tables.
- Pass `UUID` directly as GRDB raw SQL arguments — never use `.uuidString` (BLOB vs TEXT encoding mismatch).
- Prefer StructuredQueries DSL over raw SQL for type safety. Use `#sql` macro only in migrations or when the DSL cannot express the query.
- Always mark `@FetchAll`, `@FetchOne`, `@Fetch`, and `@Dependency` with `@ObservationIgnored` inside `@Observable` classes.
- Tie observation lifetime to view lifecycle using `.task { try? await $wrapper.load(query).task }`.
- CloudKit-synced tables require UUID primary keys with `IdentifierStringConvertible` conformance.
- Only root records (no foreign keys pointing elsewhere) can be shared via CloudKit. Child records inherit sharing from their parent.
- `prepareDependencies` should be called exactly once, as early as possible (typically in the `App.init()`).


## Output Format

Organize findings by file. For each issue:

1. State the file and relevant line(s).
2. Name the rule being violated.
3. Show a brief before/after code fix.

Skip files with no issues. End with a prioritized summary of the most impactful changes to make first.

If the user asks you to write or improve database code, follow the same rules above but make the changes directly instead of returning a findings report.

Example output:

### ItemModel.swift

**Line 8: Missing `@ObservationIgnored` on `@FetchAll` inside `@Observable` class.**

```swift
// Before
@Observable
class ItemModel {
    @FetchAll(Item.order { $0.name }) var items

// After
@Observable
class ItemModel {
    @ObservationIgnored
    @FetchAll(Item.order { $0.name }) var items
```

**Line 22: Using `.uuidString` in raw SQL argument — pass UUID directly.**

```swift
// Before
try Item.find(db, key: itemID.uuidString)

// After
try Item.find(db, key: itemID)
```

### Summary

1. **Data flow (high):** Missing `@ObservationIgnored` on line 8 will cause observation conflicts.
2. **Query correctness (high):** UUID encoding mismatch on line 22 will fail to match BLOB-stored UUIDs.

End of example.


## References

- `references/setup.md` - Database configuration, `prepareDependencies`, `defaultDatabase`, dependency access.
- `references/tables.md` - `@Table` macro, `@Column`, `@Selection`, model definitions, Draft types.
- `references/queries.md` - Query DSL: where, order, limit, joins, aggregates, column expressions.
- `references/fetching.md` - `@FetchAll`, `@FetchOne`, `@Fetch`, `FetchKeyRequest`, `FetchSubscription`, dynamic loading.
- `references/writes.md` - Insert, update, delete operations, seeding, transactions.
- `references/migrations.md` - `DatabaseMigrator`, `#sql` macro, schema creation and evolution.
- `references/cloudkit.md` - `SyncEngine`, CloudKit sync, sharing, `SyncMetadata`, schema requirements.
- `references/observable.md` - `@Observable` patterns, `@ObservationIgnored`, SwiftUI/UIKit integration.
- `references/testing.md` - Preview and test database setup, `SQLiteDataTestSupport`, dependency injection.
- `references/advanced-topics.md` - Custom database functions, raw SQL execution, `QueryCursor`, static table methods.
