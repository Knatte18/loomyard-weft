# Decision record — run-reentry

## Goal

A goto, a resume or a stop keeps what the run already settled: a converged or accepted review verdict, the spent review budget and the reason a run is paused.
A review segment re-entered after its verdict settled reviews only the task's change since then, under a small allowance, or reviews nothing when nothing changed.
A bounce-budget-exhausted halt is lifted only by the parent's resume, never by a goto.
A stop starts no friction session, and a review round whose chair spawned too few forks is retried instead of failing the run.

## Scope

In:

- Shed's goto and re-launch semantics: what a goto keeps, what it resets, and the pause reason it records (`internal/shedengine`, `internal/shedverbs`, the goto help texts of `lyx loom goto`, `lyx shed goto` and `lyx batten goto`).
- A persisted pause reason on every write of `paused`, with the pause verbs' caller label, read by the Go driver.
- The Bouncer's settle record and its handling of a re-entered segment: re-settling an unchanged subject, and an amendment generation with a small allowance and a scoped review (`internal/shedadapters`, its wiring in `internal/shedrecipe`, `internal/loomcli`, darn's wiring, and PR-Rework's run-directory archive in `internal/loomshed`).
- One run-wide amendment allowance key in `loom.yaml` and `darn.yaml`.
- The bounce-budget-exhausted halt: its way forward, its resume grant and its driver rule.
- Every refusal, halt reason, help text and stencil that names a goto as the way to a fresh budget is rewritten to D1–D3.
- Removing the per-stop friction reflection; halt notes stay and Friction-Reflect covers them at the run's end.
- A retried Burler review attempt on a fork shortfall, and a halt error that says how the chair ended.
- Package docs (`doc.go`) of every package whose observable behaviour changes, in the same commit.

Out:

- The Webster reroute edge (`Webster` → `Plan-Burler-Review`) keeps its episode rule; its round N+1 makes the Plan-Review generation unsettled, and the Bouncer judges it as any round.
- The tackler and the arbiter stay in the stop chain; only the per-stop friction session goes.
- A tender pause or hibernate interrupts sessions and writes no shed status, so it records no `pause_reason`; holding work under a tender pause is `tender-pause-holds-reprompt`'s.
- Card-requested halts (`card-halt-is-question`), process ownership (`run-process-ownership`) and the shed-wide arbiter (`shed-arbiter-all`) stay with their own entries.
- Shuttle's session-state reconciliation stays as it is: a `done` with every output file present is completion, per `PATTERN-completion-signal`.
- Removing the hub orch rule "A review segment that already converged" from `_board/_lyx/orch/rules.md` is the orch's step through `lyx orch rules` after this task lands, never a card of this run.
- D5 and D6 build the "review verdict bound to content" and "incremental re-verify" leads of `orchestrator-survey-ideas`; marking them covered is the orch's step.

## Decisions

### D1. Goto keeps the segment's spent budget

A goto entry no longer ends any row's bounce-budget episode: `episodeStuckCount` counts across an `OutcomeGoto` entry as across any other non-Done entry.
This holds for `Goto` and for `Relaunch`'s goto alike.
Only the `OutcomeGoto` branch goes; a reroute into a segment still ends its episode, as do a Done by the row or a row of its segment and a budget-capped entry (D3).
Every reader of the count follows, the Bouncer's `Bounces` seam included, since it counts through `Routing.Bounces`.
A goto still resets what the target row needs to run: `error`, `transient`, `pending_by`, and for `Goto` alone the pause conditions; it moves `current_producer` and writes `paused`.

Rationale: the operator decided on 2026-10-19 that taking another round does not reset the review budget.

### D2. Goto never re-runs a settled review, bar the verify retry

A goto onto a review or fix row of a segment whose Bouncer holds a settled settle record (D5) is refused, with the status file unchanged.
Its way forward names the segment's Bouncer row, which re-settles at no review cost (D5), and the writer row, which starts an amendment (D6).
One goto is admitted onto a settled segment's review row: when the run's Publish failure record names that row as its target, which is loom's verify retry onto `Webster-Burler-Review`.
That round runs as today, as round N+1 of the settled generation, which makes the generation unsettled.
Shed knows no review state, so the check is a told seam on `GotoRequest` and `RelaunchRequest`, filled by loom and darn; batten tells none.
A refusal inside the arbiter's or tackler's re-launch hands back to the parent with the refusal text.

