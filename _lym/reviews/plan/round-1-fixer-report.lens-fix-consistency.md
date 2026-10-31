```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: checking that the fixer's change set agrees with the sibling cards, decisions, criteria and cross-references it touches.

1. **MEDIUM.** Card 11's new paragraph adds the `contract: loom` refusal to `New`, but nothing tests it and nothing labels it as a guard.
   - It reads: "With both recipes declaring the keys, `New` refuses a recipe that does not declare `contract: loom`, naming `NewPawl` or `lym shed`…"
   - The fix removed card 8's sentence that called this refusal a fail-loud wiring guard. Card 11 now carries the refusal with neither a reaching test row nor that label.
   - Between cards 11 and 19, a batten seed armed through `lym loom` reaches it: `newShed` branches off only pawl, and card 19's `armAt` refusal does not exist yet.
   - `lens-review.md:14` settled coverage-9 and guard-reach-2 on that label ("card 8 labels `New`'s non-loom refusal a fail-loud wiring guard"). The settlement now points at text that is gone, and the refusal has no row.

2. **MEDIUM.** Card 16's new "Not committed at HEAD" text disagrees with D4 in the decision record.
   - The card's text: "A crash inside Record-Write leaves this … and so does a darn run in flight … On any later row: `lym loom goto --to Record-Write`…"
   - D4 says "Not committed: only a crash inside Record-Write leaves this, with the run's current row still Record-Write, so the way forward is `lym loom resume`" (`_lym/discussion/decision-record.md:104`).
   - D4's acceptance criterion says "`decision add` refuses a darn record not committed at HEAD naming `lym loom resume`" (`decision-record.md:290`).
   - The record has no Added-after-Discussion entry for the change. The review's F8 fix asked for the parent to record one, so this is an open twin until the parent records it.

3. **LOW.** Card 9's `pattern/PATTERN-cwd-resolution.md` edit leaves the file's other tier lines stale.
   - The edit: "`pattern/PATTERN-cwd-resolution.md`'s tiers end at `preflight.Check`".
   - The same Tiers list still says "Three tiers" and "An orchestrator needs tier 3" (`pattern/PATTERN-cwd-resolution.md:21-23`). With `CheckSeed` gone those lines no longer hold, and the card does not say how they change.

4. **LOW.** Card 11's ImpactSummary was not updated for the new paragraph.
   - It still reads "Adds one producer, one registry entry, one Env passthrough and one darn row; no existing producer changes, and darn becomes a declared loom recipe."
   - The card now also changes `New`'s refusal and its verify-attempts stamp condition (the new `internal/loomrecipe#New` target), and the summary does not mention either.

Tool calls:
- Grepped the plan cards other than the changed ones for references to the changed text (card 8, `contract: loom`, `perchHardCap`, `CommitDescription`, the PATTERN files, the card 14/15 references). My context held only the changed cards.
- Read the overview's PATTERN-changes section, `PATTERN.md`'s index lines and the D4 criteria in the decision record. My context held the D4 body but not its acceptance criteria or the overview list.
- Grepped cards 14, 15 and 18 for darn commit handling, to check card 17's `commitDescription` against its siblings. My context did not hold those cards.
