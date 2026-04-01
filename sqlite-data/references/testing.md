# Testing and Previews

SQLiteData uses swift-dependencies for dependency injection, making it straightforward to configure databases for tests and previews.

## Preview Setup

Use `prepareDependencies` with `let _` binding to configure the preview database:

```swift
#Preview {
    let _ = prepareDependencies {
        $0.defaultDatabase = try! previewDatabase()
    }
    ItemListView()
}
```

Create a dedicated preview database function that seeds sample data:

```swift
func previewDatabase() throws -> any DatabaseWriter {
    let database = try defaultDatabase()

    var migrator = DatabaseMigrator()
    // Register all migrations...
    try migrator.migrate(database)

    try database.write { db in
        try db.seed {
            Item(id: UUID(), name: "Preview Item 1", isComplete: false)
            Item(id: UUID(), name: "Preview Item 2", isComplete: true)
        }
    }

    return database
}
```

## Test Database Setup

In tests, `defaultDatabase()` automatically returns a temporary `DatabasePool`. Configure it in your test setup:

```swift
@Test func itemCreation() throws {
    let database = try defaultDatabase()
    var migrator = DatabaseMigrator()
    // Register migrations...
    try migrator.migrate(database)

    try database.write { db in
        try Item.insert { Item.Draft(name: "Test") }.execute(db)
    }

    let items = try database.read { db in
        try Item.fetchAll(db)
    }
    #expect(items.count == 1)
    #expect(items[0].name == "Test")
}
```

## Dependency Injection in Tests

Override dependencies using `prepareDependencies` or `withDependencies`:

```swift
@Test func modelFetchesItems() async throws {
    try await withDependencies {
        $0.defaultDatabase = try testDatabase(seeded: true)
    } operation: {
        let model = ItemModel()
        try await $model.items.load()
        #expect(model.items.isEmpty == false)
    }
}
```

## SQLiteDataTestSupport

The `SQLiteDataTestSupport` module provides `assertQuery` for snapshot-testing queries:

```swift
import SQLiteDataTestSupport

assertQuery(
    Item.where { $0.isComplete }.order { $0.name },
    sql: {
        """
        SELECT * FROM "items" WHERE "isComplete" ORDER BY "name"
        """
    },
    results: {
        """
        | id  | name     | isComplete |
        | ... | "Task A" | true       |
        """
    }
)
```

This tests both the generated SQL and the decoded results in table format.

## Isolation Between Tests

Each test should use its own database instance. Since `defaultDatabase()` creates a temporary file with a unique UUID in test contexts, parallel tests are naturally isolated:

```swift
// Each call creates a fresh, isolated database:
let db1 = try defaultDatabase()  // Unique temp file
let db2 = try defaultDatabase()  // Different unique temp file
```
