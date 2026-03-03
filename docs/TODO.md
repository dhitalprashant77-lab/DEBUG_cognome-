# Documentation — TODO

Last updated: 2026-03-03

---

## Needs Writing

- [ ] **Contributor setup guide** — SDK install, project build, Postgres setup, LMDB compilation steps. Critical for onboarding.
- [ ] **Architecture overview** — Current two-phase pipeline (char PBD → vocab PBD), LMDB data flow, socket API. Replace stale architecture.md references.
- [ ] **Scene pipeline design doc** — Triple scene pipeline for GPU/CPU overlap. Patrick's design exists in memory files, needs proper doc.
- [ ] **Label phase design doc** — Label tier 0 broadphase, label propagation rules. Design exists in memory, needs doc.

## Needs Updating

- [ ] **Stale research docs** — Several docs in `docs/research/` reference outdated approaches (force patterns, sub-categorization). Review and either update or mark as historical.
- [ ] **DB specialist consultation** — `docs/db-specialist-consultation.md` has 17 questions. Some answered in `docs/engine-to-db-feedback-response.md`. Consolidate or close resolved questions.

## Cleanup

- [ ] **Purge "7 force types" references** — research docs and specs still contain stale force-type references. Files: `english-force-patterns.md`, `force-pattern-db-requirements.md`, `force-pattern-db-review.md`, `concept-mesh-decomposition.md`, `particle-vs-md-operations.md`.

---

## Completed (recent)

- [x] status.md updated to Phase 2 state (27151b6)
- [x] roadmap.md open questions resolved (27151b6)
- [x] 6 superseded research docs deleted: OpenMM x3, Taichi, Warp, LMDB assembly (27151b6)
- [x] LMDB design specs added: brief-lmdb-vocab-compilation.md, engine-response-lmdb-vocab.md (27151b6)
- [x] DB consultation docs added: db-specialist-consultation.md, feedback request/response (27151b6)
- [x] Design concepts added: byte-resolution-design.md, concept-forces.md, discord-relay.md (27151b6)
