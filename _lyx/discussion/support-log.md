# Support log — plan-glyph-precision

## Interview

Autonomous mode: no operator was present, so the interview ran as exploration plus self-picks.

1. Board entry read: four strands — glyph-use test first, member-glyph `Edit:` targets in the stencil and rubric, verdict stability between Plan-Write and dispatch, mechanical caller coverage for deletions.
   `bugfix-webster` is off the board and its card-order checks are on this branch, so the dependency is met.
2. Containment: `internal/websterengine/doc.go` states cards run one at a time in one worktree, and `sequence.go` only asserts that batches follow plan card order.
   Both containment tiers guard blind parallel dispatch, which no longer happens; the board's second option ("do not refuse overlaps between cards in different batches") generalises to every pair of cards, since cards inside a batch also run in sequence.
   Picked: retire both cross-card checks (D2).
3. Redundancy: a `plan:` handle has no file until built, so a file-versus-member redundancy check run at dispatch would see a pair the plan gate could not.
   Picked: plan gates only (D3), with handles counted as members of their declared unit in the pure tier.
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

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): retire cross-card containment or scope it by batch? — Retire (D2); webster is sequential.
- Q2 (resolved, auto-pick): where does the same-card redundancy check run? — Plan gates only (D3).
- Q3 (resolved, auto-pick): blocking or informational redundancy? — Blocking at plan gates (D3).
- Q4 (resolved, auto-pick): adjust or retire `handle-unreferenced`? — Retire (D4).
- Q5 (resolved, auto-pick): how does a plan mark a re-signed member? — `glyph -> head` arrow on an `Edit` bullet (D5).
- Q6 (resolved, auto-pick): caller coverage by batch or by card order? — Card order, same or earlier card (D5).
- Q7 (resolved, auto-pick): caller coverage severity for methods and fields? — Informational; blocking for package-level members (D5).
- Q8 (resolved, auto-pick): how is the glyph-use test made durable? — Committed fixture, integration tier, shape classes from real plans (D1).
- Q9 (resolved, auto-pick): what happens to a defect rooted in quarry? — Row asserts current behavior with a comment; named in the summary (D1).
- Q10 (resolved, auto-pick): may a test file stay a file path? — Yes when the card adds tests to it; a test function changed in place is a member (D6).
- Q11 (resolved, auto-pick): plan card order? — Test, fixes, D2, D4, D3, D5, stencil and rubric last (D7).
