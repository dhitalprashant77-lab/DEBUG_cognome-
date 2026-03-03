# HCP Engine — TODO

Last updated: 2026-03-03

## Legend

- **[BLOCKED]** — waiting on another task or decision
- **[READY]** — can be picked up now
- **[IN PROGRESS]** — someone is working on it

---

## Active: Phase 2 Optimized Resolution Pipeline

### Dead Code Cleanup — [GitHub #22](https://github.com/Human-Cognome-Project/human-cognome-project/issues/22)

- [ ] **[READY]** Strip dead HCPWriteKernel class from HCPStorage.h/cpp (~1,570 lines). Keep live entity cross-ref functions. Consider moving entity functions to own file.
- [ ] **[READY]** Remove dead TierAssembly + ChamberManager + ResolutionChamber from HCPResolutionChamber.h/cpp (~1,100 lines). Keep shared structs (ResolutionManifest, MorphBit, ResolutionResult, CharRun, StreamRunSlot). Consider renaming file.
- [ ] **[READY]** Remove HCPDetectionScene.h/cpp (421 lines entirely dead). Remove from hcpengine_files.cmake.
- [ ] **[READY]** Remove dead forward sub-db code from HCPVocabulary (~35 lines).

### Performance

- [ ] **[READY]** Triple scene pipeline pipelining — [GitHub #24](https://github.com/Human-Cognome-Project/human-cognome-project/issues/24). State machine refactor of BedManager::ResolvePass for GPU/CPU overlap. ~50% Phase 2 speedup target.
- [ ] **[READY]** Label propagation — if a word appears as a Label anywhere in text, restore firstCap on all suppressed instances of the same token_id. Not yet implemented.

### Bugs

- [ ] **[BLOCKED]** Entity cross-ref OR matching — [GitHub #23](https://github.com/Human-Cognome-Project/human-cognome-project/issues/23). GetFictionEntitiesForDocument matches ANY name token instead of ALL. Blocked on entity data cleanup.
- [ ] **[READY]** Service file binary mismatch — [GitHub #25](https://github.com/Human-Cognome-Project/human-cognome-project/issues/25). hcp-engine.service references wrong binary name + has restart-loop workaround.

### Infrastructure

- [ ] **[READY]** Persist bond tables as files — [GitHub #5](https://github.com/Human-Cognome-Project/human-cognome-project/issues/5). Bond compiler still hits Postgres on startup (~3s). Store compiled tables as binary files.

---

## Workstation

- [ ] **[READY]** Var positions in panel — display position data for each docvar occurrence in the Vars tab.
- [ ] **[READY]** Proper candidate detection in tokenizer — flag out-of-place capitalization during tokenization.
- [ ] **[READY]** Alias grouping review workflow — confirm/reject/promote groups in the Vars tab panel.

---

## Format Builders (contributor tasks)

- [ ] **[READY]** PDF text extractor — [GitHub #14](https://github.com/Human-Cognome-Project/human-cognome-project/issues/14)
- [ ] **[READY]** EPUB text extractor — [GitHub #15](https://github.com/Human-Cognome-Project/human-cognome-project/issues/15)
- [ ] **[READY]** HTML text extractor — [GitHub #16](https://github.com/Human-Cognome-Project/human-cognome-project/issues/16)
- [ ] **[READY]** Markdown text extractor — [GitHub #17](https://github.com/Human-Cognome-Project/human-cognome-project/issues/17)
- [ ] **[READY]** Wikipedia dump processor — [GitHub #18](https://github.com/Human-Cognome-Project/human-cognome-project/issues/18)

---

## Future (not yet planned in detail)

- Triple scene pipeline (design exists, implementation pending)
- Custom physics engine (~65 core forces, linguist-defined)
- Conversation levels (documents as entities in level workspaces)
- Language shard system (new languages via vocabulary + force constants)
- Texture engine (linguistic force bonding — surface language rules)

---

## Completed (recent)

- [x] WriteKernel decomposition — monolith split into 7 kernel modules (615bbfc)
- [x] Persistent vocab beds — 15 PBD systems replace 175 per-batch chambers (63750f5)
- [x] Morphological resolution — MorphBit field, RunTag routing, inflection stripping (650cea0)
- [x] Entity annotator — multi-word entity recognition from LMDB (8e9b3f6)
- [x] Manifest scanner — single-pass PBM from resolution manifest (fe573fe)
- [x] phys_ingest endpoint — full ingest pipeline via socket API (8e9b3f6)
- [x] Workstation overhaul — socket client, embedded DB kernels, systemd service (3b86915)
- [x] inline constexpr cleanup — ODR fix, DbUtils consolidation (4109085)
