# 02 — Project Structure

## Flutter (Dart)

```
lib/
  main.dart                   # Composition Root
  domain/
    entities/                 # Pattern #1
    value_objects/            # Pattern #2
    use_cases/                # Pattern #3
    repositories/             # Pattern #4 — port interfaces
    events/                   # Pattern #5
    errors/                   # DomainError sealed class
  data/
    remote/                   # Pattern #14 — remote data sources
    local/                    # Pattern #15 — local data sources (Hive/Drift/SQLite)
    repositories/             # Pattern #16 — repository adapters
  infrastructure/
    auth/                     # Pattern #19 — auth interceptor
    secure_storage/           # Pattern #20 — Keychain/Keystore adapter
    feature_flags/            # Pattern #21
    notifications/            # Pattern #22
    biometric/                # Pattern #23
    deep_links/               # Pattern #24
    background_jobs/          # Pattern #25
    observability/            # Pattern #26
  presentation/
    state/                    # Pattern #6 — UI state sealed classes
    viewmodels/               # Pattern #7
    screens/                  # Pattern #8
    navigation/               # Pattern #9
    effects/                  # Pattern #10
  core/
    result.dart               # Pattern #11 — Result<T,E>
    app_state_machine.dart    # Pattern #12
    error_mapper.dart         # Pattern #13
test/
  domain/
  data/
  infrastructure/
  presentation/
  architecture/               # import-rule tests
specs/
  NNN-<feature>/
    spec.md
    plan.md
    tasks.md
```

## iOS — Swift + SwiftUI

```
Sources/
  App/
    AppMain.swift              # @main, Composition Root
  Domain/
    Entities/                  # Pattern #1
    ValueObjects/              # Pattern #2
    UseCases/                  # Pattern #3
    Repositories/              # Pattern #4 — protocols
    Events/                    # Pattern #5
    Errors/                    # DomainError enum
  Data/
    Remote/                    # Pattern #14
    Local/                     # Pattern #15 — CoreData/SwiftData/GRDB
    Repositories/              # Pattern #16
  Infrastructure/
    Auth/                      # Pattern #19
    SecureStorage/             # Pattern #20 — Keychain
    FeatureFlags/              # Pattern #21
    Notifications/             # Pattern #22
    Biometric/                 # Pattern #23
    DeepLinks/                 # Pattern #24
    BackgroundJobs/            # Pattern #25
    Observability/             # Pattern #26
  Presentation/
    State/                     # Pattern #6
    ViewModels/                # Pattern #7
    Views/                     # Pattern #8
    Navigation/                # Pattern #9
    Effects/                   # Pattern #10
  Core/
    Result.swift               # Pattern #11
    AppStateMachine.swift      # Pattern #12
    ErrorMapper.swift          # Pattern #13
Tests/
  DomainTests/
  DataTests/
  InfrastructureTests/
  PresentationTests/
  ArchitectureTests/
specs/
```

## Android — Kotlin + Jetpack Compose

```
app/src/main/java/com/<package>/
  domain/
    entities/                  # Pattern #1
    valueobjects/              # Pattern #2
    usecases/                  # Pattern #3
    repositories/              # Pattern #4 — interfaces
    events/                    # Pattern #5
    errors/                    # DomainError sealed class
  data/
    remote/                    # Pattern #14
    local/                     # Pattern #15 — Room
    repositories/              # Pattern #16
  infrastructure/
    auth/                      # Pattern #19
    securestorage/             # Pattern #20 — Keystore
    featureflags/              # Pattern #21
    notifications/             # Pattern #22
    biometric/                 # Pattern #23
    deeplinks/                 # Pattern #24
    backgroundjobs/            # Pattern #25 — WorkManager
    observability/             # Pattern #26
  presentation/
    state/                     # Pattern #6
    viewmodels/                # Pattern #7
    screens/                   # Pattern #8
    navigation/                # Pattern #9
    effects/                   # Pattern #10
  core/
    Result.kt                  # Pattern #11
    AppStateMachine.kt         # Pattern #12
    ErrorMapper.kt             # Pattern #13

app/src/test/
app/src/androidTest/
specs/
```

## React Native — TypeScript

```
src/
  domain/
    entities/                  # Pattern #1
    valueObjects/              # Pattern #2
    useCases/                  # Pattern #3
    repositories/              # Pattern #4 — interfaces
    events/                    # Pattern #5
    errors/                    # DomainError union
  data/
    remote/                    # Pattern #14
    local/                     # Pattern #15 — MMKV/SQLite
    repositories/              # Pattern #16
  infrastructure/
    auth/                      # Pattern #19
    secureStorage/             # Pattern #20
    featureFlags/              # Pattern #21
    notifications/             # Pattern #22
    biometric/                 # Pattern #23
    deepLinks/                 # Pattern #24
    backgroundJobs/            # Pattern #25
    observability/             # Pattern #26
  presentation/
    state/                     # Pattern #6
    viewModels/                # Pattern #7
    screens/                   # Pattern #8
    navigation/                # Pattern #9
    effects/                   # Pattern #10
  core/
    result.ts                  # Pattern #11
    appStateMachine.ts         # Pattern #12
    errorMapper.ts             # Pattern #13
__tests__/
specs/
```
