# 06 — Presentation Patterns

The presentation layer observes ViewModel state and delegates all actions to ViewModels. Screens hold zero business logic.

---

## Pattern #6 — UI State

A sealed type that covers every possible display state.

```kotlin
// Kotlin
sealed class UiState<out T> {
  data object Loading : UiState<Nothing>()
  data class Success<T>(val data: T) : UiState<T>()
  data class Error(val message: String) : UiState<Nothing>()
  data object Empty : UiState<Nothing>()
}
```

**Rules:**
- Never add a `data: T?` field to `Loading` or `Error` — keeps each state unambiguous.
- Use `Empty` explicitly when the result set is legitimately empty (vs `Error`).

---

## Pattern #7 — ViewModel / Presenter

Holds state and processes user actions. No UI framework imports.

```kotlin
// Kotlin (Android ViewModel + Kotlin Flow)
class OrderViewModel(
  private val getOrder: GetOrderUseCase,
  private val createOrder: CreateOrderUseCase,
  private val errorMapper: DomainErrorMapper
) : ViewModel() {

  private val _state = MutableStateFlow<UiState<OrderUiModel>>(UiState.Loading)
  val state: StateFlow<UiState<OrderUiModel>> = _state.asStateFlow()

  private val _effects = Channel<OrderEffect>(Channel.BUFFERED)
  val effects: Flow<OrderEffect> = _effects.receiveAsFlow()

  fun load(id: OrderId) {
    viewModelScope.launch {
      _state.value = UiState.Loading
      getOrder(id).fold(
        onSuccess = { _state.value = UiState.Success(it.toUiModel()) },
        onFailure = { _state.value = UiState.Error(errorMapper.map(it)) }
      )
    }
  }
}
```

**Rules:**
- State flows only flow in one direction: ViewModel → Screen.
- User actions flow: Screen → ViewModel (method calls, never shared mutable state).
- ViewModel must be unit-testable without any UI framework.

---

## Pattern #8 — Screen / View

Observes ViewModel state. Delegates every action.

```swift
// Swift + SwiftUI
struct OrderDetailView: View {
  @StateObject private var viewModel: OrderDetailViewModel

  var body: some View {
    Group {
      switch viewModel.state {
      case .loading:
        ProgressView()
      case .success(let order):
        OrderContentView(order: order, onSave: viewModel.save)
      case .error(let message):
        ErrorView(message: message, onRetry: { viewModel.load(id: orderId) })
      case .empty:
        EmptyStateView()
      }
    }
    .onAppear { viewModel.load(id: orderId) }
    .onReceive(viewModel.effects) { handleEffect($0) }
  }

  private func handleEffect(_ effect: OrderEffect) {
    switch effect {
    case .navigateTo(let route): navigator.push(route)
    case .showSnackbar(let msg): // show toast
    }
  }
}
```

---

## Pattern #9 — Navigation Contract

Typed routes documented in `contracts/navigation.md` before any screen is built.

```dart
// Flutter
sealed class AppRoute {
  const AppRoute();
}

class OrderList extends AppRoute { const OrderList(); }
class OrderDetail extends AppRoute {
  final String orderId;
  const OrderDetail(this.orderId);
}
class OrderCreate extends AppRoute { const OrderCreate(); }
```

**Rules:**
- All routes are defined before Phase 4 starts (declared in plan.md navigation contracts).
- No `Navigator.pushNamed(context, '/orders/$id')` string-based navigation.
- Deep link handler maps URIs to these typed routes.

---

## Pattern #10 — One-Shot Effect

Single-consumption side effects: navigation, dialogs, snackbars.

```kotlin
// Kotlin
sealed class OrderEffect {
  data class NavigateTo(val route: AppRoute) : OrderEffect()
  data class ShowSnackbar(val message: String) : OrderEffect()
  data class ShowDialog(val title: String, val body: String) : OrderEffect()
}
```

**Rules:**
- Effects are emitted through a `Channel` (Kotlin) or `PassthroughSubject` (Swift Combine) or `Stream` (Flutter).
- They are **not** replayed to new subscribers — one-shot only.
- The Screen collects effects in `LaunchedEffect`/`onReceive`/`listen` and acts on them exactly once.
