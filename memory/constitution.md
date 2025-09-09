# Web Application Delivery Constitution

Defines the non‑negotiable engineering & product constraints for this web application platform. This document governs architecture, quality, security, and operational practice. It supersedes ad‑hoc preferences.

## Core Principles

### I. Modular, Boundary-Driven Architecture
Every domain capability lives in a clearly scoped module (package, service, or feature slice) with: (1) a single responsibility, (2) explicit public contracts (API types + events), (3) zero cyclic dependencies, (4) deterministic, hermetic tests, (5) generated docs. No "grab bag" utility buckets. Cross-module communication uses typed interfaces or versioned APIs only. UI layers consume domain adapters—not raw data sources.

### II. Contract-First Interfaces (API, Events, & Automation)
All external + inter-module communication is contract-first: schema (OpenAPI/GraphQL SDL / AsyncAPI / JSON Schema) defined, reviewed, versioned before code. Backwards compatibility required for MINOR/PATCH releases; breaking changes require MAJOR + migration notes. Automation surfaces (CLI scripts, maintenance jobs) follow text/JSON in → structured stdout out; stderr only for actionable errors. No hidden side effects.

### III. Test-First Quality Gates (NON-NEGOTIABLE)
Red-Green-Refactor required: write failing unit + contract tests → implement → refactor. Minimum gates: (a) 95% critical path coverage (auth, payments, data integrity), (b) mutation score target ≥ 70% on core domain packages, (c) contract tests for each public API version, (d) accessibility tests (axe/pa11y) for all user-facing flows, (e) performance guardrails baked into CI (Core Web Vitals + p95 API latency). A PR without failing initial tests is presumed untested.

### IV. Security, Privacy & Compliance by Construction
Threat model per new surface (API, queue, admin UI). Principle of least privilege enforced at code + infra (scoped tokens, role matrix). Mandatory: input validation at boundary, output encoding in templates, zero raw SQL string concatenation, centralized secrets management, audit logging for mutating operations, encryption in transit + at rest, PII classification & data minimization. Security tests (SAST, dependency audit, secret scan) must pass before merge. Privacy impact assessment required for new data collection fields.

### V. Observability, Performance & Reliability
Three pillars: structured logs (correlated via trace/span IDs), metrics (RED + USE), and traces for all network & queue spans. Error budgets defined per service; feature velocity throttled if budget exhausted. Performance SLOs: p95 API < 300ms (in-region), p99 < 800ms; Largest Contentful Paint < 2.5s on 3G Fast baseline; p95 critical job queue latency < 5s. Capacity planning doc required before doubling infra cost. Any regression >10% in a key SLO blocks release unless a documented waiver.

### VI. Versioning & Change Management
Semantic versioning for deployable artifacts & public schemas. Each change type mapped: PATCH = bug/security fix, MINOR = backward-compatible feature, MAJOR = breaking change (with deprecation policy + migration guide + automated detection where possible). No silent contract drift. Changelog entries mandatory and user-focused (problem → impact → upgrade note). Feature flags must have expiry/cleanup date.

### VII. Radical Simplicity & Maintainability
Prefer boring tech: introduce new language/runtime/build system only with written ROI (performance, security, or cost). Ban premature microservices—split only on sustained (a) independent scaling need, (b) conflicting latency profiles, (c) isolated compliance boundary, or (d) team ownership friction. YAGNI enforced: if not required to satisfy a user story, an SLO, or a security control, defer it. Remove dead code within two releases of deprecation.

## Web Application Constraints & Standards

### Architecture & Boundaries
- Frontend: progressive enhancement, SSR or static prerender for first meaningful view, hydration minimized (islands/components only where interactivity needed).
- Backend: stateless request handlers; persistent state only in designated data layer (databases/queues/object storage). No business logic in migrations or seeds.
- API styles allowed: REST (preferred) or GraphQL; events via message bus (topic naming: `domain.action.vX`).
- Shared types generated from single source-of-truth schemas.

### Security
- Mandatory headers: CSP (deny-all then allowlist), HSTS, X-Frame-Options: DENY, X-Content-Type-Options: nosniff, Referrer-Policy: strict-origin-when-cross-origin.
- OAuth2/OIDC or WebAuthn for authentication; no bespoke password storage logic.
- Rate limiting + abuse detection at edge; circuit breakers on downstream calls.
- Dependency freshness: high severity vuln patch SLA 24h, medium 7d.

