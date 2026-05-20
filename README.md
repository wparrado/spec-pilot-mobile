# 📱 Spec-Pilot Mobile

> **Stop vibe-coding mobile apps. Start shipping production-grade iOS, Android, and cross-platform apps.**

**Spec-Pilot Mobile** is a language-agnostic MCP server that plugs into your AI agent (Claude, Cursor, Windsurf, Continue…) and enforces a structured **Spec-Driven Development (SDD)** workflow for **mobile development** — from idea to App Store / Play Store, step by step, with no shortcuts.

No copy-pasting prompts. No skipped steps. No hand-waving about architecture.  
The agent follows the 9-step SDD cycle automatically, backed by 13 playbook documents and 26 battle-tested mobile patterns.

> **One install. Every mobile project. Automatically.**

---

## ✨ Why Spec-Pilot Mobile?

| Without Spec-Pilot Mobile | With Spec-Pilot Mobile |
|---|---|
| Agent jumps straight to code | Agent clarifies requirements first |
| Architecture invented on the fly | 26 proven mobile patterns, always applied |
| "Works on simulator" | 120-item production checklist enforced |
| Security gaps in production | OWASP Mobile Top 10 baked into every layer |
| Tech debt from day one | Domain → Data → Infra → Presentation, clean separation |
| Offline surprises in prod | Offline-First strategy enforced from Phase 2 |

---

## Who is this for?

| Role | How to use it |
|------|--------------|
| **Mobile Developer** | Configure `.mcp.json` once — the agent handles every step automatically. |
| **AI Agent** | The SDD tools appear automatically; the agent self-routes through the 9-step cycle. |
| **Tech Lead** | Use the verification checklist tool to review compliance across the team. |

### Supported stacks

| Platform | Frameworks |
|----------|-----------|
| **iOS** | Swift + SwiftUI, Swift + UIKit |
| **Android** | Kotlin + Jetpack Compose, Kotlin + XML Views |
| **Cross-platform** | Flutter (Dart), React Native (TypeScript) |

---

## 🔄 The 9-Step SDD Cycle

| Step | Tool | What happens |
|------|------|-------------|
| 0 | `spec_constitution` | Establishes project principles + 26 mandatory mobile patterns *(once per project)* |
| 1 | `spec_feature` | Defines **WHAT** to build: user stories + acceptance criteria |
| 2 | `spec_clarify` | Resolves every `[NEEDS CLARIFICATION]` marker before planning |
| 3 | `spec_plan` | Defines **HOW** to build: patterns, data model, navigation contracts |
| 4 | `spec_analyze` | Cross-artifact consistency gate — spec ↔ plan alignment verified |
| 5 | `spec_tasks` | Generates executable task list by layer with parallelization hints |
| 6 | `scaffold_phase` | Implements one phase at a time: Domain → Data → Infra → Presentation → Tests+CI |
| 7 | `verify_project` | Runs the **120-item** Mobile Playbook compliance checklist |
| 8 | `adjust_finding` | Fixes every ❌ finding with targeted remediation, then re-verifies |

**Helper:** `sdd_next_step` — call any time to find out which step to run next.

Each step is a gate for the next. The agent enforces the order automatically.

---

## 🏗️ Mobile Architecture (5 Phases)

```
Phase 1 — Domain Layer
  Entity · Value Object · Use Case · Repository Port · Domain Event · Result/Railway

Phase 2 — Data Layer
  Remote Data Source · Local Data Source · Repository Adapter · Offline-First · Cache-Aside

Phase 3 — Infrastructure
  Auth Interceptor · Secure Storage · Feature Flags · Push Notifications
  Biometric Auth · Deep Linking · Background Jobs · Crash & Observability

Phase 4 — Presentation
  UI State (Loading|Success|Error|Empty) · ViewModel · Screen · Navigation
  One-Shot Effects · Error Mapping · App State Machine

Phase 5 — Tests, CI & Distribution
  Architecture Tests · Coverage ≥80% (domain ≥90%) · Signing
  CI Pipeline (GitHub Actions / Bitrise / Xcode Cloud)
  Distribution (TestFlight / Firebase App Distribution / Play Internal Track)
```

---

## 📦 MCP Server

