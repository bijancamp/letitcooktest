# Implementation Plan: VS Code Insider Podcast Landing & Episodes Experience

**Branch**: `001-modern-dark-themed` | **Date**: 2025-09-09 | **Spec**: `/specs/001-modern-dark-themed/spec.md`
**Input**: Feature specification from `/specs/001-modern-dark-themed/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
4. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
5. Execute Phase 1 → contracts, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, or `GEMINI.md` for Gemini CLI).
6. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
7. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
8. STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 7. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary
Deliver a modern dark-themed public web experience (landing page + episodes listing + episode detail with transcript + related episodes) for the VS Code Insider podcast. All data is mock/in-memory (no persistence, auth, or external streaming). Focus on accessibility, fast initial load (≤1s TTI mock), transcript segmentation, and deterministic related episode selection (shared speaker/tag ranking). Implementation will use a single Next.js (App Router) project with contract-first internal module boundaries (types + data providers) while keeping simplicity (no backend service split) per constitution.

## Technical Context
**Language/Version**: TypeScript (ES2023) / Node 18 LTS (Next.js baseline)  
**Primary Dependencies**: Next.js (App Router), React 18, TypeScript, ESLint (with a11y plugin), Testing Library + Jest (unit/UI), Playwright (integration/a11y), Axe (accessibility CI), Faker (mock content)  
**Storage**: None (in-memory mock data module exporting arrays/objects)  
**Testing**: Jest + React Testing Library (unit), Playwright (integration + a11y scans), Contract tests (Type-level + runtime schema validation using Zod)  
**Target Platform**: Web (SSR + static prerender) deployed to edge-capable platform (generic)  
**Project Type**: single (web)  
**Performance Goals**: Landing TTI ≤ 1s (mock), LCP ≤ 1.8s (desktop fast 3G baseline), Episode detail transcript initial render ≤ 500ms after navigation  
**Constraints**: JS initial payload ≤ 120KB gzip (below constitutional 170KB budget), CSS ≤ 50KB gzip, No client fetch for base episode list (pre-render)  
**Scale/Scope**: 20–30 mock episodes; up to ~10 speakers; simple in-memory relationships

No unresolved clarifications remain; decisions locked (see spec Confirmed Scope & Decisions).

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

**Simplicity**:
- Projects: 1 (single Next.js workspace) ✔
- Using framework directly: Yes (no abstraction layer) ✔
- Single data model module: Yes (episode/speaker/transcript) ✔
- Avoiding patterns: No repositories/UoW introduced ✔

**Architecture**:
- Feature boundaries as folders (domain/episodes, domain/speakers) ✔
- Internal contracts: TypeScript interfaces + Zod schemas ✔
- CLI need: Not required (pure web feature) — justified (no automation scope) ✔
- Library docs: Will generate README in domain modules (post Phase 1) ✔

**Testing (NON-NEGOTIABLE)**:
- TDD cycle planned (contract & failing tests first) ✔
- Commit order policy: enforce via task sequence ✔
- Order adaptation: Contract (Zod schema tests) → Integration (Playwright flows) → Unit (component logic) → A11y scans ✔
- Real dependencies? (In-memory only; acceptable due to No DB scope) ✔
- Integration tests for routing + related episodes logic planned ✔
- No implementation before failing tests: enforced via tasks ✔

**Observability**:
- Minimal structured console logging for navigation + related algorithm path (dev only) ✔
- Error boundaries for episode not found path ✔
- Additional telemetry deferred (no backend) ⚠ (acceptable for MVP)

**Versioning**:
- Feature version start: 0.1.0 (pre-release) ✔
- Build increments with changes (to be tracked in CHANGELOG.md) ✔
- Breaking change concept minimal (internal only) ✔

Initial Constitution Check: PASS (no violations needing Complexity Tracking)

## Project Structure

### Documentation (this feature)
```
specs/[###-feature]/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
# Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure]
```

**Structure Decision**: Option 1 (Single project) — Next.js root with `src/` containing domain slices.

## Phase 0: Outline & Research
Unknowns: NONE (spec decisions closed). Instead perform validation research to document rationale & alternatives for: (a) Next.js SSR vs static generation mix, (b) Transcript segmentation strategy, (c) Related episodes algorithm ranking fairness, (d) A11y testing tooling choice, (e) Performance budget alignment with constitution.

Planned research.md sections:
1. Rendering Strategy (Landing prerender, Episodes list prerender, Episode detail hybrid static + on-demand build) — choose static + incremental rationale.
2. Data Modeling Approach (plain TS modules vs mock API abstraction) — choose modules.
3. Related Algorithm (speaker weight > tag weight > recency) — fairness & tie-breaking.
4. Accessibility Tooling (Axe + Playwright integration) — CI gating.
5. Performance Budgets (map feature budgets to constitutional max budgets & justify tighter JS/CSS limits).

All will use Decision/Rationale/Alternatives template.

**Output**: research.md capturing each decision (no open questions).

## Phase 1: Design & Contracts
*Prerequisite: research.md complete*

Adaptation (no backend service): internal contracts as TypeScript + Zod schemas; minimal pseudo-API layer to preserve contract-first discipline.

1. Data Model (`data-model.md`): Define Episode, Speaker, TranscriptSegment, RelatedComputation with fields + invariants (e.g., unique episode numbers, segments sequential, diversity enforcement rule) + validation summary.
2. Contracts (`/contracts/`):
   - JSON Schema / Zod definitions exported: EpisodeSchema, SpeakerSchema, TranscriptSegmentSchema, RelatedEpisodeSchema.
   - A pseudo endpoint spec file describing consumer expectations:
     * GET /episodes (list summary shape)
     * GET /episodes/:slug (detail including segmented transcript + related)
     * GET /episodes/:slug/related (subset — used for internal test; still pure mock)
3. Contract Tests (failing initially):
   - Validate sample mock dataset against schemas (will fail until dataset added).
   - Related algorithm test ensures ordering & diversity.
   - Transcript segmentation test expects collapse rule parameters.
4. Integration Test Scenarios (Playwright draft): landing rendering, episodes navigation flow, transcript expansion, related click navigation, keyword filter usage.
5. Quickstart (`quickstart.md`): Steps to install deps, run dev server, run tests (contract first), view a11y report.
6. Agent Context File: Add new tech references (Next.js, Zod, Playwright, Axe) via update script (if present) — else manual README note.

**Output**: data-model.md, contracts schemas, failing contract/unit tests, quickstart.md.

## Phase 2: Task Planning Approach
*Scope description only (tasks.md generated later by /tasks)*

**Task Generation Strategy**:
- Derive tasks from: schemas (Episode/Speaker/TranscriptSegment), dataset creation, related algorithm, UI pages, accessibility validation, performance budget guard tests.
- Each contract schema → create schema + failing validation test task [P]
- Each user flow (landing load, browse, open episode, expand transcript, navigate related, filter episodes) → integration test first.
- Dataset creation before UI implementation; UI blocked until schema tests green.
- Add tasks for: a11y CI test scaffold, performance smoke test, dark theme token definition.

**Ordering Strategy**:
1. Initialize project scaffolding (Next.js + config) (foundation)
2. Schema + contract tests (fail) [parallelizable]
3. Mock dataset module
4. Related algorithm + tests (fail initially)
5. Episode list & detail contract adapter (to make tests pass)
6. Integration tests (navigate flows) (failing until pages stubbed)
7. Page components minimal to satisfy flows
8. A11y + performance tests
9. Enhance UI (visual polish) after tests pass

**Estimated Output**: ~28 tasks (10 contracts/tests, 6 data/model, 6 UI flow, 3 a11y/perf, 3 polish).

No task includes premature optimization; microservice or API split explicitly out-of-scope.

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following constitutional principles)  
**Phase 5**: Validation (run tests, execute quickstart.md, performance validation)

## Complexity Tracking
No deviations; table omitted.


## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [x] Phase 0: Research complete (/plan command)
- [x] Phase 1: Design complete (/plan command)
- [x] Phase 2: Task planning complete (/plan command - describe approach only)
- [ ] Phase 3: Tasks generated (/tasks command)
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS
- [x] Post-Design Constitution Check: PASS
- [x] All NEEDS CLARIFICATION resolved
- [x] Complexity deviations documented (none required)
 - [x] Required Phase 0/1 artifacts present (research.md, data-model.md, contracts/, quickstart.md)

---
*Based on Constitution v3.0.0 - See `/memory/constitution.md`*
