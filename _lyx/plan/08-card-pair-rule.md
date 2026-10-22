# Card 8 — one pair rule for card dependency edges

**Create:**
- `plan:internal/planparser#RefKind` -> `type RefKind int`
- `plan:internal/planparser#RefUse` -> `const RefUse RefKind = 1`
- `plan:internal/planparser#RefChange` -> `const RefChange RefKind = 2`
- `plan:internal/planparser#RefExtends` -> `const RefExtends RefKind = 3`
- `plan:internal/planparser#CardRefKind` -> `func CardRefKind(card Card, ref string) RefKind`
- `plan:internal/planparser#DependencyEdge` -> `func DependencyEdge(earlier Card, earlierKind RefKind, later Card, laterKind RefKind) bool`
- `plan:internal/planparser#TestDependencyEdge` -> `func TestDependencyEdge(t *testing.T)`

**Edit:**
- `internal/websterengine#deriveEdges`
- `internal/websterengine#backwardEdgeFinding`
- `internal/websterengine#firstSharedRef` -> `func firstSharedRef(producer, consumer planparser.Card, consumerUses bool) string`
- `internal/websterengine_test#TestCheckBatchOrder`
- `internal/websterengine/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/planparser#CardPatternEntries`
- `plan:internal/planparser#CardTypeExtends`
- `internal/websterengine_test#oneCardBatch`

**Intent:**
D4: one function in `internal/planparser` decides whether a pair of overlapping refs of two cards makes a dependency edge; overlap detection stays each caller's.

`CardRefKind(card, ref)` says what a card does with a ref, compared by exact string after trimming: `RefChange` when a group of any type label but Extends holds it, or when `card.Targets` holds it and no group does (a hand-built card with no groups, as tests build); `RefExtends` when only Extends groups hold it; `RefUse` when only `Uses` holds it; 0 when the card does not name it.
`DependencyEdge(earlier, earlierKind, later, laterKind)` answers, for a pair its caller already found overlapping:
false when either kind is 0 or both are `RefUse`;
for two `RefExtends`, true exactly when the two cards fence a verbatim `PATTERN.md` line for one entry ID (`CardPatternEntries` over both), else false;
true otherwise: Extends against any other label, a `Uses` against an Extends target, and every pair today's rule makes an edge.
The bound D4 states: a wrongly labelled Extends drops a real dependency, which with batches run in card order changes nothing, and with parallel batches surfaces as a merge conflict or a verify failure on the merged tree.

`deriveEdges` keeps its two clauses and refines them per shared ref: a `Uses` entry of b matching a target of a adds the edge when `DependencyEdge(a, CardRefKind(a, ref), b, RefUse)` holds, and a target shared by a and b with `a.Number < b.Number` adds it when `DependencyEdge(a, CardRefKind(a, ref), b, CardRefKind(b, ref))` holds.
`firstSharedRef` takes the two cards and returns the first ref that makes an edge under the same rule, so `backwardEdgeFinding` names a ref that is a real dependency; its two call sites in `backwardEdgeFinding` pass the producer, the consumer and whether the consumer's side is its `Uses`.
`websterengine/doc.go` and `loom-plan-spec.md`'s dependency-graph text state the rule: two Extends on one ref make no edge unless both fence a line for one PATTERN entry ID.

Tests:
`TestDependencyEdge`, table-driven in package `planparser`, builds cards with `TargetGroups`, `Targets`, `Uses` and `Text`: Extends against Extends (false); the same with both cards fencing ``- `PATTERN-x` …`` (true) and fencing different IDs (false); Extends against Edit (true); Uses against Extends (true); Uses against Uses (false); a groupless hand-built target against another (true); and `CardRefKind` for each label, for Uses and for an unnamed ref.
`TestCheckBatchOrder` gains rows built with `TargetGroups` and `Text` beside `oneCardBatch`: two cards in reverse batch order sharing a ref only under Extends are accepted; Extends against Edit on that ref in reverse order is refused; a Uses against a later Extends target is refused; two Extends on `PATTERN.md` fencing a line for one ID in reverse order are refused.

**ImpactSummary:** `firstSharedRef` is called only by `backwardEdgeFinding`, both re-signed or edited here; `deriveEdges` feeds only `CheckBatchOrder`; cards without Extends groups keep today's edges exactly.
