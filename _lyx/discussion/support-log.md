# Support log — plan-glyph-precision

## Interview

Autonomous mode: no operator was present, so the interview ran as exploration plus self-picks.

1. Board entry read: four strands — glyph-use test first, member-glyph `Edit:` targets in the stencil and rubric, verdict stability between Plan-Write and dispatch, mechanical caller coverage for deletions.
   `bugfix-webster` is off the board and its card-order checks are on this branch, so the dependency is met.
2. Containment: `internal/websterengine/doc.go` states cards run one at a time in one worktree, and `sequence.go` only asserts that batches follow plan card order.
   Both containment tiers guard blind parallel dispatch, which no longer happens; the board's second option ("do not refuse overlaps between cards in different batches") generalises to every pair of cards, since cards inside a batch also run in sequence.
   Picked: retire both cross-card checks (D2).
   `webster-parallel-execution` would reintroduce parallelism in separate worktrees, with its own merge gate; it is a parked note and designs its own guard if built.
3. Redundancy: a `plan:` handle has no file until built, so a file-versus-member redundancy check run at dispatch would see a pair the plan gate could not.
   Picked: the file-versus-member pairing at plan gates only (D3), with handles counted as members of their declared unit in the pure tier, whose verdict is then stable when record-batch binds the handle.
4. `handle-unreferenced`: a `Create` bullet is its own card's use of the handle, so the board's "does not fire for a handle its own declaring card uses" means the check never fires; picked retirement (D4).
   Binding and done-checks still catch a handle that is not built.
5. Caller coverage: there is no quarry reference API; `deleteorder.go` already scans identifiers by name.
   The format had no way to mark which `Edit` member changes its signature; picked the `glyph -> head` arrow on `Edit` bullets (D5), reusing the `Create`/`Rename` grammar.
   Coverage is judged by card order, not batch, because cards run in card order and the partition is unknown at Plan-Write.
6. Glyph-use test: real plans' trees drift and `card-fabric-reference` forbids live corpora, so the test is a committed fixture with one row per shape class harvested from real plans (D1).
7. Stencil and rubric wording (D6), and card order within the plan (D7).

## Rejected alternatives

