---
round: 1
focus:
  - "Card 1 Create list: does every Create glyph (test-only hooks `SetScratchBeforeLinkHookForTest`, `SetScratchPathTrackedForTest`, `scratchShapeCase`, `scratchOutcome.detail`) earn its own card entry under the granularity rule, or is it a private supporting symbol that belongs in another symbol's entry?"
  - "Test economy across cards 1-4: does any new test function or file cover behavior an existing test already covers, or exist per symbol rather than per public surface? Name the existing test."
  - "Card order: does card 2, 3 or 4 use a symbol that only a later card creates, and do cards 2-4 only use what card 1 creates?"
  - "Attack-surface: card 1's test hooks and the `.scratch` migration (existing contents, tracked paths, exclude seeding) and card 3's unlink through the gate: does each Intent state what the change can skip or let through and what bounds it?"
  - "Reachability: every refusal or guard branch a card prescribes (tracked `.scratch`, non-link occupant, stale-sweep skip, remove-unlink refusal) has a named input and a named test that reaches it, and no upstream check rejects that input."
  - "Edit cards: member glyphs and the re-sign arrow where Intent changes parameters or results (`wireJunctionSet` signature, Unwire, Remove, `applyStaleRemoval`, reconcile pair Detail), and `ImpactSummary` carrying a real conclusion rather than restating Intent."
  - "Fidelity: every Decision and Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and no card adds scope the record does not license; no card edits this task's own board entry."
  - "Verify coverage: every package any card targets (including `internal/cleanupcli`, `cmd/lyx`, and any package a doc or `fabric-windows` Windows-check change touches) has its tests run by the `## verify:` command."
---

Seed focus only; no round has been judged.
