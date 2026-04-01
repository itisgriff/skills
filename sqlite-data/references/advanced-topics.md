# Advanced Topics

## Custom Database Functions

Register custom SQL functions for use in queries and triggers.

### Scalar Functions

Return a single value:

```swift
db.add(function: myScalarFunction)
```

Where `myScalarFunction` conforms to `ScalarDatabaseFunction`.

### Aggregate Functions

Compute values across multiple rows:

```swift
db.add(function: myAggregateFunction)
```

Where `myAggregateFunction` conforms to `AggregateDatabaseFunction`.

### Removing Functions

```swift
db.remove(function: myFunction)
```

## Raw SQL Execution

SQLiteData extends StructuredQueries' `Statement` type with GRDB execution methods. Use these when the query DSL cannot express what you need.

### Execute (no return value)

For INSERT, UPDATE, DELETE, and DDL statements:

```swift
try statement.execute(db)
```

### Fetch All Rows

```swift
let items: [Item.QueryOutput] = try statement.fetchAll(db)
```

### Fetch One Row

```swift
let item: Item.QueryOutput? = try statement.fetchOne(db)
```

### Fetch Cursor (lazy iteration)

For large result sets, use a cursor to avoid loading everything into memory:

```swift
let cursor: QueryCursor<Item.QueryOutput> = try statement.fetchCursor(db)
while let item = try cursor.next() {
    // Process item lazily
}
```

`QueryCursor` conforms to GRDB's `DatabaseCursor` protocol.

### Fetch Count

On `SelectStatement`:

```swift
let count: Int = try statement.fetchCount(db)
```

### Find by Primary Key

On tables with primary keys:

```swift
let item = try Item.find(db, key: someUUID)  // Throws NotFound if missing
```

## Static Table Convenience Methods

`Table` and `PrimaryKeyedTable` provide static methods that bypass the query DSL for simple operations:

```swift
// Fetch all rows:
let items = try Item.fetchAll(db)

// Fetch first row:
let item = try Item.fetchOne(db)

// Count all rows:
let count = try Item.fetchCount(db)

// Lazy cursor:
let cursor = try Item.fetchCursor(db)

// Find by primary key (PrimaryKeyedTable only):
let item = try Item.find(db, key: someUUID)
```

These are equivalent to `Item.all.fetchAll(db)`, etc., but more concise for simple cases.

## SyncEngine.isSynchronizing (Static)

A static `@DatabaseFunction` that returns whether the sync engine is currently processing changes. Useful in database triggers:

```swift
@DatabaseFunction("sqlitedata_icloud_syncEngineIsSynchronizingChanges")
public static var isSynchronizing: Bool
```

## Attaching the Metadatabase

For CloudKit apps that need to access sync metadata from a different database connection:

```swift
try db.attachMetadatabase(containerIdentifier: "iCloud.com.example.myapp")
```
