---
phase: 1
slug: foundation-and-manual-assets
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-04-20
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.1.4 (backend + frontend) |
| **Config file** | None — Wave 0 creates vitest.config.ts in both backend/ and frontend/ |
| **Quick run command** | `cd backend && npx vitest run --reporter=verbose` |
| **Full suite command** | `cd backend && npx vitest run && cd ../frontend && npx vitest run` |
| **Estimated runtime** | ~10 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd backend && npx vitest run --reporter=verbose` (or frontend equivalent)
- **After every plan wave:** Run full suite (backend + frontend)
- **Before `/gsd-verify-work`:** Full suite must be green
- **Max feedback latency:** 10 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 1-01-01 | 01 | 1 | DATA-01 | T-1-01 / N/A | class-validator DTOs enforce required fields | unit | `cd backend && npx vitest run --reporter=verbose` | ❌ W0 | ⬜ pending |
| 1-01-02 | 01 | 1 | DATA-01 | — | N/A | unit | `cd frontend && npx vitest run --reporter=verbose` | ❌ W0 | ⬜ pending |
| 1-02-01 | 02 | 1 | DATA-02 | — | N/A | unit | `cd backend && npx vitest run --reporter=verbose` | ❌ W0 | ⬜ pending |
| 1-02-02 | 02 | 1 | DATA-02 | — | N/A | unit | `cd frontend && npx vitest run --reporter=verbose` | ❌ W0 | ⬜ pending |
| 1-03-01 | 03 | 1 | DATA-07 | T-1-01 / N/A | enum enforcement prevents invalid categories | unit | `cd backend && npx vitest run --reporter=verbose` | ❌ W0 | ⬜ pending |
| 1-03-02 | 03 | 1 | DATA-07 | — | N/A | unit | `cd frontend && npx vitest run --reporter=verbose` | ❌ W0 | ⬜ pending |
| 1-04-01 | 04 | 2 | DATA-01 | — | N/A | integration | manual — requires running backend + DB | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `backend/vitest.config.ts` — Vitest config for NestJS (tsconfig paths, module resolution)
- [ ] `backend/src/modules/assets/assets.service.spec.ts` — Unit tests for AssetsService (DATA-01, DATA-02, DATA-07)
- [ ] `backend/src/modules/assets/assets.controller.spec.ts` — Unit tests for AssetsController
- [ ] `frontend/vitest.config.ts` — Vitest config for React (jsdom environment)
- [ ] `frontend/src/components/assets/AssetForm.test.tsx` — Form rendering and validation tests
- [ ] `frontend/src/components/assets/AssetList.test.tsx` — List rendering and filtering tests
- [ ] `frontend/src/hooks/useAssets.test.ts` — TanStack Query hook tests

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Full CRUD flow end-to-end | DATA-01, DATA-02 | Requires running backend + DB + frontend | Start docker compose, add/edit/delete assets via UI, verify persistence |
| Docker compose up starts all services | Phase goal | Requires Docker runtime | Run `docker compose up`, verify backend + frontend + DB are healthy |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 10s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
