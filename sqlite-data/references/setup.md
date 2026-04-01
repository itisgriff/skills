# Database Setup and Configuration

## Creating the Database

Use the `defaultDatabase(path:configuration:)` helper to create a database connection. It returns a context-sensitive `DatabaseWriter`:
- **Live**: `DatabasePool` at the specified path (or Application Support directory if `nil`)
- **Preview/Test**: `DatabasePool` at a temporary file with a unique UUID name

```swift
public func defaultDatabase(
  path: String? = nil,
  configuration: Configuration = Configuration()
) throws -> any DatabaseWriter
```

## App Entry Point Setup

Call `prepareDependencies` exactly once, as early as possible — typically in the `App.init()`. Calling it multiple times produces runtime warnings.

```swift
@main
struct MyApp: App {
    init() {
        prepareDependencies {
            $0.defaultDatabase = try! appDatabase()
        }
    }
    var body: some Scene {
        WindowGroup { ContentView() }
    }
}
```

## The appDatabase Function

Create a dedicated function that configures and returns the database. This is where you set up configuration, run migrations, and optionally seed data.

```swift
func appDatabase() throws -> any DatabaseWriter {
    let database = try defaultDatabase(
        configuration: {
            var config = Configuration()
            // Optional: configure tracing, foreign keys, etc.
            return config
        }()
    )

    var migrator = DatabaseMigrator()
    // Register migrations here...
    try migrator.migrate(database)

    return database
}
```

## Accessing the Database

Use the `@Dependency` property wrapper to access the database anywhere:

```swift
@Dependency(\.defaultDatabase) var database
```

Inside `@Observable` classes, mark it with `@ObservationIgnored`:

```swift
@Observable
class MyModel {
    @ObservationIgnored
    @Dependency(\.defaultDatabase) var database
}
```

## Re-exported Types

SQLiteData re-exports these — do not import GRDB or StructuredQueries separately:
- From Dependencies: `@Dependency`, `prepareDependencies`
- From StructuredQueriesSQLite: `@Table`, `@Column`, `@Selection`, query DSL
- From GRDB: `Configuration`, `Database`, `DatabaseError`, `DatabaseMigrator`, `DatabasePool`, `DatabaseQueue`, `DatabaseReader`, `DatabaseWriter`, `ValueObservationScheduler`

## Adding to an Existing GRDB App

If migrating from an existing GRDB app that uses `PersistableRecord`/`FetchableRecord`:
- Replace record types with `@Table` structs
- Replace manual column enums with the generated `TableColumns`
- Replace `ValueObservation` with `@FetchAll`/`@FetchOne`
- Be aware that `@Table` encodes `UUID` as a BLOB, while GRDB's `PersistableRecord` encodes it as TEXT. If mixing both, pass `UUID` directly (never `.uuidString`) in raw SQL.
