# Data Model: VS Code Insider Podcast

## Overview
Static, mock, in-memory dataset. All entities validated by Zod schemas. No persistence layer.

## Entities
### Episode
Fields:
- number (int > 0, unique)
- slug (kebab-case string, unique)
- title (string 5-120 chars)
- abstract (string ≤ 240 chars)
- publishDate (ISO date string)
- durationMinutes (int 1–180)
- speakers (array<SpeakerRef> length ≥1)
- tags (array<Tag> length 1–6 unique)
- transcriptSegments (array<TranscriptSegmentRef> length ≥8)
- heroImage (optional string URL or null)

Invariants:
- Unique (number, slug)
- Publish date not in future (for mock: allow <= today)
- Transcript segments all belong to same episode number/slug

### Speaker
Fields:
- id (uuid-like string)
- name (string 2–60)
- role (string 2–80)
- bio (optional string ≤ 400)
- avatar (optional URL)
- episodeNumbers (array<int> length ≥1)

Invariants:
- id unique
- avatar optional; fallback generated if absent

### TranscriptSegment
Fields:
- id (uuid-like string)
- episodeNumber (int, FK Episode.number)
- index (int >=0 sequential without gaps)
- speakerId (FK Speaker.id)
- text (string 1–1200 chars)
- charCount (derived: length(text))

Invariants:
- index continuous (0..n)
- text trimmed, no leading/trailing whitespace

### RelatedComputation (Derived, not stored)
Inputs:
- current episode
- candidate episodes
Algorithm:
1. Score each candidate: speakersScore = sharedSpeakersCount
2. tagScore = sharedTagCount
3. recencyScore = inverse publish date rank (newer higher)
4. Sort by speakersScore desc, tagScore desc, recencyScore desc
5. Enforce diversity: limit to 2 with same primary speaker (first listed speaker) unless <3 total results
6. Take first 3–6

### Tag
Simple string (kebab-case, 2–24 chars). Limited vocabulary curated manually.

## Validation Summary
- Zod schemas enforce field constraints.
- Dataset loaded at startup; contract tests validate all objects.
- Any schema failure blocks build (test-first enforcement).

## Error Handling (Logical Only)
- Missing slug lookup → NotFound sentinel handled by page fallback.
- Related computation with <1 candidate → returns empty array (UI hides section).

## Future Evolution (Not In Scope)
- Real persistence (DB or CMS)
- Internationalization (localized fields)
- Rich text transcripts (markup + timestamps)

---
Conforms to Constitution: modular (single data module), contract-first (schemas), simplicity (no unnecessary abstraction).
