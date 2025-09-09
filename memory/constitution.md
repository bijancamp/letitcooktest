# Web Application Platform Constitution
<!-- Foundational operating agreement for architecture, delivery, quality & governance -->

## Core Principles

### I. Domain & Contract First
All externally consumed behavior (HTTP/GraphQL/WebSocket APIs, events, queues, CLI utilities) begins with an explicitly reviewed contract (OpenAPI / GraphQL SDL / AsyncAPI / JSON Schema). No implementation starts before:
1. Contract PR merged
2. Backwards compatibility strategy defined (additive, deprecated, or breaking w/ migration plan)
3. Success metrics & SLOs identified

### II. Test-Driven & Continuous Quality (NON‑NEGOTIABLE)
Red‑Green‑Refactor strictly applied. Each change must introduce/extend tests that would fail pre-change and pass post-change. Required layers:
- Unit: 90%+ critical path coverage, fast (<300ms per test file target)
- Contract: Schemas validated in CI against published canonical spec
- Integration: Service boundary & data-store interactions
- End-to-End (Smoke & Journey): Critical user paths (auth, checkout, data mutation) executed on ephemeral environments
Failing tests block merge; flaky tests quarantined within 24h or removed.

### III. Security & Privacy by Default
Security is a build-time and runtime invariant:
- Principle of least privilege (infra, DB, service tokens)
- Secrets managed only via vault / parameter store (never in repo, env vars only injected at runtime)
- Mandatory input validation at trust boundaries; output encoding/escaping
- Zero tolerance for known critical/high CVEs > 7 days old
- Data classification tags required for new persisted fields (PII / Sensitive / Public)
- Pseudonymization/anonymization strategies documented for analytics

### IV. Observability & Operability
Every user-impacting feature ships with: structured logs (JSON), metrics (RED + custom domain KPIs), traces (W3C Trace Context), and health/readiness endpoints. Error budgets define release discipline. Pages/alerts only on actionable signals (SLO burn-rate, saturation, security anomalies). All incidents produce a blameless postmortem within 48h.

### V. Performance, Resilience & Scalability
Targets are defined before implementation. Default baselines (adjust per service):
- P50 < 150ms, P95 < 400ms for synchronous API endpoints under expected peak load
- Horizontal scaling > vertical scaling preference; stateless services; idempotent handlers
- Circuit breakers & exponential backoff around remote calls
- Database queries must have explain plans for new complex queries; slow query budget tracked
- Caching (content, application, DB read replicas) justified with hit-rate metric goals

### VI. Accessibility & Inclusive UX
User-facing web surfaces meet WCAG 2.1 AA. Automated axe-core scans in CI for pages/components. Keyboard navigation and semantic markup mandatory. Color contrast enforced via design tokens linting.

### VII. Simplicity & Evolutionary Architecture
Prefer composable, boring solutions over framework overreach. Introduce new infrastructure or runtime tech only with an ADR (Architecture Decision Record) referencing comparative evaluation & operational cost.

## Technical & Operational Constraints

### Stack Requirements
- Frontend: TypeScript, React (functional components, hooks), CSS Modules or Tailwind (one per repo), build via Vite/Next.js (SSR only when required by SEO or dynamic personalization)
- Backend: TypeScript / Node.js (LTS), lightweight framework (Fastify/Express) behind a thin HTTP adapter
- Data: Postgres primary (normalized), Redis (ephemeral caching, rate limiting), S3-compatible object storage
- APIs: OpenAPI 3.x for REST; GraphQL reserved for aggregate composition use cases; Async events use CloudEvents JSON schema
- Infrastructure: IaC via Terraform; immutable container images (distroless preferred); deployments via GitOps or progressive delivery (blue/green or canary)

### Security & Compliance
- OWASP Top 10 regression tests automated (DAST + SAST gates)
- Mandatory dependency vulnerability scanning (SCA) each pipeline run
- Rate limiting + WAF in front of public ingress
- JWT / OAuth2.1 for user auth; service-to-service mTLS + short-lived tokens (SPIFFE/SPIRE or equivalent)
- Audit log for authenticated data mutations (tamper evident, append-only)
- Privacy review for features touching PII; data retention policies codified in schema comments

### Data & Schema Governance
- All schema migrations are forward-only, reversible via new migration (no manual DB edits)
- Breaking DB changes require a 2-step expand/contract pattern
- Naming conventions: snake_case DB, camelCase in code, kebab-case for resource path segments

### Performance & Capacity Management
- Load test before major release or +25% traffic forecast
- Auto-scaling policies version-controlled; scaling events observable
- CDN for static assets, immutable cache headers with content hash

### Observability Policies
- Minimum telemetry: request_id, user_id (if authenticated), trace_id, feature_flag_state, latency_ms, outcome (success|error_code)
- Error classification: user_error / system_error / third_party / security
- No log PII beyond allowable classification; redaction filters enforced in ingestion pipeline

## Delivery Workflow & Quality Gates

### Branching & Integration
- Trunk-based with short-lived feature branches (<48h ideal)
- Every PR: linked issue/spec, contract diff (if API), test evidence
- Auto-merge allowed only when: all checks green + required reviews met + no governance label blocks

### Required Pipeline Stages (Fail Fast)
1. Static Checks: lint (ESLint), types (tsc --noEmit), formatting (Prettier --check)
2. Unit Tests: parallel shards, coverage threshold enforced
3. Contract Validation: diff against main; breaking requires override label + approved ADR
4. Security: SAST, dependency scan, secret scan
5. Build & Image Scan: SBOM generated, signature (cosign) attached
6. Integration Tests: ephemeral environment w/ isolated DB
7. E2E Smoke: core journeys (login, create/update entity, permission boundary)
8. Performance Smoke: subset load (e.g., k6 1-min ramp) gating egregious regressions
9. Policy as Code: Terraform plan compliance (OPA/Conftest) & drift detection

### Release Management
- Semantic Versioning (MAJOR.MINOR.PATCH)
- Release notes auto-generated from conventional commits; manual curation for user-facing changes
- Feature flags for all non-trivial changes; default off until metrics & error budgets confirm health
- Rollback policy: <= 10 min MTTR target; previous N images retained; DB revert = forward migration path

### Documentation Requirements
- Each service: README (purpose, APIs, run, health, SLOs), ADR folder, OpenAPI spec, runbook (oncall actions, dashboards, alerts)
- Public docs generated automatically from source contracts

## Governance

### Authority & Amendments
This Constitution overrides ad-hoc practice. Amendments require:
1. Draft ADR referencing affected sections
2. Impact analysis (risk, migration, deprecation timeline)
3. Approval: ≥2 senior reviewers (architecture & security) + product sign-off for user-impacting changes
4. Version bump (MINOR for additive, MAJOR for structural/breaking)

### Compliance & Enforcement
- Every PR template includes a Constitution checklist; unchecked items require explicit justification label
- Automated policy bots flag violations (naming, coverage, spec drift). Red = merge blocked
- Incident postmortems must map contributing factors to principle adherence (or lack thereof)

### Sunset & Deprecation
- Deprecations require: announcement (CHANGELOG + internal comms), dual-run or compatibility layer, observability of old vs new usage, removal date

### Exceptions
Temporary exceptions time-boxed (max 30 days) with owner + remediation task. Extensions need re-approval.

**Version**: 1.0.0 | **Ratified**: 2025-09-09 | **Last Amended**: 2025-09-09
<!-- Update version and dates upon amendment -->
