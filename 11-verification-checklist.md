# 11 — Verification Checklist

This is the 120-item checklist used by the `verify_project` tool. All items must be ✅ before release.

---

## Domain Layer (20 items)

- [ ] 1. All entities have immutable fields (final / val / let / readonly)
- [ ] 2. No entity has a public mutable setter
- [ ] 3. All value objects created through `create()` → `Result`
- [ ] 4. All value object constructors are private
- [ ] 5. All repository interfaces defined in domain layer
- [ ] 6. No concrete DB/network type appears in any repository interface
- [ ] 7. All use cases have exactly one public method
- [ ] 8. All use case dependencies are injected via constructor
- [ ] 9. No framework import in any domain file
- [ ] 10. No networking import in any domain file
- [ ] 11. No UI import in any domain file
- [ ] 12. Domain errors defined as a sealed/enum type
- [ ] 13. Domain events defined as immutable types
- [ ] 14. All use cases unit-tested (happy path + failure path)
- [ ] 15. InMemory repository exists for every repository port
- [ ] 16. Domain test coverage ≥ 90%
- [ ] 17. Value object equality tested
- [ ] 18. Entity mutation methods tested for invariant violations
- [ ] 19. Domain events raised correctly in use case tests
- [ ] 20. Architecture test confirms domain has no framework/data/presentation imports

## Data Layer (20 items)

- [ ] 21. Remote data source handles network timeout
- [ ] 22. Remote data source handles 401 Unauthorized
- [ ] 23. Remote data source handles 404 Not Found
- [ ] 24. Remote data source handles 500 Server Error
- [ ] 25. Remote data source maps all errors to typed NetworkError variants
- [ ] 26. Local data source supports insert, update, get, delete
- [ ] 27. Local data source has migration support (versioned schema)
- [ ] 28. Repository adapter implements Offline-First (local-first reads)
- [ ] 29. Repository adapter queues writes when offline
- [ ] 30. Repository adapter maps NetworkError → DomainError
- [ ] 31. Cache-Aside implemented if plan requires it
- [ ] 32. InMemory repository used in use case tests (not real DB)
- [ ] 33. Data layer integration tests use real local storage + mock HTTP
- [ ] 34. Data layer test coverage ≥ 80%
- [ ] 35. No domain logic in repository adapter (only orchestration)
- [ ] 36. DTOs are separate from domain entities
- [ ] 37. DB schema documented in plan.md
- [ ] 38. Last-sync-at timestamp stored for offline freshness checks
- [ ] 39. Sync queue items survive app restart
- [ ] 40. Architecture test confirms data layer does not import presentation

## Infrastructure Layer (20 items)

- [ ] 41. Auth interceptor injects Bearer token on all authenticated requests
- [ ] 42. Auth interceptor handles 401 → refresh → retry
- [ ] 43. Auth interceptor emits SessionExpired on refresh failure
- [ ] 44. All tokens stored in Secure Storage (Keychain/Keystore)
- [ ] 45. No tokens in UserDefaults / SharedPreferences / plain files
- [ ] 46. Feature flags return false for unknown flag names
- [ ] 47. Push notification permission requested at appropriate time (not on launch)
- [ ] 48. Push token registered with backend after permission granted
- [ ] 49. Biometric auth falls back gracefully when unavailable
- [ ] 50. Deep links map to typed routes (no string-based navigation)
- [ ] 51. Background jobs use exponential backoff on failure
- [ ] 52. Crash reporting initialized before any other service
- [ ] 53. No PII in crash logs or analytics
- [ ] 54. User identifier hashed in observability services
- [ ] 55. Analytics disabled in debug build by default
- [ ] 56. Certificate pinning active on all authenticated endpoints
- [ ] 57. TLS 1.2+ enforced
- [ ] 58. OWASP M1–M10 checklist completed (see 10-security.md)
- [ ] 59. Infrastructure test coverage ≥ 80%
- [ ] 60. Architecture test confirms infrastructure does not import presentation

## Presentation Layer (20 items)

