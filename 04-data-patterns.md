# 04 — Data Patterns

The data layer implements the repository ports defined in the domain. It knows about networking, databases, and caching. The domain knows nothing about the data layer.

---

## Pattern #14 — Remote Data Source

Wraps the HTTP client. Maps API responses to domain entities or DTOs.

```kotlin
// Kotlin
class RemoteOrderDataSource(private val api: OrderApi) {
  suspend fun fetchById(id: String): Result<OrderDTO, NetworkError> = runCatching {
    api.getOrder(id)
  }.fold(
    onSuccess = { Result.success(it) },
    onFailure = { Result.failure(it.toNetworkError()) }
  )
}
```

**Rules:**
- Returns `Result<DTO, NetworkError>` — never throws.
- Maps HTTP status codes to typed `NetworkError` variants (timeout, unauthorized, not_found, server_error).
- No domain logic here — DTOs are raw data transfer objects, not domain entities.

---

## Pattern #15 — Local Data Source

Wraps the local persistence engine (Room, CoreData, SwiftData, Drift, MMKV).

```swift
// Swift
class LocalOrderDataSource {
  private let context: NSManagedObjectContext

  func getById(_ id: String) -> Order? {
    let request = OrderEntity.fetchRequest()
    request.predicate = NSPredicate(format: "id == %@", id)
    return try? context.fetch(request).first.map { $0.toDomain() }
  }

  func upsert(_ order: Order) throws {
    // insert or update by id
  }

  func lastSyncAt() -> Date? {
    UserDefaults.standard.object(forKey: "order_last_sync") as? Date
  }
}
```

**Rules:**
- Returns plain domain entities or nil — never throws across the API boundary.
- Includes `lastSyncAt()` for offline-first freshness checks.
- All schema migrations are versioned and tested.

---

## Pattern #16 — Repository Adapter

Implements the domain `Repository` port. Orchestrates remote and local sources.

```dart
// Flutter / Dart
class OrderRepositoryAdapter implements OrderRepository {
  final RemoteOrderDataSource _remote;
  final LocalOrderDataSource _local;

  @override
  Future<Result<Order, DomainError>> findById(OrderId id) async {
    // Offline-first: serve stale data immediately
    final cached = await _local.getById(id.value);
    if (cached != null) return Success(cached);

    final result = await _remote.fetchById(id.value);
    return result.fold(
      onSuccess: (dto) async {
        final entity = dto.toDomain();
        await _local.upsert(entity);
        return Success(entity);
      },
      onFailure: (err) => Failure(err.toDomainError()),
    );
  }
}
```

**Rules:**
- Implements a domain port — no direct UI or ViewModel knowledge.
- Always reads from local first (offline-first).
- Maps `NetworkError` → `DomainError` before returning.

---

## Pattern #17 — Offline-First

Always read from local storage first; sync remote in the background.

```kotlin
// Kotlin — background sync pattern
class OrderRepositoryAdapter(...) : OrderRepository {
  override suspend fun save(order: Order): Result<Unit, DomainError> {
    // 1. Optimistic local write — instant for user
    local.upsert(order)

    // 2. Attempt remote sync
    val result = remote.createOrUpdate(order.toDTO())
    if (result.isFailure) {
      // 3. Queue for retry if offline
      syncQueue.enqueue(SyncOperation.Upsert(order))
    }
    return Result.success(Unit)
  }
}
```

**Rules:**
- Local write always succeeds synchronously (from user's perspective).
- Remote failure triggers a retry queue, not an error to the user.
- Sync queue items are persisted across app restarts.

---

## Pattern #18 — Cache-Aside

Read through cache; populate on miss; invalidate on TTL or mutation.

```typescript
// TypeScript
class CachedProductRepository implements ProductRepository {
  private cache = new Map<string, { data: Product; expiresAt: number }>();
  private readonly TTL_MS = 5 * 60 * 1000; // 5 minutes

  async findById(id: ProductId): Promise<Result<Product, DomainError>> {
    const entry = this.cache.get(id.value);
    if (entry && entry.expiresAt > Date.now()) {
      return ok(entry.data);
    }
    const result = await this.remote.fetchById(id.value);
    if (result.ok) {
      this.cache.set(id.value, { data: result.value, expiresAt: Date.now() + this.TTL_MS });
    }
    return result;
  }
}
```

**Rules:**
- Cache key uniquely identifies the resource.
- TTL is configurable, not hardcoded deep in logic.
- Invalidated on any `save()` or `delete()` for the same key.
