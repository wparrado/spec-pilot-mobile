# 08 — Testing Strategy

## Test pyramid for mobile

```
         /\
        /E2E\          Detox / XCTest UI / Espresso  (5%)
       /------\
      / Widget \       Flutter widget tests / XCTest / Compose UI Tests  (15%)
     /----------\
    / Integration\     Repository adapters, interceptors, local sources  (20%)
   /--------------\
  /    Unit         \  Domain logic, use cases, ViewModels  (60%)
 /------------------\
```

## Coverage thresholds

| Scope | Minimum |
|-------|---------|
| Domain layer (entities, value objects, use cases) | **90%** |
| Data layer (repository adapters, data sources) | **80%** |
| Presentation layer (ViewModels) | **80%** |
| Overall | **80%** |

CI fails if thresholds are not met.

---

## Unit tests

### Domain / Use Case

```kotlin
// Kotlin — test with InMemoryRepository (no DB, no network)
class CreateOrderUseCaseTest {
  private val repository = InMemoryOrderRepository()
  private val useCase = CreateOrderUseCase(repository)

  @Test
  fun `returns success when order is valid`() = runTest {
    val result = useCase(CreateOrderParams(items = listOf(/* ... */)))
    assertTrue(result.isSuccess)
    assertEquals(1, repository.count())
  }

  @Test
  fun `returns validation error when items list is empty`() = runTest {
    val result = useCase(CreateOrderParams(items = emptyList()))
    assertTrue(result.isFailure)
    assertEquals(DomainError.ValidationError::class, (result as Failure).error::class)
  }
}
```

### ViewModel

```swift
// Swift — test ViewModel with mock use cases
class OrderViewModelTests: XCTestCase {
  func test_load_emitsLoadingThenSuccess() async {
    let vm = OrderViewModel(
      getOrder: MockGetOrderUseCase(returning: .success(Order.fixture())),
      errorMapper: DomainErrorMapper()
    )
    var states: [UiState<OrderUiModel>] = []
    let cancel = vm.$state.sink { states.append($0) }

    vm.load(id: OrderId("123"))
    await Task.yield()

    XCTAssertEqual(states, [.loading, .success(Order.fixture().toUiModel())])
    cancel.cancel()
  }
}
```

---

## Integration tests

Test the wiring between local/remote data sources and the repository adapter.

```dart
// Flutter — real SQLite, mock HTTP
test('repository serves stale data when offline', () async {
  final local = SqfliteOrderDataSource(db: await openTestDb());
  await local.upsert(Order.fixture());
  final remote = MockRemoteOrderDataSource()..throws(TimeoutException());

  final repo = OrderRepositoryAdapter(remote: remote, local: local);
  final result = await repo.findById(OrderId('123'));

  expect(result, isA<Success>());
  verifyNever(remote.fetchById(any));  // local hit, remote never called
});
```

---

## Architecture tests

Verify that no cross-layer imports exist. Fail CI if violated.

```dart
// Flutter (using dart_dependency_validator or custom script)
// Domain must not import from data/, infrastructure/, or presentation/
void main() {
  test('domain has no framework imports', () {
    final domainFiles = Directory('lib/domain').listSync(recursive: true)
        .whereType<File>().where((f) => f.path.endsWith('.dart'));
    for (final file in domainFiles) {
      final content = file.readAsStringSync();
      expect(content, isNot(contains("import 'package:hive")));
      expect(content, isNot(contains("import 'package:dio")));
      expect(content, isNot(contains("import 'package:flutter/")));
    }
  });
}
```

---

## Test naming convention

```
[unit under test]_[scenario]_[expected outcome]

Examples:
  createOrder_withEmptyItems_returnsValidationError
  orderViewModel_onLoad_emitsLoadingThenSuccess
  authInterceptor_on401_refreshesTokenAndRetries
  orderRepository_whenOffline_servesStaleCache
```

---

## Mock conventions

- Domain tests: use hand-written `InMemory*Repository` implementations.
- Use Case tests: use hand-written fake adapters.
- ViewModel tests: use `Mock*UseCase` returning pre-configured `Result` values.
- Integration tests: use real persistence + mock HTTP (no real network calls).
- Never mock what you own (no mocking domain interfaces in domain tests).
