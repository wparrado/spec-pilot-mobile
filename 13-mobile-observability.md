# 13 — Mobile Observability

## Crash Reporting

All mobile apps must configure crash reporting before shipping to production.

### Required setup (Pattern #26)

- **Firebase Crashlytics** (recommended for cross-platform)
- **Sentry** (alternative)
- **Apple Crash Reporter** (iOS-only)

**Initialization** — must happen in `main.*` before any other service:

```dart
// Flutter
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();

  // Catch all Flutter framework errors
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

  // Catch all Dart async errors
  await runZonedGuarded(
    () async => runApp(const MyApp()),
    (error, stack) => FirebaseCrashlytics.instance.recordError(error, stack, fatal: true),
  );
}
```

---

## Privacy requirements

| Data | Allowed | Rule |
|------|---------|------|
| User ID | ✅ (hashed) | Hash with SHA-256, never raw email/phone |
| Screen name | ✅ | e.g. `order_detail`, not `order_detail?id=123` |
| Action name | ✅ | e.g. `add_to_cart` |
| Error type | ✅ | e.g. `DomainError.unauthorized` |
| Email / phone | ❌ | Never log PII |
| Raw user input | ❌ | Never log form field values |
| Payment details | ❌ | Never, under any circumstances |

---

## Performance Monitoring

Track key user-perceived metrics:

| Metric | Tool | Threshold |
|--------|------|-----------|
| App cold start | Firebase Performance | < 2 s |
| Screen load time | Custom trace | < 1 s |
| Network request time | Firebase Performance | < 3 s (p95) |
| JS bundle load (RN) | Flipper / Metro | < 1 s |
| Crash-free sessions | Crashlytics | ≥ 99.5% |
| ANR rate (Android) | Play Console | < 0.47% |

---

## Logging conventions

Use structured logging with levels:

```kotlin
// Kotlin
object Logger {
  fun debug(tag: String, message: String) {
    if (BuildConfig.DEBUG) Log.d(tag, message)
  }

  fun error(tag: String, message: String, throwable: Throwable? = null) {
    // Production: send to Crashlytics non-fatal
    FirebaseCrashlytics.getInstance().log("[$tag] $message")
    throwable?.let { FirebaseCrashlytics.getInstance().recordException(it) }
  }
}
```

**Rules:**
- `debug` logs are stripped in release builds.
- `error` logs go to crash reporting as non-fatal events.
- No stack traces printed in production logs.
- Log the error *type* and a safe message, not the user's data.

---

## Alerting

Set up Firebase Crashlytics alerts for:
- New crash types (immediate alert)
- Crash-free session rate drop below 99.5% (immediate alert)
- Crash velocity spike (> 100 new users affected in 1 hour)

Configure these alerts in the Firebase Console → Crashlytics → Alerts.
