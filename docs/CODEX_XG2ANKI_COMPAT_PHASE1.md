# Codex task: XG2Anki Filter Builder compatibility Phase 1

Read `docs/XG2ANKI_FILTER_BUILDER_COMPAT.md` first. Treat it as the canonical specification.

## Objective

Implement the smallest compatibility layer that lets AnkiGammon-generated cards work with the high-value XG2Anki Filter Builder Move Filters and checker-play Action Done filters.

Do not attempt full XG2Anki compatibility.

## Required implementation

Create a pure compatibility module, preferably:

```text
ankigammon/anki/xg2anki_compat.py
```

It should derive searchable compatibility metadata from `Decision` without changing analysis results or rendering.

Phase 1 fields:

```text
Multiple Choice List
Multiple Choice Answer
Best Action
Action Done
```

Phase 1 tag support:

```text
X2A::ErrorSize::<bucket>
```

## Required integrations

Update both export paths:

1. AnkiConnect direct sync
2. APKG export

For AnkiConnect:

- add missing compatibility fields to existing current/legacy note types
- append fields, never reorder existing fields
- populate fields on add and update/upsert
- preserve review history and XGID identity

For APKG:

- append compatibility fields after `AnalysisData`
- keep `XGID` at field index 0
- preserve StableNote GUID semantics

## Serialization requirements

`Multiple Choice List` must be deterministic and plain text:

```text
(a) MOVE1;(b) MOVE2;(c) MOVE3;
```

Preserve XG notation and `*` hit markers.

`Best Action`:
- checker-play notation of the rank-1 move

`Action Done`:
- checker-play notation of the move with `was_played == True`
- empty if unknown
- never infer or fabricate

`Multiple Choice Answer`:
- lower-case option letter for the best action in the compatibility list
- independent of randomized visual MCQ order

## Error-size tags

Implement the bucket map documented in the canonical specification.

Use the authoritative checker error where available:
1. `Decision.xg_error_move`
2. played move `xg_error`
3. played move `error`

Do not tag missing error as `NoError`.

## Tests first

Add focused unit tests before changing production behavior.

At minimum test:

- Multiple Choice List exact format
- preservation of `*`
- deterministic option labels
- Best Action
- Action Done
- missing Action Done
- ErrorSize boundary values
- model migration appends fields
- APKG XGID remains field 0
- direct sync add/update includes all new fields
- re-upsert same XGID does not create duplicates

Run the full existing test suite after focused tests pass.

## Manual acceptance

Build a small test deck and verify queries copied from https://fb.xg2anki.de/ for:

- Hit
- Double Hit
- Stack or Stack & Up
- one anchor-breaking filter
- Missed Hit
- Wrong Hit

Do not replace these with hand-written "equivalent" queries. The purpose is compatibility with the real Filter Builder output.

## Out of scope

Do not implement in this task:

- cube Action Done compatibility
- DN/TP ActionType tags
- XG Results - Extra Info
- Score Matrix
- PositionType classification
- Dirk tags
- BestMoveDistinctCount
- Volatility
- full XG2Anki field parity

If implementation reveals that a requirement in the canonical specification is incorrect or impossible, stop and document the conflict instead of silently changing the design.
