# Query DSL

SQLiteData uses the StructuredQueries DSL for type-safe query construction. Prefer the DSL over raw SQL for compile-time safety.

## Selecting All Rows

```swift
// All rows from a table:
Item.all

// Equivalent shorthand — just use the type with @FetchAll:
@FetchAll var items: [Item]
```

## Where Clauses

Use closure-based key-path syntax:

```swift
Item.where { $0.isComplete }
Item.where { $0.name == "Groceries" }
Item.where { !$0.isComplete }
Item.where { $0.name.contains("shop") }
Item.where { $0.id.in([id1, id2, id3]) }
```

Combine conditions:

```swift
Item.where { $0.isComplete && $0.name.contains("shop") }
Item.where { $0.isComplete || $0.name == "Urgent" }
```

Key-path shorthand for simple equality:

```swift
Item.where(\.isComplete)
Player.where(\.isInjured)
```

## Ordering

```swift
// Ascending (default):
Item.order(by: \.name)

// Descending:
Item.order { $0.name.desc() }

// Multiple columns:
Item.order { $0.isComplete }.order { $0.name }
```

## Limit and Offset

```swift
Item.limit(10)
Item.limit(10).offset(20)
```

## Joins

```swift
// Left join:
RemindersList
    .leftJoin(SyncMetadata.all) { $0.syncMetadataID.eq($1.id) }

// Inner join:
Item.join(Category.all) { $0.categoryID.eq($1.id) }
```

## Aggregates

```swift
// Count:
Item.count()
Item.where { $0.isComplete }.fetchCount(db)

// Other aggregates in selections:
Player.select { $0.score.sum() }
Player.select { $0.score.min() }
Player.select { $0.score.max() }
Player.select { $0.score.avg() }
```

## Grouping

```swift
Item.group(by: \.categoryID)
    .select { ($0.categoryID, $0.id.count()) }
```

## Column Expressions and Operators

```swift
// Equality:
$0.id.eq(someID)
$0.name == "value"

// Inequality:
$0.score > 100
$0.score >= 50

// IN:
$0.id.in([id1, id2])

// NULL checks:
$0.deletedAt.isNull()
$0.deletedAt.isNotNull()

// String operations:
$0.name.contains("search")
$0.name.like("%pattern%")
```

## The #sql Macro

Use `#sql` for raw SQL when the DSL cannot express the query. Primarily used in migrations:

```swift
try #sql("""
    CREATE TABLE "items" (
        "id" TEXT PRIMARY KEY NOT NULL DEFAULT (uuid()),
        "name" TEXT NOT NULL
    ) STRICT
""").execute(db)
```

For raw queries, prefer the DSL. If you must use `#sql`, pass arguments safely — never interpolate values directly.
