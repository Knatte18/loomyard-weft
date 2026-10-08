---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/11-caller-coverage.md:59
    summary: Card 11's caller coverage turns the Delete leg of card 1's shapes#Generic row and the arrow leg of card 10's same row red, because GenericMethod's receiver names Generic outside its span; card 1 claims no fixture member references another and no card plans the fix
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/10-resign-head-match.md:30
    summary: Cards 10 and 11 keep their checks out of ValidateDispatch without stating what dispatch lets through and what bounds it, which card 8 and decision D5 do state
  - id: F3
    severity: NIT
    class: scope
    location: _lyx/plan/06-retire-handle-unreferenced.md:18
    summary: Two comments in handle_test.go still cite both containment tiers after card 5, and neither card 5 nor card 6 names them
---

### [BLOCKING] Caller coverage makes the Generic rows of cards 1 and 10 fail when card 11 lands

**Location:** `_lyx/plan/01-glyphchain-plan-gate.md:21`, `:35`, `:46`; `_lyx/plan/10-resign-head-match.md:39`; `_lyx/plan/11-caller-coverage.md:59`.

**Issue:**
Card 1 states "No fixture member references another, so no later caller-coverage card changes a shape row's outcome."
That is false for the `shapes#Generic` row.
The fixture declares `type Generic[E any] struct{ value E }` and `func (g *Generic[E]) GenericMethod() E`.
I built the lyx binary and ran `lyx quarry resolve` on a copy of exactly that shape.
quarry resolves `shapes#Generic` to the type's own line only, and `shapes#Generic.GenericMethod` to a separate line.
The receiver token `Generic` in `GenericMethod` therefore sits in the declaring file outside the type's resolved span.
Decision D5's token rule counts an identifier token of the name, not preceded by `.`, in the member's own package, as a reference.
That is also what Go would say: deleting or re-signing `Generic` breaks the method.

Two legs the plan plans for that row therefore turn blocking once card 11 lands:

- Card 1's Delete leg ("one card deleting the member validates clean") for `shapes#Generic`, which card 1 builds in `glyphchain_test.go`, gets a blocking `caller-uncovered` for `shapes/shapes.go`.
- Card 10's arrow leg ("a head re-signing the member validates clean for every shape a Go declaration names") for `shapes#Generic` gets the same finding, because a re-signed package-level type has the same receiver reference.

Card 11 edits `glyphchain_test.go` but its Intent and row list never mention changing these two legs, and card 1's sentence tells the implementer that none can change.
D1 requires that no card commits a failing row, and every card leaves the suite green.
As written, card 11 fails its own gate on rows it neither owns nor mentions, and the implementer must guess between changing the fixture (impossible: a generic method must name its receiver type) and changing the expectation.
The other rows are unaffected: no other fixture member names another, and `main` does not call `run`, which the plan already took care of.

**Fix:**
Correct card 1's sentence and decide the Generic rows' expected outcome in the plan.
The consistent answer, which follows D5, is that the Delete leg and the arrow leg of `shapes#Generic` list `shapes#Generic.GenericMethod` on the same card as the cover (the member glyph whose span holds the receiver), or that card 11 states those two legs flip to the blocking finding and the rows assert it.
Say which in card 11's Intent or its row list, and make card 1 and card 10 name the Generic rows as the exception.

### [LOW] Cards 10 and 11 do not state what dispatch lets through

**Location:** `_lyx/plan/10-resign-head-match.md:30`; `_lyx/plan/11-caller-coverage.md:24`.

**Issue:**
Card 8 states, for its plan-gate-only check, what `ValidateDispatch` now lets through and what bounds it.
Card 10 says only "It reads the tree, so it stays out of `ValidateDispatch`" for `resign-interface-method`.
Card 11 says only that `caller-uncovered` "runs at the plan gates and never in `ValidateDispatch`".
Its "What this lets through, bounded" list covers the name-based under-reporting, the method case, `Rename` and file or package `Delete`, but not the dispatch skip itself.
D5 gives the dispatch skip its own sentence: a reference set that shrinks as earlier cards land cannot change a verdict mid-run, so dispatch never refuses on it.
The rubric's attack-surface question wants what a pass left out of dispatch can skip and what bounds it, in the card that introduces it.
For both checks a plan edit made after the plan gate reaches dispatch unchecked, and the bound is webster's per-card build and test gate.

**Fix:**
Add one sentence to each of cards 10 and 11 naming that dispatch does not run the check and what bounds a violation that appears after the plan gate.

