# Phase 0 Research: VS Code Insider Podcast Feature

## 1. Rendering Strategy
- **Decision**: Use Next.js static generation (pre-render) for landing & episodes index; static generation with on-demand (all known episodes at build) for episode detail pages.
- **Rationale**: All data is static mock; SSR would add runtime cost without benefit. Static improves TTI and aligns with performance goals.
- **Alternatives Considered**:
  - Full SSR: Rejected (unnecessary complexity, slower first byte).
  - Client-only fetch: Rejected (delays content, hurts SEO & a11y skeleton exposure).

## 2. Data Modeling Approach
- **Decision**: Single domain data module exporting typed arrays/objects + Zod schemas for validation.
- **Rationale**: Contract-first discipline without network overhead; easy to evolve to API later.
- **Alternatives**:
  - Mock REST layer: Rejected (extra boilerplate, no user value).
  - GraphQL schema: Rejected (no multi-client need yet).

## 3. Related Episodes Algorithm
- **Decision**: Rank by shared speakers (desc) → tag overlap (desc) → recency (newest first) with diversity cap (max 2 per dominant speaker unless <3 results).
- **Rationale**: Speaker continuity provides strongest relevance; tags refine; recency keeps content fresh; diversity avoids clustering.
- **Alternatives**:
  - Tag-first: Rejected (speaker recognition primary user heuristic).
  - Random shuffle: Rejected (inconsistent relevance, hard to test).

## 4. Transcript Segmentation
- **Decision**: Segment by speaker turns; split long turns at sentence boundaries ≤1200 chars; show first 8 segments collapsed with expand.
- **Rationale**: Improves scanning, a11y navigation, reduces initial DOM weight.
- **Alternatives**:
  - Full unsegmented text: Rejected (hard to navigate, performance cost).
  - Fixed chunk size ignoring speaker: Rejected (loses context, reduces clarity).

## 5. Accessibility Tooling
- **Decision**: Use Axe via Playwright (per-page scan) + Jest a11y unit checks for key components.
- **Rationale**: Combines integration context with fast feedback; supports gating (0 serious/critical allowed).
- **Alternatives**:
  - Cypress + Axe: Rejected (Playwright already chosen for cross-browser, simpler stack needed).
  - Manual only: Rejected (non-compliant with constitution test-first a11y gate).

## 6. Performance Budgets
- **Decision**: Internal stricter budgets (JS ≤120KB gzip, CSS ≤50KB) under constitution caps.
- **Rationale**: Provides headroom for future features while meeting LCP target.
- **Alternatives**:
  - Match constitutional max: Rejected (no buffer, risk regression).

## 7. Testing Strategy Mapping
- **Decision**: Contract (schema validation) → Integration (Playwright flows + a11y) → Unit (pure functions e.g., related ranking) → Visual polish.
- **Rationale**: Validates core user value early, ensures accessibility not deferred.
- **Alternatives**: Unit-first: Rejected (misaligns with user-centric flow & contract-first principle).

## 8. Observability (MVP)
- **Decision**: Minimal console structured logs (development) + lightweight timing instrumentation in dev mode for transcript render.
- **Rationale**: No backend; adding full tracing premature.
- **Alternatives**: Custom telemetry layer: Rejected (scope creep).

## 9. Risk Register
- Potential scope creep (search, audio streaming) → Mitigation: Explicit deferral.
- Overfitting related algorithm early → Mitigation: Simple deterministic ranking; revisit only w/ user feedback.
- a11y drift during styling → Mitigation: CI a11y gating from first styling commit.

## 10. Open Items
- None (all clarifications resolved). Future enhancements tracked separately once real data or streaming needs arise.

---
All decisions align with Constitution v3.0.0 (Modular, Test-First, Simplicity).
