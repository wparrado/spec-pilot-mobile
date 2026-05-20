# 09 — Quality Gates

## Gate 1 — After `spec_feature` + `spec_clarify`

- [ ] All `[NEEDS CLARIFICATION]` markers resolved
- [ ] User stories in `Given / When / Then` format
- [ ] Acceptance criteria are testable (no ambiguous language)
- [ ] Platform target confirmed (iOS / Android / Cross-platform)

## Gate 2 — After `spec_plan`

- [ ] Plan references at least one pattern from the Mobile Playbook (#1–#26)
- [ ] Navigation contracts documented (all routes named)
- [ ] Offline behavior specified (what happens with no connectivity)
- [ ] Authentication strategy specified
- [ ] Data model field types and validation rules documented

## Gate 3 — After `spec_analyze`

- [ ] spec.md ↔ plan.md: all user stories have corresponding planned components
- [ ] plan.md ↔ patterns: every component maps to a pattern number
- [ ] No contradictions between spec and plan

## Gate 4 — Phase 1 complete (Domain)

- [ ] All entities have immutable fields
- [ ] All value objects validate through `create()` → `Result`
- [ ] All repository ports defined as interfaces (no concrete implementations in domain)
- [ ] Use cases have 100% happy-path + failure-path unit test coverage
- [ ] Domain layer has **zero** framework/networking/UI imports
- [ ] Domain test coverage ≥ **90%**
- [ ] `✋ STOP` gate satisfied (all domain tests green)

## Gate 5 — Phase 2 complete (Data)

- [ ] Remote data source handles: success, timeout, 401, 404, 500
- [ ] Local data source handles: insert, update, get, delete, and migration
- [ ] Repository adapter: cache-hit (no remote call), cache-miss (remote called + cached), offline write queued
- [ ] Offline-First strategy implemented
- [ ] Data layer test coverage ≥ **80%**

## Gate 6 — Phase 3 complete (Infrastructure)

- [ ] Auth interceptor: token injection, 401 refresh flow, session expiry transition
- [ ] Secure storage: write/read/delete; verified NOT in plain preferences/UserDefaults
- [ ] Feature flags: unknown flag returns `false`
- [ ] OWASP Mobile Top 10 items checked (see `10-security.md`)
- [ ] Infrastructure test coverage ≥ **80%**

## Gate 7 — Phase 4 complete (Presentation)

- [ ] Every screen observes ViewModel state only (no direct use case calls)
- [ ] ViewModel unit tests: loading → success, loading → error, effect emissions
- [ ] Error mapper: every `DomainError` variant maps to a non-empty localized string
- [ ] Navigation: all routes in `AppRoute` resolve to a screen
- [ ] No business logic in Screen/View files
- [ ] Presentation test coverage ≥ **80%**

## Gate 8 — Phase 5 complete (Tests + CI + Distribution)

- [ ] Architecture tests pass (no cross-layer imports)
- [ ] Overall test coverage ≥ **80%** (domain ≥ **90%**)
- [ ] CI pipeline green (format → lint → test → build)
- [ ] Signing configured: debug / staging / production
- [ ] Distribution configured: TestFlight / Firebase App Distribution / Play Internal Track
- [ ] `.env.example` (or equivalent) documents all env variables
- [ ] `.mcp.json` points to `spec-pilot-mobile-mcp` binary

## Final Gate — `verify_project` passes

All 120 checklist items green. `adjust_finding` called for every ❌ until all are ✅.
