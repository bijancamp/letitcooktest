# Feature Specification: VS Code Insider Podcast Landing & Episodes Experience

**Feature Branch**: `001-modern-dark-themed`  
**Created**: 2025-09-09  
**Status**: Draft  
**Input**: User description: "Modern dark-themed VS Code Insider podcast landing site with featured speakers, 20 mock episodes with transcripts, episode discovery & related episode navigation."

## Execution Flow (main)
```
1. Parse user description from Input
   → If empty: ERROR "No feature description provided"
2. Extract key concepts from description
   → Identify: actors, actions, data, constraints
3. For each unclear aspect:
   → Mark with [NEEDS CLARIFICATION: specific question]
4. Fill User Scenarios & Testing section
   → If no clear user flow: ERROR "Cannot determine user scenarios"
5. Generate Functional Requirements
   → Each requirement must be testable
   → Mark ambiguous requirements
6. Identify Key Entities (if data involved)
7. Run Review Checklist
   → If any [NEEDS CLARIFICATION]: WARN "Spec has uncertainties"
   → If implementation details found: ERROR "Remove tech details"
8. Return: SUCCESS (spec ready for planning)
```

---

## ⚡ Quick Guidelines
- ✅ Focus on WHAT users need and WHY
- ❌ Avoid HOW to implement (no tech stack, APIs, code structure)
- 👥 Written for business stakeholders, not developers

### Section Requirements
- **Mandatory sections**: Must be completed for every feature
- **Optional sections**: Include only when relevant to the feature
- When a section doesn't apply, remove it entirely (don't leave as "N/A")

### For AI Generation
When creating this spec from a user prompt:
1. **Mark all ambiguities**: Use [NEEDS CLARIFICATION: specific question] for any assumption you'd need to make
2. **Don't guess**: If the prompt doesn't specify something (e.g., "login system" without auth method), mark it
3. **Think like a tester**: Every vague requirement should fail the "testable and unambiguous" checklist item
4. **Common underspecified areas**:
   - User types and permissions
   - Data retention/deletion policies  
   - Performance targets and scale
   - Error handling behaviors
   - Integration requirements
   - Security/compliance needs

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story
As a curious developer visiting the VS Code Insider podcast site, I want to quickly understand what the show offers, see featured conversations with notable speakers, browse or search episodes, and open an episode page that provides a full transcript and related recommendations so I can decide what to listen to next without friction.

### Acceptance Scenarios
1. **Given** a first-time visitor on the landing page, **When** the page loads, **Then** they see: site title, tagline, 3–5 featured speaker conversation cards (image/name/title/episode title teaser), a call to explore all episodes, and a dark theme interface consistent across viewport sizes.
2. **Given** a visitor on the landing page, **When** they click "View All Episodes", **Then** they are taken to an Episodes listing page showing at least 20 episode summaries (title, number, publish date, short abstract, speakers) with pagination or lazy load.
3. **Given** a visitor on the Episodes page, **When** they select an episode card, **Then** they navigate to that Episode page displaying full title, publish metadata, participating speakers, embedded player placeholder, complete transcript (mock), and a Related Episodes section (3–6 items) based on shared speakers or topic tags.
4. **Given** a visitor on an Episode page, **When** they click a related episode, **Then** the new episode page loads with updated transcript and recommendations.
5. **Given** a visitor scrolling the landing page on mobile, **When** they reach featured speakers, **Then** cards are readable, accessible, and maintain contrast without horizontal scroll.
6. **Given** a visitor using keyboard navigation, **When** they tab through interactive elements, **Then** focus order follows visual hierarchy and all actionable elements are reachable.

### Edge Cases
- Episode has a speaker appearing in multiple episodes: related section prioritizes diversity (no more than 2 episodes featuring the same primary speaker unless <3 total candidates exist).
- Transcript extremely long (e.g., >15k characters): initial view shows first 8 segments (~first 2,500–3,000 characters) with an "Expand Full Transcript" control (progressive disclosure).
- Missing speaker image asset: fallback avatar with initials displayed.
- User opens direct deep link to an episode referencing a speaker not on landing: page still loads correctly.
- Network latency / partial transcript load: display skeleton state until segments ready (target <500ms render after data available).
- Mobile very narrow (<360px): single-column layout; no horizontal scroll.