Bound: the seam only refuses; it never moves a run or writes a file, and a seam error refuses with the error and a way forward to re-run the goto.
The verify retry bypasses the refusal only for the one row the failure record names.

### D3. A budget-exhausted halt is lifted by the parent's resume

The Stuck entry of a bounce-budget-exhausted halt is marked `budget_capped`, and `episodeStuckCount` stops counting at it, as it does for the reroute cap's `reroute_capped` entry.
So resuming that halt grants the row one more budget, and nothing else does.
The halt's reason keeps its `ReasonBounceBudgetExhausted` prefix; its way forward is a told clause naming the recipe's resume verb, as the reroute cap's `RerouteCapWayForward` is.
The Go driver gets a rule ahead of every rule that spawns the tackler or the arbiter: a blocked stop whose reason starts with `ReasonBounceBudgetExhausted` hands back to the parent.
The rule lives in `internal/sheddriver`, so it covers every shed-driven recipe.
A Bouncer whose latest round settled never reaches this halt, since D5 re-settles it without returning a Stuck; the board's converged-round case is covered that way.

Bound: one resume of one budget-exhausted halt grants exactly one more budget to that row; the driver never resumes it on its own.

### D4. Every write of paused records a reason

The status file gains `pause_reason`, Shed-owned like `state`: written by every Shed persist, `Goto` and `Relaunch` that records `paused`, cleared by every other one of them, and carried through unchanged by every outside writer (the pause verb, `RecordResumer`).
The reasons:

- A pause requested through the pause verb: `pause requested` followed by the verb's caller label.
  The pause verbs (`lyx loom pause`, `lyx shed pause`, `lyx batten pause`) take a `--by` label as goto does, stored beside `pause_requested` and cleared with it.
- A cancelled step: the existing `reasonCancelled`.
- A fired `pause_before` or `pause_after` condition: its condition reason.
- `Goto`: `goto to <row>`, followed by the goto's `by` label when one is set.
- `Relaunch` with a target: `re-launched onto <row>`.

`lyx loom status`, `lyx shed status` and the drive report show the reason.
The Go driver's paused rule reads the reason from the stop or, when the stop came from the status, from `pause_reason`.
A paused status with no reason, as an older binary writes it, is labelled "paused with no recorded reason" in the hand-back report and keeps the existing friction note.

Rationale: a paused run must say who or what paused it.

### D5. A Bouncer's settle is recorded with its subject, and a re-entry compares against it

The Bouncer writes a settle record in its run directory through `internal/state` whenever it settles a generation approved: at a harvested CONVERGED verdict or a pending accept, before carry-over, Approve and Commit, and on every `approveWithoutReview` path (the skip seam, a zero-round perch, a zero allowance).
The record holds the subject's identity and the base the identity was taken at, and is marked settled once Commit returns.
The subject's identity comes from a told seam per segment:

- a document segment (Discussion, Plan): the reviewed artifact files, minus what Approve itself writes;
- a code segment (Webster-Review, Darn-Review): the task's own change, the diff of the tree against its merge base with the parent branch, so a Merge-In alone changes no identity;
- Merge-Review: the merge-in it reviewed.

At Call entry, at any round including 0, the Bouncer reads the settle record before anything else:

1. A record whose identity equals the live subject: nothing changed since the verdict, or the settle was interrupted (a failed commit, a crash).
   The Bouncer runs carry-over, Approve and Commit again, marks the record settled and returns Done; no seed, judge or review spawns, and no Stuck is returned, so no budget is spent.
   A failing step returns an error, as today.
2. A record whose identity differs: an amendment, handled by D6.
3. No record, and a settled verdict on disk: a generation an older binary settled; today's clear and full re-seed.

Every archive of a review run directory keeps the settle record in the new directory: the Bouncer's own generation archive and PR-Rework's archive of the review run directories.
The Approve and Commit seams must be idempotent: a re-run over an approved, committed subject succeeds and commits nothing.

Rationale: the operator decided on 2026-10-19 that once an artifact is approved, no further review runs on it, and a failed mechanical step after a converged verdict re-runs only that step.

### D6. An amendment generation runs under a small allowance, scoped to the change

