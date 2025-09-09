# Contracts (Internal Mock Schemas)

These contracts define the shape of internal data consumed by UI components. Though no external API exists, contract-first discipline is preserved.

## Schemas
- EpisodeSchema
- SpeakerSchema
- TranscriptSegmentSchema
- RelatedEpisodeSchema

## Pseudo Endpoints (Conceptual)
- GET /episodes → EpisodeSummary[] (subset fields: number, slug, title, abstract, publishDate, speakers[], tags[])
- GET /episodes/:slug → EpisodeDetail (Episode + transcriptSegments[] + related[])
- GET /episodes/:slug/related → RelatedEpisode[]

## Versioning
Current version: 0.1.0 (pre-release). Breaking contract changes will increment MAJOR.

## Testing Strategy
- Zod parsing tests ensure dataset compliance.
- Related algorithm tests assert ordering & diversity invariants.

---
Aligned with Constitution (Contract-First, Simplicity).
