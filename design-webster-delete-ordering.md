# Plan-Gate refuses a Delete card not ordered after the cards that remove its callers

`webster-delete-ordering` · Note · bug, webster

Seen 2026-10-03 in the `pair-lifecycle` run: Webster scheduled card 12 (`delete-half-removed-helpers`, which deletes `fabricengine.Topology.RemovePairBranch`, `PairBranchResult` and `PairSiblingRemnant`) before card 11, the card that removes their only production caller in `internal/battencli/wire.go`.
The batch then failed, with `recover-batch` refusing because the delete targets still resolve, and the run blocked at Webster.

`websterengine.deriveEdges` (`internal/websterengine/sequence.go`) orders card B after card A only when B's Uses names one of A's targets, or when A and B share a target and A has the lower number.
A card that deletes a symbol has no edge to the cards that edit its callers, because the callers appear in neither field.
Plan review did not catch the missing ordering either.

Possible fixes:
- Derive an edge from a Delete target to every card that edits a member referencing it, resolved through quarry.
- Have plan validation refuse a Delete whose symbol is still referenced from a member that a lower-numbered card edits, unless an edge already orders them.


From GitHub #350 (intake 2026-10-04).

Seen again 2026-10-06 in `skills-in-one-turn`: card 4 deleted `SkillLoader.SkillLoadSequence`, `SkillUnknown` and `Runner.LoadSkill` before card 3 moved `internal/orchengine` and `internal/orchcli` off them.
The recovery batch failed twice, and because batch 4 had begun, `webster rebaseline` refused the one-line `Uses:` fix; the run needed `webster reset --to start`, an archived `state.json` and a goto, redoing two finished batches.
A Plan-Gate check would have cost nothing at plan time.

## Decision (operator, 2026-10-06)

Webster follows the plan as written and never derives order on its own; the first fix option (an edge derived through quarry inside Webster) is ruled out.
The plan carries the order: a card that deletes a symbol names, in `Uses:`, the cards' targets that move its callers off it.
Plan-Gate refuses a plan whose Delete target is still referenced from a member another card edits when no `Uses:` edge orders the two, so an invalid plan never reaches Webster.
The plan spec and the plan template say so to the plan writer.

## Root cause in skills-in-one-turn (2026-10-06)

Card 2's `Uses:` named `internal/shuttleengine#Runner.skillLoader`, which card 4 edits, giving an edge 4 → 2; cards 2 and 4 also shared test-file targets, giving 2 → 4.
`SequenceBatches` condensed the cycle {2, 4}, ran it as 02 then 04, and placed card 3 (which uses card 2's new symbols) after the whole block, so 04 ran before 03.
The cycle was reported only as a `Cycle` warning nobody surfaced.
Plan-Gate refuses a plan whose derived edges form a cycle, alongside the unordered-Delete check above.
