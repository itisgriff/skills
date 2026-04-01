# Reactive Data Fetching

SQLiteData provides three property wrappers for observing database changes reactively: `@FetchAll` for collections, `@FetchOne` for single values, and `@Fetch` for custom multi-query transactions.

All three are backed by `SharedReader` from swift-sharing, conform to `DynamicProperty` for SwiftUI, and support Combine publishers.

## @FetchAll

Fetches and observes a collection of rows.

```swift
// Fetch all rows from a table:
@FetchAll var items: [Item]

// With a query:
@FetchAll(Item.order(by: \.name)) var items

// With animation (iOS 17+):
@FetchAll(Item.order { $0.name }, animation: .default) var items

// With a specific database:
@FetchAll(Item.all, database: myDB) var items
```

### Properties

- `wrappedValue: [Element]` — the fetched collection
- `projectedValue: Self` — access to `loadError`, `isLoading`, `publisher`, and `load()`
- `sharedReader: SharedReader<[Element]>` — the underlying shared reader
- `loadError: (any Error)?` — the last error from loading
- `isLoading: Bool` — whether the initial load is in progress
- `publisher: some Publisher<[Element], Never>` — Combine publisher for observation outside SwiftUI

## @FetchOne

Fetches and observes a single value or row.

```swift
// Count:
@FetchOne(Item.count()) var itemCount = 0

// Single optional row:
@FetchOne(Item.where { $0.id == someId }) var item: Item?

// First row from table:
@FetchOne var firstItem: Item?
```

Same API surface as `@FetchAll` (`loadError`, `isLoading`, `publisher`, `load()`).

## @Fetch with FetchKeyRequest

For complex fetches requiring multiple queries in a single transaction:

```swift
struct PlayersRequest: FetchKeyRequest {
    struct Value {
        let injuredCount: Int
        let players: [Player]
    }
    func fetch(_ db: Database) throws -> Value {
        try Value(
            injuredCount: Player.where(\.isInjured).fetchCount(db),
            players: Player.where { !$0.isInjured }.order(by: \.name).limit(10).fetchAll(db)
        )
    }
}

// Usage:
@Fetch(PlayersRequest()) var response = PlayersRequest.Value(injuredCount: 0, players: [])
```

The `FetchKeyRequest` protocol:

```swift
public protocol FetchKeyRequest<Value>: Hashable, Sendable {
    associatedtype Value
    func fetch(_ db: Database) throws -> Value
}
```

## Dynamic Loading

Reload with a different query at runtime using the projected value:

```swift
// Reload current query:
try await $items.load()

// Load a new query:
try await $items.load(Item.where { $0.isActive })

// Load with animation:
try await $items.load(Item.all, animation: .default)
```

## FetchSubscription and View Lifecycle

`load()` returns a `FetchSubscription`. Use its `task` property to tie observation lifetime to a SwiftUI view:

```swift
.task {
    try? await $items.load(Item.all).task
}
```

The `task` property suspends until cancelled (when the view disappears), then terminates the observation. You can also cancel manually:

```swift
let subscription = try await $items.load(Item.all)
// Later:
subscription.cancel()
```

## Dynamic Queries with .task(id:)

Use `.task(id:)` to reload when a dependency changes:

```swift
@State var searchText = ""
@FetchAll var items: [Item]

var body: some View {
    List(items) { item in ... }
        .searchable(text: $searchText)
        .task(id: searchText) {
            try? await $items.load(
                Item.where { $0.name.contains(searchText) },
                animation: .default
            ).task
        }
}
```

## Important: @State @FetchAll Caveat

Do **not** combine `@State` with `@FetchAll`/`@FetchOne`. They are already stateful property wrappers:

```swift
// BAD:
@State @FetchAll var items: [Item]

// GOOD:
@FetchAll var items: [Item]
```

## NotFound Error

`find()` and certain `@FetchOne` operations throw `NotFound` when no row matches:

```swift
public struct NotFound: Error {}
```

Handle it explicitly:

```swift
do {
    let item = try Item.find(db, key: someID)
} catch is NotFound {
    // Handle missing record
}
```
