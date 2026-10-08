# Card 9 — resume-circling

**Create:**
- `plan:internal/shedadapters#PendingCirclingDecision` -> `func PendingCirclingDecision(runDir string) (bool, error)`

**Edit:**
- `internal/shedadapters/circling_test.go`
- `internal/shedadapters#Bouncer.escalationReason`
- `internal/shedadapters/bouncer_escalation_test.go`
- `internal/shedadapters/bouncer_circling_test.go`
- `internal/shedadapters/doc.go`
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils/claims_test.go`
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.resumeFromRunState`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli/resume_test.go`
- `internal/loomcli#circlingWayDecided`
- `internal/loomcli#circlingVerb`
- `internal/loomcli#loomCLI.circlingCmd`
- `internal/loomcli/circling_test.go`
- `internal/loomcli#Command`
- `internal/battenshed#awaitingHandOff`
- `internal/battenshed#innerRunProducer.awaitingStep`
- `internal/battenshed/innerrun_awaiting_test.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/loomrecipe/bouncerrows.go`

**Intent:**
Make `lyx loom resume` resume a recorded circling decision, the step the hub's role already names (D6, issue #446).

- `shedadapters` gains `PendingCirclingDecision(runDir string) (bool, error)` in `circling.go`, whose file header then names `lyx loom resume` among the decision file's readers: whether the run directory's latest round, resolved through `ResolveRound` as `RecordCirclingDecision` resolves it, carries a decision file that is not settled, read through `readCirclingDecision`.
  Every error names the path it concerns: the run directory or a report file from `ResolveRound`, which already name theirs, or the decision file, whose parse errors `readCirclingDecision` returns without the path and `PendingCirclingDecision` wraps with it.
  A settled accept never sits under an `awaiting` status, since the Bouncer settles it only inside a step that has already moved the run to `running`, so its `false` needs no refusal of its own.
- `resume.go`, for a run `awaiting`: the verb looks the current producer up through a new `loomCLI` field `bouncerSubdir`, which `wire` fills with `loomrecipe.BouncerRunSubdir` and a test replaces, as the circling verbs' own `bouncerSubdir` dependency is; `resumeFromRunState` keeps its signature.
  When the lookup names a Bouncer row and its run directory under `loomengine.LoomReviewsDir` has a pending decision, the verb takes the halted path unchanged: the run-lock, live-driver, park-marker and unfinished-merge checks and refusals apply as today, and a parked live driver is woken, whose step re-calls the Bouncer, which acts on the decision.
  Awaiting at a Bouncer row with no pending decision refuses, naming `lyx loom circling accept <slug>` / `lyx loom circling continue <slug>` and then `lyx loom resume`.
  A `PendingCirclingDecision` error refuses with that error, which names the run directory, report file or decision file it concerns, and the way forward: fix the file or directory the message names, and for a malformed decision file delete it and record the decision again with `lyx loom circling accept <slug>` / `lyx loom circling continue <slug>`; then re-run `lyx loom resume`.
  A failed recipe lookup refuses with the way forward of the circling verbs' own row: the embedded loom recipe failed to parse, so this lyx build is broken; rebuild or reinstall lyx, then re-run `lyx loom resume`.
  Awaiting at any other row, PR-Gate included, keeps today's approve/reject refusal byte for byte.
  The verb's `Long` table gains the Bouncer-row rows, and the header of `resume.go`, the file that declares the verb, names the awaiting-at-a-Bouncer-row case with a pending circling decision beside the halted case it wakes.
- `circling.go` (loomcli): the verbs' `resume` output field becomes `lyx loom resume`; `circlingWayDecided` names `lyx loom resume`; the file header, the group's and both verbs' `Long` texts name `lyx loom resume` instead of `lyx loom start`.
- `bouncer.go`'s `escalationReason` names `lyx loom resume` in the task worktree as the resume step, after the circling verbs.
  The Stuck-path ways forward for an unreadable or settled decision and an unreadable or unwritable escalation record stay on `lyx loom start`, since those halt the run rather than leave it awaiting.
