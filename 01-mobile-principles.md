# 01 — Mobile SDD Principles

## What is Spec-Driven Development (SDD)?

SDD is a structured workflow where **every line of code is preceded by a written spec**. The AI agent and developer co-author the spec before any implementation begins, then the agent enforces a layered build sequence with explicit quality gates.

## The 9 Non-Negotiable Rules

1. **Spec before code.** No implementation until `spec_feature` + `spec_clarify` + `spec_plan` + `spec_analyze` are all complete.
2. **Domain is king.** The domain layer has zero dependencies on frameworks, networking, or UI libraries.
3. **Offline-first by default.** Every repository adapter reads from local storage first; remote sync happens in the background.
4. **Ports and Adapters everywhere.** Every external service (network, storage, platform APIs) is hidden behind an interface defined in the domain.
5. **UI is a detail.** Screens and widgets observe ViewModel state; they never call use cases directly and hold no business logic.
6. **One-shot effects, not shared state.** Navigation, dialogs, and snackbars are emitted as single-consumption effects.
7. **Security is not optional.** Every credential goes through Secure Storage (Keychain/Keystore). No plain-text secrets. Certificate pinning on all authenticated endpoints.
8. **Gate before proceeding.** All tests for a phase must pass before the next phase starts. The agent enforces this automatically.
9. **Traceability by convention.** Every file links to its pattern number in a comment or doc annotation.

## The STOP / PASS convention

- **✋ STOP** — a hard gate. The agent will not proceed until tests pass.
- **PASS** — issued by a tool response after the gate is satisfied.

## Quality thresholds (non-negotiable)

| Metric | Threshold |
|--------|-----------|
| Domain layer test coverage | ≥ 90% |
| Overall test coverage | ≥ 80% |
| Clippy warnings (CI) | 0 |
| OWASP Mobile Top 10 compliance | All items checked |
| Architecture test (no cross-layer imports) | Pass |