- Placing a `plan:` handle by its declared file at plan time: a handle declares a package unit, not a file, so containment would have to guess the file; adding a file field to the handle grammar only to feed a check guarding a hazard that no longer exists.
- Skipping containment only between cards in different batches: the partition is computed at first webster init and may change on rebaseline, so the verdict would still depend on state the plan gate cannot see; cards inside a batch also run in sequence, so the batch boundary is not the line that matters.
- Running redundancy at dispatch too: reintroduces the verdict change.
- Making redundancy informational everywhere: the board wants it flagged, and informational findings do not stop `validate-plan`.
- Keeping `handle-unreferenced` but exempting self-use: every handle is used by its declaring card's `Create` bullet, so the exemption is the same as removal with more code.
- Inferring re-signs from Intent prose, or treating every `Edit` member as re-signed: the first is not mechanical; the second forces callers of every body-only edit into the card.
- A new `**Resign:**` label: a seventh-plus card type for what is an `Edit`; the arrow keeps it one label.
- Blocking caller coverage for methods and fields: the name scan matches any `.Name` selector in the tree.
- An audit tool over weft history instead of a committed fixture: would read live files of another repo and could not run in the test suite.
- A plan-gate-only mode flag on `planparser.ValidateFormat` for the pure redundancy pairings: they are state-stable, so the split buys nothing; only the resolve-backed pairing and caller coverage need keeping out of `ValidateDispatch`.
- Adding the plan-gate-only checks inside `resolvePass` behind a parameter: `ValidateDispatch` shares that pass, so a separate pass it never calls is the simpler seam.
- The test card carrying the fixes its own rows expose: a card's targets are fixed at plan time, and the Plan producer cannot foresee which defects the probe finds until it has run it, so the fixes need their own cards written after the probe.
- A second `quarry.Name` call site for the re-sign heads, added to the chokepoint pin: two batched calls each need their own length-plus-echo guard, where joining the existing call reuses its guard.
- Reusing `deleteorder.go`'s line-textual patterns for caller coverage: a package-level name's same-package pattern matches comments, strings, selectors and struct keys, which turns generic names (`New`, `Load`, `Run`) into routine blocking findings.
- Coverage of a re-signed member's caller by an earlier card: an unrelated earlier edit of the caller file would satisfy the check while the changing card still breaks the caller.
- Caller coverage for a `Rename` Old side: the spec's Rename mechanic rewrites every reference itself and needs no `ImpactSummary`, so listing caller files would add a blast-radius burden the mechanic exists to avoid; amending the spec's Rename text to demand it buys nothing the grep verify and build gate do not already give.
- Caller coverage for file and package `Delete` targets, by listing the file self glyph's members: a second resolve-backed enumeration for a case webster's build and test gate and `delete-before-reference`'s import match already bound.
- An arrow head for an interface method or struct field written as the enclosing type's head: it names the type, not the member, and would need a special case in the head-to-member match.
- Comparing the re-sign head with the built declaration at record-batch: the arrow's job is selecting members for caller coverage, and the build and test gate already catches a wrong signature.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): retire cross-card containment or scope it by batch? — Retire (D2); webster is sequential.
- Q2 (resolved, auto-pick): where does the same-card redundancy check run? — File-versus-member at plan gates only; pure pairings wherever `planparser.ValidateFormat` runs (D3, refined by Q12).
- Q3 (resolved, auto-pick): blocking or informational redundancy? — Blocking at plan gates (D3).
- Q4 (resolved, auto-pick): adjust or retire `handle-unreferenced`? — Retire (D4).
- Q5 (resolved, auto-pick): how does a plan mark a re-signed member? — `glyph -> head` arrow on an `Edit` bullet (D5).
- Q6 (resolved, auto-pick): caller coverage by batch or by card order? — Card order: same or earlier card for a `Delete`, same card for a re-sign (D5, refined by Q21 and Q23).
- Q7 (resolved, auto-pick): caller coverage severity for methods and fields? — Informational; blocking for package-level members (D5).
- Q8 (resolved, auto-pick): how is the glyph-use test made durable? — Committed fixture, integration tier, shape classes from real plans (D1).
- Q9 (resolved, auto-pick): what happens to a defect rooted in quarry? — Row asserts current behavior with a comment; named in the summary (D1).
- Q10 (resolved, auto-pick): may a test file stay a file path? — Yes when the card adds tests to it; a test function changed in place is a member (D6).
- Q11 (resolved, auto-pick): plan card order? — Test, fixes, D2, D4, D3, D5, stencil and rubric last (D7).
- Q12 (resolved, auto-pick, review round 1): how are the new checks kept out of `ValidateDispatch`? — Pure redundancy pairings run everywhere, being state-stable; the file-versus-member pairing and caller coverage run in a pass `ValidateFormat` and `Validate` call and `ValidateDispatch` does not (D3, D5).
- Q13 (resolved, auto-pick, review round 1): how does a defect found by the D1 test become a card? — The Plan producer probes the chain during exploration and writes one fix card per defect; a missed one fails its batch and goes through edit, `rebaseline --card`, `recover-batch` (D1).
- Q14 (resolved, auto-pick, review round 1): what reads the re-sign head? — Nothing downstream; it is parse-checked and member-matched, and the build and test gate bounds a misstated signature (D5).
- Q15 (resolved, auto-pick, review round 1): which files does caller coverage scan? — Every `.go` file of the tree, tests and build-tagged files included, `testdata` and `vendor` excluded; alias and dot imports are let through and bounded by the build gate (D5).
- Q16 (resolved, auto-pick, review round 2): when does a handle stop being a handle? — At record-batch's `BindHandles`; `CanonicalizeHandles` runs at every gate and keeps it a handle, and a `Rename` to-side handle counts under its Old glyph's unit (D3).
- Q17 (resolved, auto-pick, review round 2): where does the re-sign head-to-member match run? — `planglyph`, under `resolvePass`, at every gate including dispatch, since it reads only plan text; an arrow on a `plan:` handle is blocking (D5, seam refined by Q20).
- Q18 (resolved, auto-pick, review round 2): what backstops an unmarked signature change? — A Plan-Review rubric finding for an `Edit` member whose Intent changes its signature without the arrow (D6).
- Q19 (resolved, auto-pick, review round 2): which card removes the stencil's `handle-unreferenced` sentence? — D6's stencil card; D4's card removes the check and spec row only.
- Q20 (resolved, auto-pick, review round 3): how does the head-to-member match reach `quarry.Name` under `planglyph`'s chokepoint pin? — Through `CanonicalizeHandles`' one batched call, the heads a second declaration source whose answers are compared with their glyphs and kept out of canonicalization (D5).
- Q21 (resolved, auto-pick, review round 4): may an earlier card cover a renamed or re-signed member's caller? — No: its callers change in the same card, since an earlier card cannot call a name or signature that does not exist yet and every card leaves the build green; a `Delete` keeps same-or-earlier coverage (D5; Rename dropped from the check by Q23).
- Q22 (resolved, auto-pick, review round 4): how does caller coverage keep package-level false positives down? — A `go/scanner` token walk that skips comments and strings, and a selector for a package-level name; `deleteorder.go`'s line patterns are not reused (D5).
- Q23 (resolved, auto-pick, review round 5): does a `Rename` Old side get caller coverage? — No: the spec's AST-aware Rename mechanic updates every reference, bounded by its grep verify and the build gate; the spec's Rename text is unchanged (D5).
- Q24 (resolved, auto-pick, review round 5): which covers can the deleting or re-signing card itself carry, given D3? — A member glyph whose span holds the reference, or another file; the declaring file and the package self glyph only on an earlier card, for a `Delete`; references in the declaring file outside the member's span count (D5).
- Q25 (resolved, auto-pick, review round 5): are file and package `Delete` targets caller-covered? — No; bounded by the build gate and `delete-before-reference`'s import match (D5).
- Q26 (resolved, auto-pick, review round 5): which members take the re-sign arrow? — Those one Go declaration names; an interface method or struct field takes none, and the missing-arrow rubric finding exempts them (D5, D6).
- Q27 (resolved, auto-pick, review round 5): what is a member's own package for the reference walk? — The declaring directory's files sharing the declaring file's package clause; an external test package there is matched by the qualified form (D5).
