# Tasks: VS Code Insider Podcast Landing & Episodes Experience

**Input**: Design documents from `/specs/001-modern-dark-themed/`
**Prerequisites**: plan.md (required), research.md, data-model.md, contracts/, quickstart.md

## Execution Flow (main)
```
1. Confirm plan.md present → extract stack (Next.js, TS, Zod, Playwright, Axe)
2. Load data-model.md → entities (Episode, Speaker, TranscriptSegment, RelatedComputation)
3. Load contracts/ → schema list (README placeholder) → derive schema files to create
4. Load research.md → decisions (rendering, segmentation, related ranking, performance budgets)
5. Generate tasks (Setup → Tests (failing) → Core Models/Schemas → Algorithm → UI Pages → A11y/Perf → Docs/Polish)
6. Ensure all tests precede implementation
7. Mark parallel [P] where different files & no deps
8. Output tasks.md
```

## Phase 3.1: Setup
- [ ] T001 Initialize Next.js project (App Router) in repo root (`package.json`, `next.config.js`, `tsconfig.json`, `src/app/` baseline)
- [ ] T002 Add dependencies (next, react, react-dom, typescript, zod, @testing-library/react, @testing-library/jest-dom, jest, ts-jest, playwright, @axe-core/playwright, eslint + a11y plugins) and configure scripts
- [ ] T003 Create project lint & format config (`.eslintrc.cjs`, `.prettierrc`) with a11y + import rules
- [ ] T004 Configure Jest + setupTests.ts and separate `tests/` structure (unit/, contract/, integration/)
- [ ] T005 Add Playwright config (`playwright.config.ts`) with test tags (@a11y)
- [ ] T006 Define performance budget config placeholder (`perf-budget.json`) referencing JS/CSS limits
- [ ] T007 [P] Add base README with project overview & link to spec/plan
- [ ] T008 [P] Add `.nvmrc` / `.tool-versions` specifying Node 18 LTS

## Phase 3.2: Tests First (Contract + Integration) – MUST FAIL INITIALLY
Contract Schema Test Stubs (will reference non-existing schema modules until implemented):
- [ ] T009 [P] Failing test: Episode schema validation (`tests/contract/episode.schema.test.ts`)
- [ ] T010 [P] Failing test: Speaker schema validation (`tests/contract/speaker.schema.test.ts`)
- [ ] T011 [P] Failing test: Transcript segment schema validation (`tests/contract/transcript-segment.schema.test.ts`)
- [ ] T012 [P] Failing test: Related episode schema & ranking contract (`tests/contract/related.schema.test.ts`)

Integration / Flow Tests (stub pages not yet implemented):
- [ ] T013 [P] Failing Playwright test: Landing page renders featured section (`tests/integration/landing.spec.ts`)
- [ ] T014 [P] Failing Playwright test: Episodes list navigation & ordering (`tests/integration/episodes-list.spec.ts`)
- [ ] T015 [P] Failing Playwright test: Episode detail transcript collapse/expand (`tests/integration/episode-transcript.spec.ts`)
- [ ] T016 [P] Failing Playwright test: Related episodes diversity & ranking (`tests/integration/related-episodes.spec.ts`)
- [ ] T017 [P] Failing Playwright test: Keyword filter narrows results (`tests/integration/filter.spec.ts`)
- [ ] T018 [P] Failing Playwright test: Accessibility baseline (axe scan no serious) (`tests/integration/a11y.spec.ts`)

Algorithm Unit Tests (initially fail):
- [ ] T019 [P] Failing unit test: related ranking logic ordering + diversity (`tests/unit/related-ranking.test.ts`)
- [ ] T020 [P] Failing unit test: transcript segmentation rules (`tests/unit/transcript-segmentation.test.ts`)