When D5 finds the subject changed, the Bouncer archives the generation as today and starts an amendment generation, marked so in a generation record in the run directory.
Its soft budget is the allowance of D7, clamped to the perch's soft cap, and its hard cap is the round that allowance reaches, clamped to the perch's hard cap.
An allowance of 0 settles the amendment approved with no review round, through `approveWithoutReview`, and writes its settle record.
The amendment's seed, reviewers and judge are told the base the settle record holds, so they review the task's change since the reviewed subject, and, when the re-entry follows a PR-Gate reject, the reject's review file.
Every route back into the segment reaches this path through the Bouncer, PR-Gate's reject routes included: loom's PR-Rework into Plan-Bouncer and darn's Darn-Write into Darn-Bouncer.
PR-Rework's exempt-generation skip (`PlanReviewSkippable`) runs ahead of it, skips the review entirely and, through `approveWithoutReview`, writes the settle record the next re-entry compares against.

Bound: an amendment of any size gets the allowance, a plan the writer rewrote whole after a goto to Plan-Write included; the judge, the soft-cap rule and the escalation verbs work as in any generation, so an amendment that needs more rounds escalates to the parent.
Where the `Bounces` seam is unwired or errors, Shed's generic block applies the perch's own budget; loom and darn always wire the seam.

### D7. One run-wide amendment allowance key

`reviewbudget` declares `review_amend_max_bounces`, read from `loom.yaml` and `darn.yaml` through the same config stack as the other review budget keys, a task's `run_settings` included.
Its template default is 1, and 0 is the off switch D6 describes.
There is no per-perch amendment key.

### D8. No friction session per stop

A `blocked` or `failed` halt still writes its halt note (`loom-halt`) through `friction.NotePath`, but spawns no reflection: `reflectHalt` and its callers under step, the loop and arm write the note and report the friction status `skipped`.
The run's Friction-Reflect row reflects every note in the friction directory, as it does today.
So no stop holds the driver lock for a friction session, and `lyx loom resume` never waits on one.
The bootstrap allowance for a reflection after a halt (`friction_timeout_min` in `internal/loomcli/bootstrap.go`) is retired with it.

Rationale: the operator's view on 2026-10-19 that friction need not run at every stop.

### D9. A review round with too few forks is retried, and its failure says why the chair ended

In the Burler review row's attempt loop, a review that fails with `ErrClusterForksMissing` on attempt 1 is retried: its outputs, the chair's review file included, are archived and a fresh attempt 2 runs, as after a died or timed-out attempt 1.
The shortfall arrives as an error from the review run, so this is a new branch on the error path.
A shortfall on attempt 2 is a hard failure as today, and so is any other fork violation on either attempt.
The failure's error, and so the halt note, names the requested and spawned fork counts, the chair's strand, and the session states shuttle recorded for the chair (for example an interrupt before `done`); the review result carries those states to burler.

Rationale: the chair's `done` with no forks in issue #756 was a stalled round, not a design fault, and the halt gave no cause.

## Constraints

- `PATTERN-tolerant-records`: `pause_reason`, the pause label, `budget_capped`, the settle record and the generation record decode from files that lack them; every write goes through `internal/state`.
- `PATTERN-refusal-way-forward`: D2's refusal and D3's halt reason name their way forward, and a test reaches each.
- `PATTERN-no-denied-recovery`: every way forward names only commands the agents' settings allow.
- `PATTERN-shed-verb-set`: the goto and pause bodies stay generic in `internal/shedverbs`; the D2 seam and D3's way forward are told by the recipe owners.
- `PATTERN-told-geometry` and `PATTERN-shed-producer-seam`: shedengine and shedadapters get the subject identity, the settled-segment check and the allowance as told values or seams, never a path they derive.
- `PATTERN-shed-driver`: D3's rule lives in `internal/sheddriver`'s rule table.
- `PATTERN-recipe-setting-lookup`: the allowance reaches the Bouncer row as a `SegmentSettings` field or `module.key` lookup, never its own `Env` field.
- `PATTERN-fabric-git`: the code-segment identity's git reads go through the existing told seams and `internal/fabricengine`, never raw git.
- `PATTERN-friction-capture` changes with D8; the note is still written, and archived only once a reflection covers it.
- `PATTERN-completion-signal`: D9 changes burler's handling of a review result, never shuttle's completion test.
- A status file, run directory or friction directory written by an older binary keeps working.