### Functional Requirements
- **FR-001**: System MUST provide a public landing page presenting site title, tagline, and a featured speakers/conversations section.
- **FR-002**: System MUST display 3–5 featured conversation cards selectable for navigation to their episode pages.
- **FR-003**: System MUST provide an Episodes listing page with at least 20 mock episodes initially available.
- **FR-004**: System MUST show each episode summary with: episode number, title, publish date, 1–2 sentence abstract, primary speakers.
- **FR-005**: System MUST allow navigation from any episode summary or featured card to its detailed episode page.
- **FR-006**: System MUST render an Episode detail page containing: full title, episode number, publish date, duration (mock), speaker list (names + roles), full transcript (segmented), and related episodes list.
- **FR-007**: System MUST compute related episodes using shared speaker or shared tag criteria (minimum one match) excluding the current episode.
- **FR-008**: System MUST ensure related episodes list size between 3 and 6 when enough qualifying episodes exist; if fewer qualify, show all available (minimum 1 if exists).
- **FR-009**: System MUST support keyboard-only navigation across landing, episodes list, and episode detail pages.
- **FR-010**: System MUST maintain WCAG AA contrast in dark theme for text, buttons, and interactive states.
- **FR-011**: System MUST provide fallback visuals (placeholder avatar) when a speaker image asset is unavailable.
- **FR-012**: System MUST show a visible loading state if transcript retrieval exceeds 300ms.
- **FR-013**: System MUST prevent duplicate listing of the current episode in related episodes.
- **FR-014**: System MUST present consistent primary navigation accessible from all pages (e.g., Home / Episodes).
- **FR-015**: System MUST segment long transcripts and initially collapse after the first 8 segments with an expand control.
- **FR-016**: System MUST default episode ordering to newest → oldest by publish date.
- **FR-017**: System MUST provide a lightweight client-side keyword filter (title, speaker name, tag) on the Episodes listing page.
- **FR-018**: System MUST provide at least 20 unique mock episode data objects for development & validation.
- **FR-019**: System MUST allow direct deep linking to any episode page via stable URL structure.
- **FR-020**: System MUST expose accessibility roles/labels for interactive cards and navigation landmarks.
- **FR-021**: System MUST rank related episodes by (1) shared speaker count, (2) tag overlap count, (3) recency; enforce diversity cap of max 2 entries sharing the same primary speaker unless insufficient pool (<3 candidates).

### Key Entities *(include if feature involves data)*
- **Episode**: number, title, abstract, publish date, duration (mock), speakers (references), topic tags, transcript segments (references), derived related episodes, hero image (optional), slug.
- **Speaker**: name, role/title, optional short bio, avatar reference, associated episodes list.
- **Transcript Segment**: sequence index, episode reference, speaker name, text (≤1200 chars), approximate character length.
- **Related Episode Relationship (Derived)**: ranking tuple (shared_speakers, tag_overlap_count, recency_rank) computed on demand.

### Confirmed Scope & Decisions
1. Public-only experience; no auth.
2. Audio integration deferred; placeholder only.
3. Minimal keyword filter included; full-text transcript search deferred.
4. Transcripts stored as segmented plain text with speaker attribution; no rich formatting.
5. Featured speakers curated manually; fallback selects top 3–5 by episode count then recency.
6. Related diversity rule enforced (see FR-021).
7. Long transcript progressive disclosure (see FR-015).
8. Tag overlap threshold: ≥1 shared tag qualifies.
9. Baseline performance targets: Landing TTI <1s (mock data), transcript render <500ms after data ready.
10. Accessibility baseline: All interactive elements keyboard reachable with visible focus outline; segments use semantic containers for assistive tech navigation.

---

## Review & Acceptance Checklist
*GATE: Automated checks run during main() execution*

### Content Quality
- [x] No implementation details (languages, frameworks, APIs)
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders
- [x] All mandatory sections completed

### Requirement Completeness
- [x] No [NEEDS CLARIFICATION] markers remain
- [x] Requirements are testable and unambiguous  
- [x] Success criteria are measurable
- [x] Scope is clearly bounded
- [x] Dependencies and assumptions identified

---

## Execution Status
*Updated by main() during processing*

- [ ] User description parsed
- [ ] Key concepts extracted
- [ ] Ambiguities marked
- [ ] User scenarios defined
- [ ] Requirements generated
- [ ] Entities identified
- [ ] Review checklist passed

---
