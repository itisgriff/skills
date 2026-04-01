# Observable Model Patterns

## @Observable Classes

When using `@FetchAll`, `@FetchOne`, `@Fetch`, or `@Dependency` inside an `@Observable` class, you **must** mark them with `@ObservationIgnored`. These property wrappers manage their own observation and conflict with the `@Observable` macro's observation tracking.

```swift
@Observable
@MainActor
class ItemModel {
    @ObservationIgnored
    @FetchAll(Item.order { $0.name }) var items

    @ObservationIgnored
    @FetchOne(Item.count()) var count = 0

    @ObservationIgnored
    @Dependency(\.defaultDatabase) var database
}
```

## SwiftUI View Integration

In SwiftUI views, use the property wrappers directly — no `@ObservationIgnored` needed since views are structs:

```swift
struct ItemListView: View {
    @FetchAll(Item.order { $0.name }) var items

    var body: some View {
        List(items) { item in
            Text(item.name)
        }
    }
}
```

## View Lifecycle with .task

Tie observation to view lifecycle:

```swift
struct ItemListView: View {
    @FetchAll var items: [Item]

    var body: some View {
        List(items) { item in
            Text(item.name)
        }
        .task {
            try? await $items.load(Item.order { $0.name }).task
        }
    }
}
```

The `.task` modifier cancels when the view disappears, which cancels the `FetchSubscription`, terminating the observation.

## Dynamic Queries in Views

Use `.task(id:)` to reload when inputs change:

```swift
struct SearchView: View {
    @State var searchText = ""
    @FetchAll var items: [Item]

    var body: some View {
        List(items) { item in
            Text(item.name)
        }
        .searchable(text: $searchText)
        .task(id: searchText) {
            try? await $items.load(
                Item.where { $0.name.contains(searchText) },
                animation: .default
            ).task
        }
    }
}
```

## Animation Support

Pass `animation:` to initializers or `load()` for animated transitions (iOS 17+):

```swift
// In initializer:
@FetchAll(Item.all, animation: .default) var items

// In dynamic load:
try? await $items.load(Item.all, animation: .default).task
```

## UIKit Integration

Use the Combine `publisher` for UIKit view controllers:

```swift
class ItemViewController: UIViewController {
    @FetchAll(Item.order { $0.name }) var items

    override func viewDidLoad() {
        super.viewDidLoad()
        $items.publisher
            .sink { [weak self] items in
                self?.updateUI(with: items)
            }
            .store(in: &cancellables)
    }
}
```

Or use the async `observe` pattern:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    Task {
        for await items in $items.sharedReader.values {
            updateUI(with: items)
        }
    }
}
```

## Combining Multiple Fetches

Group related fetches in a single model:

```swift
@Observable
@MainActor
class DashboardModel {
    @ObservationIgnored
    @FetchAll(Item.where { !$0.isComplete }.order { $0.name }) var activeItems

    @ObservationIgnored
    @FetchOne(Item.count()) var totalCount = 0

    @ObservationIgnored
    @FetchOne(Item.where { $0.isComplete }.count()) var completedCount = 0

    @ObservationIgnored
    @Dependency(\.defaultDatabase) var database
}
```

Or use `@Fetch` with `FetchKeyRequest` for a single transaction:

```swift
struct DashboardRequest: FetchKeyRequest {
    struct Value {
        let activeItems: [Item]
        let totalCount: Int
        let completedCount: Int
    }
    func fetch(_ db: Database) throws -> Value {
        try Value(
            activeItems: Item.where { !$0.isComplete }.order(by: \.name).fetchAll(db),
            totalCount: Item.fetchCount(db),
            completedCount: Item.where { $0.isComplete }.fetchCount(db)
        )
    }
}
```
