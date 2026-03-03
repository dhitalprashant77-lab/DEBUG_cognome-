# Database — TODO

Last updated: 2026-03-03

---

## High Priority

- [ ] **Fix marker table PK collision** — [GitHub #8](https://github.com/Human-Cognome-Project/human-cognome-project/issues/8). Control tokens share (t_p3, t_p4), needs t_p5 column added to `doc_marker_positions`.
- [ ] **Activity envelope implementation** — [GitHub #10](https://github.com/Human-Cognome-Project/human-cognome-project/issues/10). Schema exists (migration 021). Engine-side activation/eviction cycle in HCPEnvelopeManager still needed. Bootstrap envelope policies TBD.
- [ ] **Entity data cleanup** — Entity data from librarian DB is messy. Many entity words missing from hcp_english vocab. Patrick and DB specialist building clean DB alongside.

## Medium Priority

- [ ] **Update boilerplate loading** — [GitHub #9](https://github.com/Human-Cognome-Project/human-cognome-project/issues/9). Remove forward walk, boilerplate pushed to LMDB on cache miss.
- [ ] **Punctuation tokens in hcp_english** — AB.AA.AA.AF.* tokens created in error, needs cleanup to hcp_core.
- [ ] **Confirm forward table populated** — verify data before removing forward sub-db from engine.

## Backlog

- [ ] Force profile junction table (deferred until PBM experimentation)
- [ ] Python code audit in src/hcp/ for decomposed queries
- [ ] Fresh English dump after any further token additions

---

## Completed (recent)

- [x] Migrations 017-022: pronoun I, particle_key, freq_rank, position modifiers, activity envelopes, var_sources rename (237eec8)
- [x] DB dump refresh with migrations 016-022 applied (6408604)
- [x] LFS switch: .sql → .gz distribution format (6408604)
- [x] load.sh: all 6 databases, gzip + checksum support (6408604)
- [x] Stale duplicates removed: fic_pbm.sql, nf_entities.sql (6408604)
- [x] Frequency rank merge: Wikipedia + OpenSubtitles → Postgres (7f33603)
- [x] LMDB compilation: vocab beds, entity sub-dbs (7f33603)
