# Card 9 — resume-circling

**Edit:**
- `internal/shedadapters/circling.go`
- `internal/shedadapters/circling_test.go`
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/bouncer_escalation_test.go`
- `internal/shedadapters/bouncer_circling_test.go`
- `internal/shedadapters/doc.go`
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils/claims_test.go`
- `internal/loomcli/resume.go`
- `internal/loomcli/resume_test.go`
- `internal/loomcli/circling.go`
- `internal/loomcli/circling_test.go`
- `internal/loomcli/cli.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/loomrecipe/bouncerrows.go`

**Intent:**
Make `lyx loom resume` resume a recorded circling decision, the step the hub's role already names (D6, issue #446).

- `shedadapters` gains `PendingCirclingDecision(runDir string) (bool, error)`: whether the run directory's latest round carries a decision file that is not settled, read through `readCirclingDecision`; a malformed file is an error.
- `resume.go`, for a run `awaiting`: when `loomrecipe.BouncerRunSubdir` names the current producer a Bouncer row and its run directory under `loomengine.LoomReviewsDir` has a pending decision, the verb takes the halted path unchanged: the run-lock, live-driver, park-marker and unfinished-merge checks and refusals apply as today, and a parked live driver is woken, whose step re-calls the Bouncer, which acts on the decision.
  Awaiting at a Bouncer row with no pending decision refuses, naming `lyx loom circling accept <slug>` / `lyx loom circling continue <slug>` and then `lyx loom resume`.
  A malformed or unreadable decision file refuses with the error, naming the file, and the way forward to fix or delete the named decision file, then re-run `lyx loom resume`.
  A failed recipe lookup refuses with the way forward of the circling verbs' own row: the embedded loom recipe failed to parse, so this lyx build is broken; rebuild or reinstall lyx, then re-run `lyx loom resume`.
  Awaiting at any other row, PR-Gate included, keeps today's approve/reject refusal byte for byte.
  The verb's `Long` table gains the Bouncer-row rows.
- `circling.go` (loomcli): the verbs' `resume` output field becomes `lyx loom resume`; `circlingWayDecided` names `lyx loom resume`; the file header, the group's and both verbs' `Long` texts name `lyx loom resume` instead of `lyx loom start`.
- `bouncer.go`'s `escalationReason` names `lyx loom resume` in the task worktree as the resume step, after the circling verbs.
  The Stuck-path ways forward for an unreadable or settled decision and an unreadable or unwritable escalation record stay on `lyx loom start`, since those halt the run rather than leave it awaiting.
- `bouncer-template-escalation.md` step 3 resumes with `lyx loom resume` in the worktree; `claims_test.go`'s escalation row pins `lyx loom resume` and a `mustNot` on `lyx loom start`.
- `doc.go` (shedadapters): the escalation paragraph names the `lyx loom resume` resume.
- `cli.go` (loomcli): the loom root command's `Long` text on the `circling` subtree says `lyx loom resume`, not `lyx loom start`, then resumes the run.
- `docs/overview.md`: the review-segment escalation paragraph's fork resumes the run with `lyx loom resume`, and a batten-driven run is resumed the same way with `lyx loom resume` in the task worktree.
- `refusal-spec.md`: the `resume: awaiting` row splits into the Bouncer-row refusal with no decision (new) and the other-row refusal (unchanged); two new rows, `resume: circling decision unreadable` and `resume: recipe lookup failed`, carry the two read-error refusals with the ways forward above; and the `circling: decision already recorded` row names `lyx loom resume`.

Bound: the new edge wakes a driver only for an await at a Bouncer row whose decision file exists and is not settled; it gives no decision itself, never spawns a driver, and every refusal of the halted path still applies.
`lyx loom approve` / `reject` outputs, PR-Gate's awaiting refusal and batten's decision reading are unchanged.

Grep the tests for the old wording: `bouncer_escalation_test.go` and `bouncer_circling_test.go` assert `lyx loom start` in the Awaiting Reason, and `circling_test.go` asserts `"resume": "lyx loom start"`; each moves to `lyx loom resume`, while the Stuck-path assertions on `lyx loom start` stay.

Tests:

- `circling_test.go` (shedadapters): `PendingCirclingDecision` is false with no decision, true for a pending accept or continue, false for a settled accept, and an error for a malformed file.
- `resume_test.go`: rows for awaiting at a Bouncer row with a pending decision and a parked live driver (woken), with a pending decision and no park marker (`driver_not_parked`), with no decision (refused naming the circling verbs and `lyx loom resume`), with a malformed decision file (refused naming the file and its fix-or-delete way forward), with a failing recipe lookup, injected through a Bouncer-subdir lookup function the verb takes as the circling verbs' `bouncerSubdir` dependency does (refused naming the rebuild way forward), and awaiting at PR-Gate (today's refusal); the table's start-as-retry check keeps holding.
- `circling_test.go` (loomcli): both verbs output `"resume": "lyx loom resume"`, and the decided refusal names it.

**ImpactSummary:** One new resume edge over an awaiting Bouncer row plus reworded ways forward; the PR-Gate, approve, reject and halted paths are unchanged.
