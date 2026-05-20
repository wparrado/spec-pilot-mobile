# 03 — Domain Patterns

The domain layer has **zero external dependencies**. No framework imports. No networking. No UI. Pure business logic.

---

## Pattern #1 — Entity / Aggregate Root

An entity has an identity that persists across state changes.

```dart
// Flutter / Dart
class Order {
  final OrderId id;         // value object
  final List<OrderItem> items;
  final OrderStatus status;

  const Order({required this.id, required this.items, required this.status});

  // domain behavior lives here, not in services
  Result<Order, DomainError> addItem(OrderItem item) {
    if (status != OrderStatus.draft) {
      return Failure(DomainError.invalidOperation('Cannot add items to a non-draft order'));
    }
    return Success(Order(id: id, items: [...items, item], status: status));
  }
}
```

**Rules:**
- All fields are final / immutable.
- Mutations return a new instance (value semantics) or return `Result`.
- No framework annotations inside the entity.

---

## Pattern #2 — Value Object

A value object has no identity; equality is determined by its fields.

```swift
// Swift
struct Email: Equatable {
  let value: String

  static func create(_ raw: String) -> Result<Email, DomainError> {
    let trimmed = raw.trimmingCharacters(in: .whitespaces).lowercased()
    guard trimmed.contains("@"), trimmed.count >= 5 else {
      return .failure(.validationError("Invalid email: \(raw)"))
    }
    return .success(Email(value: trimmed))
  }
  private init(value: String) { self.value = value }
}
```

**Rules:**
- Private init — creation always goes through `create()` which validates.
- `create()` returns `Result<Self, DomainError>`, never throws.
- Comparable / Equatable by value fields only.

---

## Pattern #3 — Use Case / Interactor

One file per user story. Pure business orchestration.

```kotlin
// Kotlin
class GetOrderUseCase(
  private val repository: OrderRepository
) {
  suspend operator fun invoke(id: OrderId): Result<Order, DomainError> =
    repository.findById(id)
}
```

**Rules:**
- One public method (`execute`, `invoke`, or `call`).
- Dependencies injected via constructor — all are domain interfaces (ports).
- No framework code. No Hive, Room, CoreData, URLSession.
- Every use case has at least one unit test with an in-memory repository.

---

## Pattern #4 — Repository Port

Interface defined in the domain; implementation lives in the data layer.

```typescript
// TypeScript
export interface OrderRepository {
  findById(id: OrderId): Promise<Result<Order, DomainError>>;
  findAll(filter?: OrderFilter): Promise<Result<Order[], DomainError>>;
  save(order: Order): Promise<Result<void, DomainError>>;
  delete(id: OrderId): Promise<Result<void, DomainError>>;
}
```

**Rules:**
- All methods return `Result<T, DomainError>` (never throw).
- No framework-specific types in parameters or return types.
- An `InMemoryOrderRepository` implementing this interface lives in `test/` for fast unit tests.

---

## Pattern #5 — Domain Event

Signals that something important happened in the domain. Used for side-effect decoupling.

```dart
// Flutter / Dart
sealed class OrderEvent {
  const OrderEvent();
}

class OrderPlaced extends OrderEvent {
  final OrderId orderId;
  final DateTime occurredAt;
  const OrderPlaced({required this.orderId, required this.occurredAt});
}

class OrderCancelled extends OrderEvent {
  final OrderId orderId;
  final String reason;
  final DateTime occurredAt;
  const OrderCancelled({required this.orderId, required this.reason, required this.occurredAt});
}
```

**Rules:**
- Events are immutable value types.
- Events are raised by entities / use cases, not by repositories.
- Handlers live in the application or infrastructure layer — never in the domain.