- `bouncer-template-escalation.md` step 3 resumes with `lyx loom resume` in the worktree; `claims_test.go`'s escalation row pins `lyx loom resume` and a `mustNot` on `lyx loom start`.
- `doc.go` (shedadapters): the escalation paragraph names the `lyx loom resume` resume.
- `cli.go` (loomcli): the loom root command's `Long` text on the `circling` subtree says `lyx loom resume`, not `lyx loom start`, then resumes the run.
- `innerrun.go` (battenshed): `awaitingHandOff`, the hint a batten wait over an awaiting child reports while no PR-Gate decision exists, names both hand-offs instead of approve and reject alone, since `lyx loom approve` refuses at a Bouncer row: `lyx loom approve` or `lyx loom reject` at PR-Gate, after which the batten run resumes the child itself, or `lyx loom circling accept` / `continue` and then `lyx loom resume` in the task worktree at a review segment's escalation.
  `awaitingStep`'s decide-again hint, which fires while an acted-on approval or rejection is still recorded and the child awaits again, and so also over a child a `goto` sent from an approved PR-Gate back into a review segment that escalated, names the same two hand-offs instead of re-running approve or reject alone.
  Batten's decision reading (`ReadDecision`) and its resume of the child are unchanged.
- `docs/overview.md`: the `lyx loom resume` sentence in the loom CLI section, which says resume wakes a halted run's parked driver and refuses every other state, names the awaiting-at-a-Bouncer-row case with a pending circling decision beside the halted case; the review-segment escalation paragraph's fork resumes the run with `lyx loom resume`, a batten-driven run is resumed the same way with `lyx loom resume` in the task worktree, and the batten paragraph on an `awaiting` child names the review-segment escalation's circling verbs and `lyx loom resume` beside PR-Gate's approve and reject.
- `refusal-spec.md`: the `resume: awaiting` row splits into the Bouncer-row refusal with no decision (new) and the other-row refusal (unchanged); two new rows, `resume: circling decision unreadable` and `resume: recipe lookup failed`, carry the two read-error refusals with the ways forward above; and the `circling: decision already recorded` row names `lyx loom resume`.

Bound: the new edge wakes a driver only for an await at a Bouncer row whose decision file exists and is not settled; it gives no decision itself, never spawns a driver, and every refusal of the halted path still applies.
`lyx loom approve` / `reject` outputs, PR-Gate's awaiting refusal and batten's decision reading are unchanged; batten's two awaiting hint texts are the only batten change.

Grep the tests for the old wording: `bouncer_escalation_test.go` and `bouncer_circling_test.go` assert `lyx loom start` in the Awaiting Reason, and `circling_test.go` asserts `"resume": "lyx loom start"`; each moves to `lyx loom resume`, while the Stuck-path assertions on `lyx loom start` stay.

Tests:

- `circling_test.go` (shedadapters): `PendingCirclingDecision` is false with no decision, true for a pending accept or continue, false for a settled accept, and an error naming the decision file for a malformed file.
- `resume_test.go`: rows for awaiting at a Bouncer row with a pending decision and a parked live driver (woken), with a pending decision and no park marker (`driver_not_parked`), with no decision (refused naming the circling verbs and `lyx loom resume`), with a malformed decision file (refused naming the file and its fix-or-delete way forward), with a failing recipe lookup, injected through the `bouncerSubdir` field (refused naming the rebuild way forward), and awaiting at PR-Gate (today's refusal); the table's start-as-retry check keeps holding.
- `circling_test.go` (loomcli): both verbs output `"resume": "lyx loom resume"`, and the decided refusal names it.
- `innerrun_awaiting_test.go` (battenshed): the no-decision wait's hint names `lyx loom approve` and `lyx loom resume`, and `TestInnerRun_AwaitingAgainAfterResumeGetsReapproveHint` and the rejection's decide-again assertions also expect `lyx loom resume` in the decide-again hint; `TestInnerRun_AwaitingRevivesADeadDriverAndKeepsItsHints` drops its assertion that the hint never names `lyx loom resume` and the doc premise that resume refuses every awaiting run.

**ImpactSummary:** One new resume edge over an awaiting Bouncer row plus reworded ways forward in loomcli, shedadapters and batten's awaiting hint; the PR-Gate, approve, reject and halted paths and batten's decision reading are unchanged.
