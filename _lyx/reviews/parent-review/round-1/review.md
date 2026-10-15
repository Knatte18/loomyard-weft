---
verdict: APPROVED
---

## Scope against the board entry

Every point of the board entry's Scope has a decision:

- Replace `burlerengine.ModelChoice` and the bare model strings with `shuttleengine.ModelChoice`: D3 covers burler, `shedadapters.BouncerModels`, treadle's `Profile`, `seatengine.Seat` and `alignment.Settings`.
- Convert every hand mapping from `Resolved` to `Spec`: D1 embeds `ModelChoice` in `Spec` and `OpenSpec`, and D2 routes every mapping through `NewModelChoice`, guarded by a scan.
- Per-round lists keep the engine, and `round-N-usage.yaml` records it: D3 and D4.
- Claude-only roles declare `Spec.Requires`: D5's table names Webster Master with its implementer forks, twill (Discussion-Write and Plan-Write with lenses), seats with advisors, the Burler cluster fan and the Claude-strand loom driver.
- Merriam's base reads the engine's instruction files: D7.

The entry's Out items stay out: the orch model, key renames (`loom-model-key-names`, which Out says waits on this task, as does `shed-arbiter-all`), and adding `codex` to the engine names.

Two decisions go past the entry's text, and both are licensed:

- D6, Webster auditing and parsing through each run's recorded engine, follows from the goal: once Master's spec can carry a non-default engine, reading its run through the default engine would be wrong.
  The support log records it as a known extension.
- D8, the `Requires` refusal-spec rows, is assigned to this task by `engine-registry`'s landing commit `88ea645bb` ("The tasks that make them reachable (`engine-model-choice`, `codex-engine`) land the rows"), and `PATTERN-refusal-way-forward` requires them once D5 makes them reachable.

No decision takes work from another entry: `creel-agents`' `SendMessage` move is kept out (D5 marks no role `peer-messaging` for its parent asks), and `engine-usage-readers` covers `tokencount`, not `round-N-usage.yaml`.

## Overlaps

`lyx loom peers` lists no parent commits. `own_row` is 3.

- `engine-config-seams` (row 3, a tie): both change `internal/shuttleengine` and the `PATTERN-shuttle-provider-seam` area, at different places.
  That run changes `shuttleengine.Config` and the reed provider tag. This one changes `Spec`, `OpenSpec` and `Result`.
  Its record puts "role specs carrying their engine, and `Spec.Requires` declarations by roles" out of scope, naming this task, so neither design contradicts the other.
  It edits the provider-seam background, and this run adds the new entry `PATTERN-model-choice-mapping`, so they do not change the same PATTERN entry.
  Its D9 provider tag is the run's resolved engine name, which fits this run's per-spec engine.
  The overlap is text-only: a tie, with no landing order given.
- `creel-core` (row 12, lands first): it changes `internal/shuttleengine/run.go`, `send.go` and `internal/shuttlecli/send.go`, and builds shuttle runners in `creelcli`.
  This run's D1 changes every `Spec` literal, so whichever lands second adapts; this run's Open risks already name it.
  No behaviour or PATTERN entry is changed two ways.
- `lighter-test-rules` (row 7, lands first): it prunes stencil-prose test assertions. This run adds no stencil prose, so nothing conflicts.

Shared bookkeeping, not an overlap: `cmd/lyx/testdata/test-budget.yaml` and `PATTERN.md` index lines, which `creel-core` also changes.
