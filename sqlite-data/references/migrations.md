# Migrations

Use GRDB's `DatabaseMigrator` to manage schema changes. Register migrations in order — they run sequentially and each migration runs at most once.

## Basic Migration Setup

```swift
var migrator = DatabaseMigrator()

migrator.registerMigration("Create items") { db in
    try #sql("""
        CREATE TABLE "items" (
            "id" TEXT PRIMARY KEY NOT NULL DEFAULT (uuid()),
            "name" TEXT NOT NULL,
            "isComplete" INTEGER NOT NULL DEFAULT 0
        ) STRICT
    """).execute(db)
}

migrator.registerMigration("Create attendees") { db in
    try #sql("""
        CREATE TABLE "attendees" (
            "id" TEXT PRIMARY KEY NOT NULL DEFAULT (uuid()),
            "name" TEXT NOT NULL,
            "itemID" TEXT NOT NULL REFERENCES "items"("id") ON DELETE CASCADE
        ) STRICT
    """).execute(db)
}

try migrator.migrate(database)
```

## The #sql Macro

Use `#sql` for raw SQL statements in migrations. It provides compile-time syntax checking:

```swift
try #sql("""
    ALTER TABLE "items" ADD COLUMN "dueDate" TEXT
""").execute(db)
```

## Schema Conventions

- Always use `STRICT` mode for type safety
- Use `TEXT` for String and UUID columns (the `uuid()` SQL function returns TEXT; `@Table` handles encoding transparently)
- Use `INTEGER` for Int and Bool columns
- Use `REAL` for Double/Float columns
- Use `BLOB` for Data columns
- Use `NOT NULL` unless the column is genuinely optional
- Use `DEFAULT (uuid())` for auto-generated UUID primary keys
- Use `DEFAULT 0` or `DEFAULT 1` for Boolean columns
- Define foreign keys with `REFERENCES` and `ON DELETE CASCADE` (or appropriate action)

## Adding Columns

```swift
migrator.registerMigration("Add due date to items") { db in
    try #sql("""
        ALTER TABLE "items" ADD COLUMN "dueDate" TEXT
    """).execute(db)
}
```

## Creating Indexes

```swift
migrator.registerMigration("Index items by name") { db in
    try #sql("""
        CREATE INDEX "items_name" ON "items"("name")
    """).execute(db)
}
```

## Migration Best Practices

- Never modify a migration that has already shipped — always add a new one.
- Give migrations descriptive names that explain what they do.
- Keep migrations small and focused on one schema change.
- Test migrations by running them against a fresh database and verifying the schema.
- For CloudKit-synced tables, UUIDs are required as primary keys. Use `SyncEngine.migratePrimaryKeys` if converting from integer keys.

## Migrating Integer Primary Keys to UUID

If you need to convert integer primary keys to UUIDs (e.g., for CloudKit compatibility), use `SyncEngine.migratePrimaryKeys`:

```swift
migrator.registerMigration("Migrate to UUID primary keys") { db in
    try SyncEngine.migratePrimaryKeys(
        db,
        tables: Item.self, Attendee.self
    )
}
```

This automates the 4-step process: creating a new UUID column, populating it, updating foreign keys, and swapping the primary key.

For manual migration, the process involves:
1. Add a new UUID column
2. Populate it with generated UUIDs
3. Update all foreign key references
4. Drop the old column and rename the new one
