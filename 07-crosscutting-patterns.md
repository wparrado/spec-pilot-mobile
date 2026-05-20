# 07 — Cross-Cutting Patterns

These patterns are used across all layers.

---

## Pattern #11 — Result / Railway

A typed container for operations that can succeed or fail. Replaces exceptions for domain operations.

```dart
// Flutter / Dart
sealed class Result<S, F> {
  const Result();
}

class Success<S, F> extends Result<S, F> {
  final S value;
  const Success(this.value);
}

class Failure<S, F> extends Result<S, F> {
  final F error;
  const Failure(this.error);
}

extension ResultExtensions<S, F> on Result<S, F> {
  R fold<R>({required R Function(S) onSuccess, required R Function(F) onFailure}) =>
    switch (this) {
      Success(:final value) => onSuccess(value),
      Failure(:final error) => onFailure(error),
    };
}
```

```swift
// Swift — use the standard Result<Success, Failure> type
extension Result {
  var isSuccess: Bool { if case .success = self { return true } else { return false } }
  var value: Success? { if case .success(let v) = self { return v } else { return nil } }
}
```

**Rules:**
- Domain and data methods return `Result<T, DomainError>` — never `throw`.
- Only infrastructure code that calls OS APIs may use `try/catch` internally; it must convert to `Result` before returning.

---

## Pattern #12 — App State Machine

Models the global authentication / session lifecycle.

```kotlin
// Kotlin
sealed class AppState {
  object Splash : AppState()
  object Unauthenticated : AppState()
  data class Authenticated(val userId: String) : AppState()
  object SessionExpired : AppState()
}

sealed class AppEvent {
  data class LoggedIn(val userId: String) : AppEvent()
  object LoggedOut : AppEvent()
  object SessionExpired : AppEvent()
}

class AppStateMachine {
  private val _state = MutableStateFlow<AppState>(AppState.Splash)
  val state: StateFlow<AppState> = _state

  fun transition(event: AppEvent) {
    _state.value = when (val current = _state.value) {
      AppState.Splash, AppState.Unauthenticated ->
        if (event is AppEvent.LoggedIn) AppState.Authenticated(event.userId) else current
      is AppState.Authenticated ->
        when (event) {
          AppEvent.LoggedOut -> AppState.Unauthenticated
          AppEvent.SessionExpired -> AppState.SessionExpired
          else -> current
        }
      AppState.SessionExpired -> AppState.Unauthenticated
    }
  }
}
```

---

## Pattern #13 — Error Mapping

Translates domain errors to user-readable localized strings. Lives in the presentation layer.

```swift
// Swift
struct DomainErrorMapper {
  func map(_ error: DomainError) -> String {
    switch error {
    case .notFound:            return NSLocalizedString("error.not_found", comment: "")
    case .validationError(let msg): return msg
    case .unauthorized:        return NSLocalizedString("error.unauthorized", comment: "")
    case .unavailable:         return NSLocalizedString("error.unavailable", comment: "")
    default:                   return NSLocalizedString("error.generic", comment: "")
    }
  }
}
```

**Rules:**
- The mapper lives in the presentation layer — never in the domain.
- All strings go through the localization system (`.strings`, ARB, etc.).
- Never expose raw error codes or stack traces to the user.
