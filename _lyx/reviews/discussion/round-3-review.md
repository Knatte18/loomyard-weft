---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:74
    summary: D5 puts a new quarry.Name call for the re-sign head inside resolvePass, but a pinned test allows quarry.Name only inside CanonicalizeHandles
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:20
    summary: Scope lists one Plan-Review rubric finding while D6 and the acceptance criteria carry two
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:131
    summary: No acceptance criterion covers a Rename to-side handle in the D3 redundancy check, the one handle whose unit is not its own spelling
---

### [MEDIUM] D5's `quarry.Name` call site breaks the pinned naming chokepoint

**Location:** `_lyx/discussion/decision-record.md:74` (D5), with `internal/planglyph/chokepoint_enforcement_test.go` and `internal/planglyph/handle.go`.

**Issue:**
D5 says the head-to-member match "calls `quarry.Name` in `planglyph`, inside `resolvePass` beside `CanonicalizeHandles`".
`TestChokepointCallSites_PinnedToTheirGuardedFunctions` fails any `quarry.Name` call whose enclosing function is not `CanonicalizeHandles` (`chokepointGuardedFuncs`).
`handle.go`'s file comment and `CanonicalizeHandles`' doc say the same: one batched `quarry.Name` call, which owns the length-plus-echo guard, and `resolvePass` is its only caller.
A card that follows D5 literally adds a call site in `resolvePass` and goes red on the repo's own enforcement test; the Constraints section does not mention the pin, so the plan writer has no warning.
The decision rests on a false premise: that `planglyph` has a free `quarry.Name` seam.

The route through the chokepoint also has a trap the record does not state.
`CanonicalizeHandles` treats every `quarry.Name` result as a handle to canonicalize (`canonicalOwners`, `subs`, `handle-canonical-collision`, the `RewriteRefs` rewrite).
An Edit head's result must be compared with its glyph and discarded, never fed into that map: the head names an existing glyph, so it would otherwise collide with or rewrite real text.

**Fix:**
Settle the seam in D5.
Either the Edit heads join `CanonicalizeHandles`' one batched `quarry.Name` call as a second declaration source whose results are compared to the glyph's `Glyph.String` and kept out of the canonicalization maps, or D5 names a second guarded function and extends `chokepointGuardedFuncs` and the file comments in the same card.
Add the pin to Constraints either way.

### [LOW] Scope omits the second rubric finding

**Location:** `_lyx/discussion/decision-record.md:20`

**Issue:**
Scope's In list names the Plan-Review rubric change as one finding, "an `Edit` card holding only file paths while its Intent names specific symbols".
D6 (lines 91–92) and the acceptance criteria (line 138) add a second, the missing re-sign arrow.
The Open risks entry leans on that second finding as the only backstop, so Scope understates what the rubric card delivers.

**Fix:**
Extend the Scope bullet to name both findings.

### [LOW] No acceptance criterion for a Rename to-side handle in the redundancy check

**Location:** `_lyx/discussion/decision-record.md:131`

**Issue:**
D3 gives a `Rename` to-side handle its own unit rule: it counts under its pair's Old glyph's unit, not the draft handle's spelling, which `CanonicalizeHandles` may correct.
The criterion on handles (line 131) covers only a `Create` handle; the D1 shape classes also hold no Rename-with-handle row.
The first gate runs the pure pairings before canonicalization, so a misspelled draft unit is exactly the case where the two rules give different answers, and nothing requires a test to reach it.

**Fix:**
Add a criterion: a card listing a package self glyph and a `Rename` whose to-side handle spells a different unit gets the redundancy finding or not according to the Old glyph's unit, at the plan gate and from `ValidateDispatch`.

## Verdicts on the round's focus entries

- **Open risks versus D6 (focus 1).**
  Settled, no defect.
  `contracts/stencils/loom/loom-rubric-plan-review.md` holds neither finding today.
  D6 builds both, the second being the `Edit` member without the arrow whose Intent changes parameters, results, type parameters or receiver.
  The Open risks wording ("the only backstop, and it catches only a signature change the Intent states") matches D6 and claims no mechanical backstop.
  The only residue is F2.
- **D3 and the handle transition (focus 2).**
  Settled, no defect beyond F3.
  The code agrees with the record: `CanonicalizeHandles` rewrites to `plan:<glyph>` and `planparser.IsHandleRef` still reports it as a handle; `BindHandles` at record-batch strips the prefix.
  D3 names that transition (line 55) and the acceptance criteria on lines 128 and 131 use it as the moment the verdict could change.
  The Rename to-side unit rule is stated (line 53) and matches `renameDeclSource`, which takes `Unit` from the Old symbol's glyph.
- **The stencil sentence (focus 3).**
  Settled, no defect.
  D4 defers the sentence to D6, D6 drops it, D7 orders D6 last, and Q19 records the decision.
  The sentence sits in `loom-template-plan.md` line 122 together with the `handle-unreferenced` ID, so one card, D6's, removes both.
- **The re-sign arrow (focus 4).**
  Mostly settled; the one defect is F1.
  D5 states the tier (`planglyph`, `resolvePass`), that `ValidateDispatch` runs it because it reads only plan text, and that an arrow on a `plan:` handle is a blocking finding.
  The bounded under-reporting and over-reporting of caller coverage are stated, as the rubric's attack-surface question requires.

I checked the premises D2 leans on against the code: `internal/websterengine/doc.go` states sequential one-fork-at-a-time execution, and `CheckBatchOrder` exists in `sequence.go`.
`LaterDeleteReferences` runs inside `resolvePass`, so `delete-before-reference` does run at dispatch; D5's "beside `delete-before-reference`" is only loosely worded, and its last Bound bullet states the plan-gate-only placement unambiguously, so I raised nothing.
