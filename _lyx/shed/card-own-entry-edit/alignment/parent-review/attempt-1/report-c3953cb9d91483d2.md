---
verdict: APPROVED
---

## Scope against the board entry

The board entry (issue #653 and the operator decision of 2026-10-15) asks for four things.
The decision record covers each:

- No card may edit the running task's own board entry: D3, D4 and D5 state it in the Discussion, Plan and darn write stencils, and D2 adds `PATTERN-own-entry-untouched`, which every spawned agent sees.
- Find what led the planner to write that step: the support log traces it to creel-core's decision record, which cited the `CLAUDE.md` sentence "a design whose code has landed moves from the board entry into `doc.go`" and the matching `docs/overview.md` bullet; it reports the Plan stencils and `contracts/specs/` as clean.
- Make the rule plain where the planner reads it: D1 rewords `CLAUDE.md` and `docs/overview.md#documentation-lifecycle` to say the entry is left to the landing and the prune.
- The board lock stays: Scope Out leaves the lock and `lyx board upsert` unchanged, and an acceptance criterion pins it.

The optional plan check ("welcome if it is cheap") is declined in Scope Out with a reason, which the entry permits.
The darn writer (D5) and the three reviewing rubrics (D6) go past the entry's letter, which names cards only.
They are the same defect at the other places a writer or reviewer reads the rule, flagged as auto-picks in the record, and add no work that belongs to another entry, so they are not a scope deviation.
Board writes to other entries and board-module code stay allowed, as the entry implies.

## Other board entries

No entry duplicates or contradicts this design.
`stencil-intent-audit` audits stencil claims in general and holds no decision on this rule.
`creel-agents` is a note that edits loom stencils for messaging, a different subject; it may touch the Discussion stencil at a different place.
`discussion-lands-invariant` concerns Discussion landing a repository edit early, not board-entry edits.

## Overlap with runs in flight

`lyx loom peers` lists no parent commits.

- `engine-model-choice` (row 20, own row 3): both runs change `PATTERN.md`.
  That run adds `PATTERN-model-choice-mapping` and this run adds `PATTERN-own-entry-untouched`, two separate new index lines; that is shared bookkeeping, not a design conflict.
  No other changed path of that run (`contracts/specs/refusal-spec.md` and Go packages) is one this design edits.
  Landing order: `engine-model-choice` is further along and lands first; this run takes its changes at Merge-In.
- `engine-usage-readers` (row 9): its plan touches `tools/tokencount`, `shuttleengine`, `claudeengine` and `shuttleengines`, none of which this design edits; no overlap.
