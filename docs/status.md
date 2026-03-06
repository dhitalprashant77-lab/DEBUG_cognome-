# Project Status

_Last updated: 2026-03-06_

## What Exists

### Governance (stable)
- **Covenant** -- Founder's Covenant of Perpetual Openness. Ratified.
- **Charter** -- Contributor's Charter. Ratified.
- **License** -- AGPL-3.0, governed by the Covenant.

### Specifications (first draft)
- **Data conventions** -- core definitions, PBM storage algorithm, atomization, NSM decomposition
- **Token addressing** -- base-50 scheme, 5-pair dotted notation, reserved namespaces (v-z for entities/names/PBMs)
- **Pair-bond maps** -- FPB/FBR structure, reconstruction, compression, error correction
- **Architecture** -- two-engine model, conceptual forces, LoD stacking
- **Identity structures** -- personality DB (seed + living layer), relationship DB, integration with core data model

These are first-pass specs derived from working notes. They need review, critique, and refinement.

### Engine: O3DE + PhysX 5 PBD Superposition Pipeline

The primary inference engine is an O3DE 25.10.2 C++ Gem using PhysX 5 GPU-accelerated Position Based Dynamics for tokenization and resolution. ~21,300 lines of C++ across 35 source modules.

**Headless daemon** (`HCPEngine.HeadlessServerLauncher`) listens on port 9720 with JSON socket API for health, ingest, retrieve, list, tokenize, and physics-based ingestion.

**Pipeline:**
```
DecodeUtf8ToCodepoints
  -> Phase 1: byte->char PBD superposition (16K codepoints/chunk, Unicode-aware)
  -> ExtractRunsFromCollapses (with capitalization suppression)
  -> Phase 2: char->word via persistent VocabBeds (BedManager, 5 GPU workspaces: 3 primary + 2 extended, triple-pipelined via RunPipelinedCascade)
    -> TryInflectionStrip (PBD is the existence check, silent-e fallback)
    -> TryVariantNormalize (V-1 g-drop: -in'→-ing; V-3 archaic: -eth→base; runs after inflection strip)
    -> AnnotateManifest (multi-word entity recognition)
    -> ScanManifestToPBM
    -> StorePBM + StorePositions
```

**Benchmarks (2026-03-04, GTX 1070 headless, pipelined):**

| Text | Size | Tokens | Vars | Wall Time | Previous (pre-pipeline) |
|------|------|--------|------|-----------|--------------------------|
| Dracula | 890 KB | 199,368 | 110 | **28.6s** | 166.5s (2026-03-02) |
| A Study in Scarlet | 269 KB | 56,061 | 54 | **12.2s** | 133.0s (2026-03-02) |
| The Yellow Wallpaper | 47 KB | 10,856 | 37 | — | — |
| The Sign of Four | — | — | 52 | — | — |

V-1 variant normalization (2026-03-04): 12/12 dialect g-drops resolved in Sign of Four. Resolution rates >97% on test corpus.

**Key engine modules** (`hcp-engine/Gem/Source/`):

| Module | Purpose |
|--------|---------|
| `HCPVocabBed.h/cpp` | VocabBed (persistent PBD per word length) + BedManager (orchestrator) |
| `HCPVocabulary.h/cpp` | LMDB reader + affix loader. 48 max DBIs. |
| `HCPSuperpositionTrial.h/cpp` | Phase 1 byte->char PBD |
| `HCPWordSuperpositionTrial.h/cpp` | CharRun extraction with cap suppression |
| `HCPEntityAnnotator.h/cpp` | Multi-word entity recognition from LMDB |
| `HCPEngineSystemComponent.h/cpp` | Top-level orchestrator. Owns BedManager + all DB kernels. |
| `HCPSocketServer.h/cpp` | TCP socket API on port 9720 |
| `HCPParticlePipeline.h/cpp` | PBD particle system: Phase 1 + Phase 2 scenes |
| `HCPEnvelopeManager.h/cpp` | Activity envelope cache lifecycle (LMDB hot cache) |
| `HCPCacheMissResolver.h/cpp` | LMDB cache miss -> Postgres fill |
| `HCPBondCompiler.h/cpp` | Sub-word PBM bond tables (char->word, byte->char) |
| `HCPTokenizer.h/cpp` | 7-step resolution cascade |

**Decomposed DB kernel modules** (split from monolith 2026-03-01):

| Module | Purpose |
|--------|---------|
| `HCPDbConnection.h/cpp` | Shared PGconn* wrapper |
| `HCPPbmWriter.h/cpp` | StorePBM + StorePositions |
| `HCPPbmReader.h/cpp` | LoadPBM + LoadPositions |
| `HCPDocumentQuery.h/cpp` | Document listing, detail, provenance, metadata |
| `HCPDocVarQuery.h/cpp` | Document-local var queries |
| `HCPBondQuery.h/cpp` | Bond queries for tokens/starters |
| `HCPDbUtils.h` | Base-50 encode/decode, token helpers |

### Source Workstation

Standalone Qt binary (`HCPWorkstation`, 14 MB) with dual-mode architecture:
- **Offline mode**: Embedded DB kernels + LMDB vocab. Browse docs, edit metadata, view bonds/text without daemon.
- **Connected mode**: All offline features + physics operations via socket to daemon on port 9720.
- **DB abstraction**: `IDatabaseBackend` with Postgres and SQLite implementations.
- Files: `Source/Workstation/` (7 source files), `hcpengine_workstation_files.cmake`

### Pre-Compiled LMDB Vocab