## Phase 3.3: Core Model & Schema Implementation
- [ ] T021 Implement Zod schemas (`src/domain/schemas.ts`) for Episode, Speaker, TranscriptSegment, RelatedEpisode
- [ ] T022 Create mock dataset generator (static file) (`src/domain/data/episodes.ts`) with 20+ episodes
- [ ] T023 Create speakers dataset (`src/domain/data/speakers.ts`) deriving episodes linkage
- [ ] T024 Implement transcript segmentation utility (`src/domain/transcript/segment.ts`)
- [ ] T025 Implement related algorithm (`src/domain/related/computeRelated.ts`)
- [ ] T026 Export contract validation helpers (`src/domain/contracts/index.ts`)
- [ ] T027 Fill perf budget config with placeholders for CI check (`scripts/check-performance.js` placeholder) [P]
- [ ] T028 Add accessibility test helper (`tests/helpers/a11y.ts`) [P]

## Phase 3.4: UI Pages & Components
- [ ] T029 Create layout shell + dark theme tokens (`src/app/theme.css` & `src/app/layout.tsx`)
- [ ] T030 Landing page with featured speakers/cards (`src/app/page.tsx`)
- [ ] T031 Episodes list page with keyword filter (`src/app/episodes/page.tsx`)
- [ ] T032 Episode detail route w/ slug dynamic page (`src/app/episodes/[slug]/page.tsx`)
- [ ] T033 Transcript segment component & expand logic (`src/components/Transcript.tsx`)
- [ ] T034 Related episodes component enforcing diversity (`src/components/RelatedEpisodes.tsx`)
- [ ] T035 Speaker avatar component w/ fallback initials (`src/components/SpeakerAvatar.tsx`) [P]
- [ ] T036 Navigation component (Home/Episodes) (`src/components/Nav.tsx`) [P]

## Phase 3.5: Make Tests Pass / Integration Wiring
- [ ] T037 Wire dataset into list & detail pages (static generation + props)
- [ ] T038 Apply related algorithm in episode detail
- [ ] T039 Wire transcript segmentation + collapse/expand
- [ ] T040 Implement keyword filter logic (client-side) in episodes list
- [ ] T041 Add a11y roles/labels & focus order refinements
- [ ] T042 Add performance measurement script stub (Lighthouse CI integration placeholder)

## Phase 3.6: Polish & Quality Gates
- [ ] T043 Add unit tests for navigation accessibility edge cases (`tests/unit/nav-a11y.test.ts`) [P]
- [ ] T044 Add unit tests for avatar fallback logic (`tests/unit/avatar-fallback.test.ts`) [P]
- [ ] T045 Performance smoke test script implementation (`scripts/check-performance.js`) filling budgets
- [ ] T046 Add mutation test placeholder or note (out-of-scope now, documented) [P]
- [ ] T047 Documentation: domain README + related algorithm notes (`src/domain/README.md`)
- [ ] T048 Update root README with Quickstart and architecture summary
- [ ] T049 Add CHANGELOG.md entry (0.1.0 initial feature) [P]
- [ ] T050 Final a11y regression run & mark tasks complete

## Dependencies
- T001–T008 before any test tasks
- T009–T020 must fail before T021+ implementation
- T021 depends on T009–T012
- T022–T024 depend on schemas (T021)
- T025 depends on datasets (T022, T023) & schemas
- T026 depends on schemas
- T029–T036 depend on T021–T024 (schemas + data + utils)
- T037–T042 depend on all prior model & page scaffolds
- T043–T050 after core wiring

## Parallel Execution Examples
```
# Example: Run all contract schema tests in parallel after setup
T009, T010, T011, T012

# Example: Run flow integration tests in parallel (independent files)
T013, T014, T015, T016, T017, T018

# Example: Run algorithm unit tests in parallel
T019, T020

# Example: Parallel component tasks
T035, T036
```

## Validation Checklist
- [ ] All schemas have failing tests before impl (T009–T012)
- [ ] All entities have tasks (Episode, Speaker, TranscriptSegment, RelatedComputation)
- [ ] All user flows mapped to integration tests (landing, list, detail, related, filter, a11y)
- [ ] Related algorithm ordering & diversity tested (T016, T019)
- [ ] Transcript segmentation tested (T015, T020)
- [ ] Accessibility gating present (T018, T041, T050)
- [ ] Performance budget tasks present (T006, T042, T045)
- [ ] Documentation tasks included (T047, T048)
- [ ] Versioning & changelog task included (T049)

---
Generated from plan & spec; adheres to Constitution v3.0.0 (Simplicity, Contract-First, Test-First).
