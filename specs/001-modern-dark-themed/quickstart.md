# Quickstart: VS Code Insider Podcast Feature

## 1. Install & Bootstrap
```
# (Placeholder - project scaffolding added in implementation phase)
npm install
```

## 2. Run Contract Tests (should FAIL initially until data + schemas implemented)
```
npm test -- --selectProjects contracts
```

## 3. Run Integration Tests (Playwright)
```
npx playwright test
```

## 4. Development Server
```
npm run dev
```

Open http://localhost:3000

## 5. Accessibility Scan
```
npx playwright test --grep @a11y
```

## 6. Performance Smoke (after pages exist)
Use Lighthouse (CI or local) – ensure: LCP <1.8s (mock), JS <120KB gzip, CSS <50KB gzip.

## 7. Transcript Expansion Behavior
1. Visit any episode detail
2. Confirm only first 8 segments visible
3. Click Expand → all segments render
4. Scroll bottom → collapse control available

## 8. Related Episodes Diversity Check
1. Open episode with >5 potential related
2. Confirm max 2 share identical first speaker

## 9. Keyword Filter
1. Navigate to Episodes list
2. Enter part of speaker name → list narrows
3. Clear filter → list resets

## 10. Updating Mock Data
- Modify domain data module
- Run contract tests; ensure pass
- Update related algorithm tests if ranking rules evolve

---
All steps align with Constitution (Test-First, Performance, Accessibility).