### Privacy & Data Governance
- Data inventory maintained (field → purpose → retention → PII class).
- Right-to-erasure executable script tested quarterly.
- Analytics events: schema reviewed; no raw PII (use hashed/opaque IDs). Consent gating for non-essential tracking.

### Accessibility (A11y)
- WCAG 2.2 AA baseline; no keyboard trap; focus order deterministic.
- Color contrast auto-tested; semantic HTML enforced (lint rules).
- ARIA only when native element insufficient. Axe CI budget: 0 serious/critical violations.

### Performance
- Performance budget per page: JS < 170KB gzip initial, CSS < 70KB, third-party script total < 80KB.
- Images: responsive sources + compression; no layout shifts > 0.1 CLS.
- API endpoints must publish p50/p95 latency metrics; slow query logs actionable.

### Reliability & Scaling
- Blue/green or rolling deploy with auto health verification.
- Zero-downtime schema migrations (expand → migrate → contract pattern).
- Chaos drills quarterly (failure injection of DB, cache, and external API).
- Disaster recovery RPO ≤ 5m, RTO ≤ 30m documented & tested.

### Observability
- Log levels: TRACE (dev only), DEBUG (feature-flagged), INFO (state changes), WARN (user-impact w/ auto recovery), ERROR (requires on-call), FATAL (immediate page).
- Traces must include user/session correlation (GDPR-safe pseudonymous ID).
- Dashboards: 1) User journey funnel, 2) Core Web Vitals, 3) Error rate & budget, 4) Latency & saturation.

### CI/CD & Automation
- Single pipeline stages: validate (lint+type+unit) → test (unit+contract+integration) → quality gates (security+a11y+performance smoke) → package → deploy (staged) → verify (synthetic probes + canary metrics) → release.
- Rollback script idempotent and documented.

### Data & Caching
- All cache entries carry explicit TTL + ownership tag.
- No client-visible stale > 5m for mutable resources (stale-while-revalidate allowed if background refresh < 30s).
- Denormalization requires rebuild job + integrity check.

### Documentation & Knowledge
- Each module: README (purpose, contracts, invariants, failure modes, runbook link).
- Architecture Decision Records (ADR) for any choice with cost-of-reversal > 2d.
- On-call runbooks for every alert rule (WHAT happened, WHY it matters, HOW to mitigate, PREVENTION suggestions).

## Development Workflow & Quality Gates

1. Plan: Story → define acceptance criteria + contracts + a11y & perf impact notes.
2. Tests First: Author failing unit + contract + a11y (if UI) tests.
3. Implement: Minimal code to satisfy tests; observe architectural boundaries.
4. Review: Checklist (architecture, tests completeness, security, a11y, performance budgets, observability, docs updated). No self-approval.
5. Merge Gate: CI must be green; required statuses: lint/type, unit, mutation (core), contract, integration (if touched interfaces), a11y, performance smoke, security scan, dependency audit.
6. Deploy: Automated; canary < 10% traffic; monitor SLO & error budget for 15m before full rollout.
7. Post-Deploy Verification: Synthetic journey + error trend diff; create incident if regression over threshold.
8. Continuous Hardening: Weekly backlog refinement includes top security & debt item.

Quality Failure Policy: Repeated flaky test (>3 occurrences) blocks unrelated merges after 48h; must be fixed or quarantined with issue + owner + SLA.

## Governance

Hierarchy: This constitution > ADRs > Code comments > Individual preferences. Amendments require: (1) problem statement, (2) impact analysis, (3) migration/retrofit plan, (4) version increment, (5) checklist sync. Emergency security exceptions expire in ≤ 30 days.

Enforcement: PR template links to this file; reviewers must tag any violation. Persistent non-compliance escalated to tech lead. Metrics (flakiness, SLO adherence, debt burndown) reported quarterly.

Change Log: Each amendment appended with rationale + version bump. Deprecated sections kept for one MAJOR line with CLEAR deprecation box.

## Version & History

**Version**: 3.0.0 | **Ratified**: 2025-09-09 | **Last Amended**: 2025-09-09

Previous notable version: 2.1.1 (pre–web app expansion; lacked explicit security & a11y performance gates)

---
This constitution is a living guardrail. If a constraint blocks validated user value without a safer alternative, propose an amendment—do not create an undocumented exception.
