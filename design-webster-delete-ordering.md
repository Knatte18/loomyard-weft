# Webster schedules a Delete before its callers are removed

`webster-delete-ordering` · Notes · bug

Seen 2026-10-03 in the `pair-lifecycle` run: Webster scheduled card 12 (`delete-half-removed-helpers`, which deletes `fabricengine.Topology.RemovePairBranch`, `PairBranchResult` and `PairSiblingRemnant`) before card 11, the card that removes their only production caller in `internal/battencli/wire.go`.
The batch then failed, with `recover-batch` refusing because the delete targets still resolve, and the run blocked at Webster.

`websterengine.deriveEdges` (`internal/websterengine/sequence.go`) orders card B after card A only when B's Uses names one of A's targets, or when A and B share a target and A has the lower number.
A card that deletes a symbol has no edge to the cards that edit its callers, because the callers appear in neither field.
Plan review did not catch the missing ordering either.

Possible fixes:
- Derive an edge from a Delete target to every card that edits a member referencing it, resolved through quarry.
- Have plan validation refuse a Delete whose symbol is still referenced from a member that a lower-numbered card edits, unless an edge already orders them.


From GitHub #350 (intake 2026-10-04).