## Auto-mode assumptions

- The board's "keeps its spent budget and gets at most a small allowance on top" is read as the allowance alone (D6).
- One run-wide allowance key suffices (D7).
- With goto keeping the budget, the parent's resume of a budget-exhausted halt is the way past it (D3).
- The #759 failed-then-paused sequence is most likely `Relaunch`'s goto, which today writes `paused` with no reason; D4 covers it and every other paused writer.

## Open risks

- The subject identity must exclude what Approve writes, or every re-entry reads as an amendment; the identity seam is the one place that decides it per segment.
- An in-flight run deployed onto this binary has no settle records, so its next re-entry into a settled segment re-reviews in full (D5 case 3).
- Halt notes are reflected only at the run's end; `run-process-ownership`'s planned `lyx batten abandon`, which removes a run's records, must reflect or keep a run's unreflected notes to stay within `PATTERN-friction-capture`.

## Acceptance criteria

- A goto into a segment with Stucks in its current episode leaves the count where it stood, for Shed's block and for the Bouncer's `Bounces` seam alike; a reroute still ends its target segment's episode.
- A goto onto a review or fix row of a settled segment is refused with the file unchanged and a way forward naming the Bouncer and the writer row; a goto onto the Bouncer itself is admitted, and so is loom's verify retry onto the row its Publish failure record names.
- An arbiter or tackler re-launch refused by D2 hands back with the refusal text.
- A budget-exhausted halt names the recipe's resume verb, not a goto; its resume grants exactly one more budget to that row, through the `Bounces` seam too; the Go driver hands that halt back without spawning the tackler or the arbiter, also after a tackler resume returned to it.
- Every path that writes `paused` writes a non-empty `pause_reason`, a pause verb's `--by` label included; any later Shed write clears it, and the pause verb's own writes carry it through.
- A driver re-launch onto a target after a failed review row leaves a non-empty `pause_reason`, and a paused status with an empty one is labelled in the drive report as having no recorded reason.
- A Bouncer whose CONVERGED settle failed at Commit, resumed with an unchanged subject, re-runs only the settle and returns Done; no seed or judge spawns and the budget count is unchanged.
- A Bouncer whose CONVERGED settle failed at Commit, followed by a goto to the writer row that changed the subject, runs an amendment generation, never an unreviewed approve.
- A Bouncer re-entered with an unchanged subject returns Done with no spawn, also after a skip-seam or zero-allowance approval and after a Merge-In that changed only the parent's side.
- A Bouncer re-entered after the subject changed runs an amendment generation whose seed is told the settle record's base, whose soft cap is `review_amend_max_bounces` clamped to the perch's soft cap, and which escalates at that allowance.
- With `review_amend_max_bounces: 0`, an amendment settles approved with no review spawn and writes a settle record.
- After a PR-Gate reject, the re-entered review segment (loom's Plan-Review after PR-Rework's archive, darn's Darn-Review via Darn-Write) runs an amendment generation whose seed names the reject's review file.
- A settled run directory with no settle record is cleared and re-seeded as today.
- A `blocked` and a `failed` halt write a `loom-halt` note and start no reflection; the Friction-Reflect row reflects the note at the run's end.
- A Burler review round whose attempt 1 spawns fewer forks than requested runs attempt 2; a shortfall on both fails with an error naming the counts, the chair's strand and its session states.
- No refusal, halt reason, help text or stencil names a goto as the way to a fresh budget; the goto help texts say that a goto keeps the segment's verdicts and spent budget, what it resets, and that it records itself as the pause reason.

## PATTERN changes

- `PATTERN-friction-capture` — changes: Halting a loom run or refusing in webster: Go writes a friction note, reflected by the run's Friction-Reflect row rather than at the stop, and archived only once a reflection covers it.

## Notes for the plan writer

- The `settledGeneration` branch of `Bouncer.Call` is where D5 and D6 attach; its Warn log already names the commit-seam-failure case D5 fixes.
- In `episodeStuckCount`, D1 removes the `OutcomeGoto` call of `gotoEndsEpisode` and keeps the reroute call; the reroute cap's `reroute_capped` handling is the model for D3's `budget_capped`.
