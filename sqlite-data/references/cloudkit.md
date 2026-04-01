# CloudKit Synchronization

SQLiteData provides bidirectional CloudKit sync via `SyncEngine`. Requires iOS 17+ / macOS 14+.

## SyncEngine Setup

Configure in `prepareDependencies` alongside the database:

```swift
@main
struct MyApp: App {
    init() {
        prepareDependencies {
            $0.defaultDatabase = try! appDatabase()
            $0.defaultSyncEngine = try! SyncEngine(
                for: $0.defaultDatabase,
                tables: Item.self, Category.self,
                privateTables: Secret.self
            )
        }
    }
}
```

- `tables:` — tables synced to both private and shared CloudKit zones
- `privateTables:` — tables synced only to the private zone (never shared)

## Accessing the Sync Engine

```swift
@Dependency(\.defaultSyncEngine) var syncEngine
```

Inside `@Observable` classes:

```swift
@ObservationIgnored
@Dependency(\.defaultSyncEngine) var syncEngine
```

## Starting and Stopping

```swift
await syncEngine.start()   // Begin sync
syncEngine.stop()           // Stop sync
syncEngine.isRunning        // Observable Bool
```

By default, `startImmediately` is `nil` — the engine starts automatically in live apps but not in tests/previews.

## Syncing Changes

```swift
// Fetch remote changes:
try await syncEngine.fetchChanges()

// Send local changes:
try await syncEngine.sendChanges()

// Both (fetch then send):
try await syncEngine.syncChanges()
```

Observable state:

```swift
syncEngine.isFetchingChanges  // Bool
syncEngine.isSendingChanges   // Bool
syncEngine.isSynchronizing    // Bool (either fetching or sending)
```

## Deleting Local Data

```swift
try await syncEngine.deleteLocalData()
```

Removes all locally synced data. Used by the default `SyncEngineDelegate` on account sign-out or switch.

## SyncEngineDelegate

Handle account changes:

```swift
public protocol SyncEngineDelegate: AnyObject, Sendable {
    func syncEngine(
        _ syncEngine: SyncEngine,
        accountChanged changeType: CKSyncEngine.Event.AccountChange.ChangeType
    ) async
}
```

Default implementation calls `deleteLocalData()` on `.signOut` / `.switchAccounts`.

## Schema Requirements for CloudKit

- **UUID primary keys required.** All synced tables must use UUID primary keys that conform to `IdentifierStringConvertible`.
- **No unique constraints** (other than primary key) — CloudKit cannot enforce them.
- **Foreign keys must reference synced tables** or use `privateTables` for private-only data.
- **Avoid reserved column names:** `recordName`, `zoneName`, `ownerName`, `share`, `parent`, `lastKnownServerRecord`, `userModificationTime`, `recordPrimaryKey`, `recordType`, `parentRecordPrimaryKey`, `parentRecordType`, `isShared`.

## IdentifierStringConvertible

Required for primary keys of CloudKit-synced tables:

```swift
public protocol IdentifierStringConvertible {
    init?(rawIdentifier: String)
    var rawIdentifier: String { get }
}
```

Built-in conformances: `String`, `Substring`, `UUID`, and `Tagged` (with `SQLiteDataTagged` trait).

## SyncMetadata

A system table tracking CloudKit sync state for each record:

```swift
@Table("sqlitedata_icloud_metadata")
public struct SyncMetadata: Identifiable {
    public let id: ID  // (recordPrimaryKey, recordType)
    public let zoneName: String
    public let ownerName: String
    public let recordName: String  // generated: "primaryKey:tableName"
    public let parentRecordID: ParentID?
    public let lastKnownServerRecord: CKRecord?
    public let share: CKShare?
    public var isShared: Bool
}
```

Join with your tables using `syncMetadataID`:

```swift
Item.leftJoin(SyncMetadata.all) { $0.syncMetadataID.eq($1.id) }
```

## Sharing

Only **root records** (records with no foreign key pointing to another synced table) can be shared. Child records inherit sharing from their parent.

### Creating a Share

```swift
let sharedRecord = try await syncEngine.share(record: item) { share in
    share[CKShare.SystemFieldKey.title] = "My Shared Item"
}
```

### Presenting the Sharing UI

```swift
CloudSharingView(
    sharedRecord: sharedRecord,
    availablePermissions: [.allowReadWrite, .allowPrivate]
)
```

### Accepting a Share

```swift
try await syncEngine.acceptShare(metadata: shareMetadata)
```

### Removing a Share

```swift
try await syncEngine.unshare(record: item)
```

## SharedRecord

Wraps a `CKShare` for the sharing UI:

```swift
public struct SharedRecord: Hashable, Identifiable, Sendable {
    public let share: CKShare
    public var id: CKRecord.ID { share.recordID }
}
```

## Primary Key Migration

Convert integer primary keys to UUID for CloudKit compatibility:

```swift
try SyncEngine.migratePrimaryKeys(
    db,
    tables: Item.self, Category.self,
    dropUniqueConstraints: false
)
```