- [ ] 61. All screens observe ViewModel state only (no direct use case calls)
- [ ] 62. Screens hold zero business logic
- [ ] 63. ViewModels tested without any UI framework
- [ ] 64. ViewModel: loading state emitted before every async operation
- [ ] 65. ViewModel: success state tested for happy path
- [ ] 66. ViewModel: error state tested for failure path
- [ ] 67. One-Shot effects emitted through Channel / PassthroughSubject / Stream
- [ ] 68. Effects not replayed to new subscribers
- [ ] 69. Navigation uses typed routes (no string-based push)
- [ ] 70. All routes in AppRoute resolve to a screen
- [ ] 71. Error mapper: every DomainError variant maps to non-empty localized string
- [ ] 72. Error messages do not expose raw error codes to user
- [ ] 73. Loading state shows a visual indicator (not a blank screen)
- [ ] 74. Empty state has a user-friendly message and/or action
- [ ] 75. Screens tested with at least one widget/component test per state
- [ ] 76. App State Machine tested: all valid transitions covered
- [ ] 77. Deep link routes integrate with typed navigation
- [ ] 78. Accessibility labels set on interactive elements
- [ ] 79. Presentation test coverage ≥ 80%
- [ ] 80. Architecture test confirms presentation does not import data layer directly

## Tests & Quality (20 items)

- [ ] 81. Overall test coverage ≥ 80%
- [ ] 82. Domain test coverage ≥ 90%
- [ ] 83. All tests in CI (not just local)
- [ ] 84. CI fails on any test failure
- [ ] 85. Architecture tests run in CI
- [ ] 86. No `.skip` or `.only` on any test in CI
- [ ] 87. Snapshot tests committed and reviewed
- [ ] 88. E2E tests cover the critical user journey (login → main feature → logout)
- [ ] 89. Test data does not use real credentials
- [ ] 90. Tests run in < 5 minutes in CI
- [ ] 91. Flaky tests eliminated or quarantined
- [ ] 92. Mock/fake objects documented
- [ ] 93. Test naming follows: `subject_scenario_expectedOutcome`
- [ ] 94. InMemory repositories cover all repository methods
- [ ] 95. No `Thread.sleep` or `delay` in unit tests
- [ ] 96. Async tests use structured concurrency (not callbacks)
- [ ] 97. Test fixtures defined in a shared `fixtures/` or `helpers/` location
- [ ] 98. CI runs on every push to main and every PR
- [ ] 99. Code coverage report uploaded in CI
- [ ] 100. All CI jobs (test, lint, build, audit) pass on main branch

## CI / CD / Distribution (20 items)

- [ ] 101. CI checks formatting (rustfmt / ktfmt / swiftformat / dart format)
- [ ] 102. CI runs lint / clippy with zero warnings
- [ ] 103. CI builds the release binary / app bundle
- [ ] 104. Release build has all debug flags disabled
- [ ] 105. Signing configured: debug / staging / production certificates
- [ ] 106. Signing certificates stored in CI secrets (not committed)
- [ ] 107. TestFlight / Firebase App Distribution / Play Internal Track configured
- [ ] 108. Distribution workflow triggered automatically on tag push
- [ ] 109. App version bump automated (not manual)
- [ ] 110. `.env.example` documents all required environment variables
- [ ] 111. No secrets committed to git
- [ ] 112. `.gitignore` excludes `target/`, `build/`, `.DS_Store`, `*.keystore`
- [ ] 113. Pre-commit hook runs format check
- [ ] 114. Dependency audit (`cargo audit` / `npm audit` / `pod audit`) in CI
- [ ] 115. `cargo audit` / security scan passes
- [ ] 116. App size budget defined and checked in CI
- [ ] 117. Crash-free session rate threshold defined
- [ ] 118. `.mcp.json` points to correct `spec-pilot-mobile-mcp` binary path
- [ ] 119. `sdd_next_step` returns "project is complete" after all steps done
- [ ] 120. `verify_project` returns all ✅ with zero ❌ findings
