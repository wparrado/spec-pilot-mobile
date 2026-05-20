# 05 — Infrastructure Patterns

The infrastructure layer provides platform-specific adapters for all ports that need OS or network access. All adapters implement domain interfaces; the domain never imports infrastructure.

---

## Pattern #19 — Auth Interceptor

Injects the access token into every authenticated request. Handles 401 → refresh → retry.

```kotlin
// Kotlin (OkHttp Interceptor)
class AuthInterceptor(
  private val secureStorage: ISecureStorage,
  private val authService: IAuthService,
  private val appState: AppStateMachine
) : Interceptor {
  override fun intercept(chain: Interceptor.Chain): Response {
    val token = secureStorage.read("access_token").getOrNull() ?: return chain.proceed(chain.request())
    val request = chain.request().newBuilder()
      .addHeader("Authorization", "Bearer $token")
      .build()
    val response = chain.proceed(request)
    if (response.code == 401) {
      val refreshToken = secureStorage.read("refresh_token").getOrNull()
        ?: run { appState.transition(AppEvent.SessionExpired); return response }
      val newTokens = authService.refresh(refreshToken).getOrElse {
        appState.transition(AppEvent.SessionExpired)
        return response
      }
      secureStorage.write("access_token", newTokens.accessToken)
      secureStorage.write("refresh_token", newTokens.refreshToken)
      return chain.proceed(request.newBuilder().header("Authorization", "Bearer ${newTokens.accessToken}").build())
    }
    return response
  }
}
```

---

## Pattern #20 — Secure Storage

Wraps platform credential storage (Keychain on iOS, Keystore on Android).

```swift
// Swift — Keychain adapter
class KeychainStorage: ISecureStorage {
  func write(_ key: String, value: String) -> Result<Void, DomainError> {
    let data = Data(value.utf8)
    let query: [CFString: Any] = [
      kSecClass: kSecClassGenericPassword,
      kSecAttrAccount: key,
      kSecValueData: data,
      kSecAttrAccessible: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    ]
    SecItemDelete(query as CFDictionary) // remove old
    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess ? .success(()) : .failure(.secureStorageError("Write failed: \(status)"))
  }
}
```

**Rules:**
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` — not backed up to iCloud, not accessible when locked.
- Android: use `EncryptedSharedPreferences` wrapping Android Keystore.
- Never write secrets to `UserDefaults`, `SharedPreferences`, or plain files.

---

## Pattern #21 — Feature Flag

Toggle features without releasing a new binary.

```dart
// Flutter
class RemoteConfigFeatureFlags implements IFeatureFlags {
  final FirebaseRemoteConfig _config;

  @override
  bool isEnabled(String flagName) {
    // Default false for all unknown flags — fail-safe
    return _config.getBool(flagName);
  }

  Future<void> refresh() async {
    await _config.fetchAndActivate();
  }
}
```

---

## Pattern #22 — Push Notifications

```kotlin
// Kotlin (Firebase Cloud Messaging)
class FcmPushNotifications(private val context: Context) : IPushNotifications {
  override suspend fun requestPermission(): Result<Boolean, DomainError> {
    val channel = NotificationChannel("default", "Default", NotificationManager.IMPORTANCE_DEFAULT)
    context.getSystemService(NotificationManager::class.java).createNotificationChannel(channel)
    return Result.success(true)
  }

  override suspend fun getToken(): Result<String, DomainError> = runCatching {
    FirebaseMessaging.getInstance().token.await()
  }.fold(onSuccess = { Result.success(it) }, onFailure = { Result.failure(it.toDomainError()) })
}
```

---

## Pattern #23 — Biometric Auth

```swift
// Swift — Local Authentication
class BiometricAuthAdapter: IBiometricAuth {
  func authenticate(reason: String) async -> Result<Void, DomainError> {
    let context = LAContext()
    var error: NSError?
    guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
      return .failure(.biometricUnavailable(error?.localizedDescription ?? ""))
    }
    do {
      let success = try await context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: reason)
      return success ? .success(()) : .failure(.biometricFailed)
    } catch {
      return .failure(.biometricFailed)
    }
  }
}
```

---

## Pattern #24 — Deep Linking

Map incoming URLs / Universal Links to typed routes.

```dart
// Flutter
class DeepLinkHandler {
  final AppRouter _router;

  void handle(Uri uri) {
    final route = _parseRoute(uri);
    if (route != null) _router.push(route);
  }

  AppRoute? _parseRoute(Uri uri) => switch (uri.pathSegments) {
    ['orders', final id] => AppRoute.orderDetail(id),
    ['profile'] => AppRoute.profile,
    _ => null,
  };
}
```

---

## Pattern #25 — Background Job

```kotlin
// Kotlin — WorkManager
class SyncWorker(ctx: Context, params: WorkerParameters) : CoroutineWorker(ctx, params) {
  override suspend fun doWork(): Result {
    val syncUseCase = DependencyInjection.syncUseCase
    return when (syncUseCase.execute()) {
      is Success -> Result.success()
      is Failure -> if (runAttemptCount < 3) Result.retry() else Result.failure()
    }
  }
}
```

---

## Pattern #26 — Crash & Observability

```dart
// Flutter
class ObservabilityService {
  static Future<void> initialize() async {
    await Firebase.initializeApp();
    FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
    PlatformDispatcher.instance.onError = (error, stack) {
      FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
      return true;
    };
  }

  static void log(String message) => FirebaseCrashlytics.instance.log(message);

  // NEVER log PII — user IDs must be hashed or anonymized
  static void setUserId(String hashedId) => FirebaseCrashlytics.instance.setUserIdentifier(hashedId);
}
```

**Rules:**
- No PII in crash logs or analytics events.
- User identifiers must be hashed or replaced with opaque tokens.
- Disable analytics in debug builds by default.