A **Rust-native binary** (~2.5 MB, zero runtime dependencies, <1 ms startup) that exposes all SDD tools via the [Model Context Protocol](https://modelcontextprotocol.io).

### 1 · Build the binary

```bash
# Requires Rust — install from https://rustup.rs if needed
cd mcp-server
cargo build --release

# Binary: mcp-server/target/release/spec-pilot-mobile-mcp
# Size:   ~2.5MB · Zero runtime dependencies · <1ms startup
```

Or install globally:
```bash
cd mcp-server && cargo install --path .
# After this: spec-pilot-mobile-mcp  (available anywhere in PATH)
```

### 2 · Configure your AI agent

**Claude Desktop** — `~/Library/Application Support/Claude/claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "spec-pilot-mobile": {
      "command": "/path/to/spec-pilot-mobile/mcp-server/target/release/spec-pilot-mobile-mcp"
    }
  }
}
```

**Cursor / Windsurf / Continue** — create `.mcp.json` at the project root:
```json
{
  "mcpServers": {
    "spec-pilot-mobile": {
      "command": "/path/to/spec-pilot-mobile-mcp"
    }
  }
}
```

---

## 🗺️ 26 Mobile Patterns

| # | Pattern | Layer |
|---|---------|-------|
| 1 | Entity / Aggregate Root | Domain |
| 2 | Value Object | Domain |
| 3 | Use Case / Interactor | Domain |
| 4 | Repository Port | Domain |
| 5 | Domain Event | Domain |
| 6 | UI State (Loading\|Success\|Error\|Empty) | Presentation |
| 7 | ViewModel / Presenter | Presentation |
| 8 | Screen / View | Presentation |
| 9 | Navigation Contract | Presentation |
| 10 | One-Shot Effect | Presentation |
| 11 | Result / Railway | Cross-cutting |
| 12 | App State Machine | Cross-cutting |
| 13 | Error Mapping | Cross-cutting |
| 14 | Remote Data Source | Data |
| 15 | Local Data Source | Data |
| 16 | Repository Adapter | Data |
| 17 | Offline-First | Data |
| 18 | Cache-Aside | Data |
| 19 | Auth Interceptor | Infrastructure |
| 20 | Secure Storage | Infrastructure |
| 21 | Feature Flag | Infrastructure |
| 22 | Push Notifications | Infrastructure |
| 23 | Biometric Auth | Infrastructure |
| 24 | Deep Linking | Infrastructure |
| 25 | Background Job | Infrastructure |
| 26 | Crash & Observability | Infrastructure |

---

## 🏁 Quick Start

```
1. Build the binary (see above)
2. Configure .mcp.json in your project root
3. Open your AI agent (Claude, Cursor, etc.)
4. Ask: "Start a new mobile project with spec-pilot-mobile"
5. The agent calls spec_constitution → guides you through all 9 steps automatically
```

---

## 🔒 Security & Quality Philosophy

- **OWASP Mobile Top 10** enforced in the infrastructure phase
- **Zero-trust by default**: certificate pinning, secure storage, no plain-text credentials
- **Gate-based progression**: the agent cannot skip phases — each step checks the previous one
- **Traceability**: every pattern links to its playbook document
- **Resilience**: offline-first strategy, circuit breaking, retry with backoff

---

## 📚 Playbook Documents

| File | Topic |
|------|-------|
| `01-mobile-principles.md` | Core SDD principles for mobile |
| `02-project-structure.md` | Folder layout per framework |
| `03-domain-patterns.md` | Entity, Value Object, Use Case, Repository Port, Domain Event |
| `04-data-patterns.md` | Remote/Local Data Source, Repository Adapter, Offline-First, Cache-Aside |
| `05-infrastructure-patterns.md` | Auth Interceptor, Secure Storage, Feature Flags, Push, Biometric, Deep Links |
| `06-presentation-patterns.md` | UI State, ViewModel, Screen, Navigation, One-Shot Effects |
| `07-crosscutting-patterns.md` | Result/Railway, App State Machine, Error Mapping |
| `08-testing-strategy.md` | Unit, integration, widget/component, E2E, architecture tests |
| `09-quality-gates.md` | Coverage thresholds, gate checklist |
| `10-security.md` | OWASP Mobile Top 10, certificate pinning, secure storage |
| `11-verification-checklist.md` | 120-item production readiness checklist |
| `12-ci-cd-distribution.md` | GitHub Actions, Fastlane, TestFlight, Play Store |
| `13-mobile-observability.md` | Crash reporting, analytics, performance monitoring |

---

## 🤝 Relationship to spec-pilot

`spec-pilot-mobile` is a **sibling project** to [spec-pilot](https://github.com/wparrado/spec-pilot), which targets backend API development. Both share the same 9-step SDD philosophy and gate-enforcement design, but have completely different playbooks, pattern catalogs, and phase structures tailored to their respective domains.

Use `spec-pilot` for backend microservices and APIs. Use `spec-pilot-mobile` for iOS, Android, and cross-platform apps.

---

## License

MIT