Offline-compiled LMDB vocabulary for zero-SQL runtime:
- 809,303 entries (37% reduction from 1.28M via morphological stripping), ~34 MB on disk
- Per-word-length sub-databases (`vbed_02`..`vbed_16`), frequency-ordered
- Labels (tier 0) -> freq-ranked (tier 1) -> unranked remainder (tier 2)
- Entity sub-databases: 723 sequences total (fic + nf), verified via dry-run. DB cleaned 2026-03-05; LMDB recompile pending (compile_entity_lmdb.py needs review for variant/morph categories before next full compile).
- Frequency data: Wikipedia 2023 + OpenSubtitles merged. 176K tokens ranked.
- Scripts: `compile_vocab_lmdb.py`, `compile_entity_lmdb.py`, `merge_frequency_ranks.py`

### Database Shards (6 databases, 24 migrations applied)

**Core shard (`hcp_core`)** -- AA namespace
- ~5,200 tokens: byte codes, Unicode characters, structural markers, NSM primitives
- Namespace allocations, shard registry (10 entries)
- Activity envelope schema (definitions, queries, composition, audit log)
- Punctuation and single-character tokens live here, not in hcp_english

**English shard (`hcp_english`)** -- AB namespace
- ~1.4M tokens: words, affixes, derivatives, multi-word entries
- Kaikki dictionary data: 1.3M entries, 1.5M senses, 870K forms, 450K relations
- Lowercase-normalized (migration 016), particle_key indexed (migration 018)
- Frequency ranks from OpenSubtitles + Wikipedia (migration 019)
- Variant form support: `tokens.category` (archaic/dialect/casual/literary) + `tokens.canonical_id` FK (migrations 023-024). 4,815 clean variant entries after Wiktionary data cleanup.
- All tokens atomized to character Token IDs

**Fiction PBM shard (`hcp_fic_pbm`)** -- v* PBMs
- PBM prefix tree storage (migration 011): documents -> starters -> bond subtables
- Positional token lists on starters (migration 013)
- Positional modifiers (migration 020): morph bits + cap flags
- Document-local vars (migration 012), var staging pipeline (migration 015)

**Fiction entities (`hcp_fic_entities`)** -- u*/t*/s* namespaces
- Fiction people, places, things from cataloged texts
- 584 tokens; entity names cleaned 2026-03-05 (lowercase_underscore, 1-indexed positions, phantom refs fixed)

**Non-fiction entities (`hcp_nf_entities`)** -- y*/x*/w* namespaces
- Source records, editions, entity lists (migration 014)
- Work entities (wA.DA.*), author entities
- 116 tokens; 27 missing place name rows inserted 2026-03-05

**Var shard (`hcp_var`)** -- short-term memory for unresolved sequences
- Var token dedup by surface form
- var_sources with doc_token_id for cross-shard references (migration 022)

DB dumps: gzipped with SHA-256 checksums. `load.sh` handles all 6 databases with auto-creation and checksum verification.

### Python Tooling (`scripts/`)
- `compile_vocab_lmdb.py` -- LMDB compiler with morphological stripping
- `compile_entity_lmdb.py` -- Entity LMDB compiler (fic/nf sub-dbs)
- `merge_frequency_ranks.py` -- Frequency data merge (Wikipedia + OpenSubtitles -> Postgres)
- `ingest_texts.py` -- Text ingestion via socket API
- `run_benchmark.py` -- Benchmark runner

### Decision Records (`docs/decisions/`)
- 001: Token ID decomposition
- 002: Names shard elimination (hcp_names merged into hcp_english)
- 005: Decompose all token references

### Research Documents (`docs/research/`)
- PBM storage schema, entity DB design, source workstation design
- Sub-categorization patterns, concept mesh decomposition, force patterns
- PhysX evaluation docs in `hcp-engine/docs/`

## What's In Progress

- **Envelope-based variant loading** -- Wire variant DB entries (env_archaic / env_dialect / env_casual) into resolve loop with VARIANT morph bits (bits 12-15). Variant forms stored in DB (4,815 entries); loading path not yet wired.
- **Entity LMDB recompile** -- Entity DB cleaned (2026-03-05); `compile_entity_lmdb.py` needs review for variant/morph category support before next compile.
- **Label propagation** -- If word appears as Label anywhere, restore firstCap on all suppressed instances.

## Recently Completed

- **V-1/V-3 variant normalization** (2026-03-04, commit `d59c2fa`) -- `TryVariantNormalize` in `HCPVocabBed.cpp`. V-1 g-drop (`-in'`→`-ing`, dialect speech) and V-3 archaic (`-eth` 3rd person). Wired after inflection strip in Stages 2 and 4. Validated: 12/12 dialect g-drops resolved in Sign of Four.
- **Entity data cleanup** (2026-03-05) -- `hcp_fic_entities` and `hcp_nf_entities` cleaned in-place. tokens.name normalized (lowercase_underscore, parentheticals stripped). entity_names position convention fixed to 1-indexed throughout. 27 missing nf place name rows inserted. 6 phantom token_id refs corrected. 34 Label tokens lowercased in hcp_english. 723 sequences verified via dry-run.

## What Doesn't Exist Yet

- Known initialisms handling (U.S., U.K., etc.)
- Web address mini-language (language-independent, in hcp_core)
- Concept force modeling (~65 core NSM forces -- categorization, axes, definitions TBD)
- Full text inference (Phase 3 -- NAPIER)
- Multi-modality support (audio, visual)
- Identity structures implementation (personality DB, relationship DB)
- Community infrastructure (issue templates, CI, discussion forums)
- Cross-platform build (Windows planned)

## Data Sources

- **Kaikki** -- Wiktionary extracts for English vocabulary. Open data.
- **Wikipedia word frequency** (2023, MIT license) -- 2.7M entries
- **OpenSubtitles word frequency** (CC BY-SA 4.0) -- 1.6M entries
- **Project Gutenberg** -- Source texts for PBM construction and benchmarking