### [NIT] Stale "containment tiers" comments left unassigned

**Location:** `_lyx/plan/06-retire-handle-unreferenced.md:18`; `internal/planglyph/handle_test.go:573`, `:603`.

**Issue:**
Card 5 lists the comments that stop naming containment (`resolvePass`, `ValidateDispatch`, `resolveTargets`, `cardOwnHandles`, `BeginBatch`).
Two more exist in `TestBindHandles_RenameNewSideHandleBinds`: its doc comment and an inline comment both say "collectGlyphTargets and both containment tiers".
Card 6 edits that test but only for the retired `handle-unreferenced` ID, and card 5 does not list the test.
After card 5 the comments cite two tiers that no longer exist.

**Fix:**
Add the two comments to card 6's `TestBindHandles_*` bullet, or to card 5's bullet list, so one card owns them.

## Checks that found nothing

These are the focus questions I settled with no finding, recorded so the next round need not redo them.

- **Cards 5 and 6 (retired guards):** both Intents state what the removal lets through and the bound, matching D2 and D4.
  The cited mechanisms exist (`CheckBatchOrder` in `internal/websterengine/sequence.go`, the `bind-count-mismatch` and `create-not-done` backstops).
- **Cards 8, 10, 11 (plan-gate-only):** `planGatePass` is called from `planglyph.ValidateFormat` and `Validate`, and `ValidateDispatch` calls only `planparser.ValidateFormat` plus `resolvePass`.
  `lyx loom validate-plan` reaches the pass through `internal/planglyph/index.go` and `ValidateRework`, so `PATTERN-gate-self-check-parity` holds.
  The `resign-head-mismatch` check sits inside `CanonicalizeHandles`, which `resolvePass` runs at every gate, as D5 requires.
- **Constraint coverage:**
  - Ledger gates: card 5 removes `gateSyntacticContainment`, card 7 and card 9 add their gates in `shape.go`.
    The planglyph-side checks classify through `planparser.IsHandleRef` and `glyph.Parse`, the route the existing planglyph code and `PATTERN-ref-shape-registry` already allow.
  - Chokepoint pin: no card adds a `quarry.Name` or `Resolve` call site.
    Card 10 builds its sources inline in `CanonicalizeHandles`, and cards 8 and 11 call `resolveTargets`.
  - Refusals: every new blocking finding names a way forward and has a spec row and a test in its own card.
- **Premises of cards 3 and 4:** I probed both.
  quarry answers `shapes#Iface.IfaceMethod` with signature `IfaceMethod() int`, and `shapes#PairB` with `var PairA, PairB = 4, 5`.
  `quarry.Name` rejects `IfaceMethodRenamed() int` as unparseable and `var PairA, PairRenamed = 4, 5` as declaring two symbols.
  It names `func (Iface) IfaceMethodRenamed() int` as `shapes#Iface.IfaceMethodRenamed`, and `var PairRenamed int` and `const PairRenamed = 0` as `shapes#PairRenamed`.
  Every glyph spelling in card 1's row list matches `lyx quarry glyphs` output on a copy of the fixture shapes, `shapes_test#TestExternal` and `cmd/tool#run` included.
  The struct-field row's `not_found` is quarry's current answer.
- **Card 10's `resign-interface-method`:** licensed by D5's acceptance line, because a head such as `func (i Iface) IfaceMethod() int` names the interface-method glyph, so `resign-head-mismatch` alone would not refuse it.
- **Test economy (cards 1-11):** the containment and `handle-unreferenced` rows in card 1 are mandated by D1 and D7 so that cards 5 and 6 can flip them.
  The new rows each pin a behavior no existing test covers, and the planparser unit cases and the planglyph integration rows differ on the before/after-binding verdict.
- **Card order:** cards 9-11 depend only on earlier cards (`CardResign` from card 9, `planGatePass` from card 8), and the D2, D4, D3, D5 flips land in the cards D7 assigns.
- **Card 11 scope:** the `deleteorder.go` region helper is what D5's "not reported twice" rule needs, and the fixture module files are licensed by D1 and D5.
  The extra exclusion of dot- and underscore-prefixed directories is a conservative superset of D5's testdata and vendor exclusion that the go tool shares.
- **Card 8 golden fixture:** the spec's worked example is stated to be byte-consistent with `testdata/goodplan`, so dropping the same bullet in both is consistent, if not strictly needed by the check.
- **`## verify:`:** `./...` covers every targeted package, `internal/websterengine` and `contracts/stencils` included, and the order is the one `loom-template-plan.md` states.
