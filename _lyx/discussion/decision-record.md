# Decision record — run-reentry

## Goal

A goto, a resume or a stop keeps what the run already settled: a converged or accepted review verdict, the spent review budget and the reason a run is paused.
A review segment re-entered after its verdict settled reviews only the task's change since then, under a small allowance, or reviews nothing when nothing changed.
A bounce-budget-exhausted halt is lifted only by the parent's resume, never by a goto.
A stop starts no friction session, and a review round whose chair spawned a fork count other than the one requested is retried instead of failing the run.

## Scope

In:

- Shed's goto and re-launch semantics: what a goto keeps, what it resets, and the pause reason it records (`internal/shedengine`, `internal/shedverbs`, the goto help texts of `lyx loom goto`, `lyx shed goto` and `lyx batten goto`), and the D2 seam where the tackler (`internal/sheddriver`) and the arbiter (`internal/loomcli`) validate a verdict's goto.
- Shed's `generation_start` history mark, at which the episode count stops and whose Stuck skips the budget comparison (`internal/shedengine`).
- A persisted pause reason on every write of `paused`, with the pause verbs' caller label, read by the Go driver.
- The Bouncer's settle record and its handling of a re-entered segment: re-settling an unchanged subject, and an amendment generation with a small allowance and a scoped review (`internal/shedadapters`, its wiring in `internal/shedrecipe`, `internal/loomcli`, darn's wiring, and PR-Rework's run-directory archive in `internal/loomshed`).
- One run-wide amendment allowance key in `loom.yaml` and `darn.yaml`.
- The bounce-budget-exhausted halt: its way forward, its resume grant and its driver rule.
- Every refusal, halt reason, help text and stencil that names a goto as the way to a fresh budget is rewritten to D1–D3.
- Removing the per-stop friction reflection; halt notes stay and Friction-Reflect covers them at the run's end.
- A retried Burler review attempt on a fork-count mismatch, and a halt error that says how the chair ended.
- Package docs (`doc.go`) of every package whose observable behaviour changes, in the same commit.

Out:

- The Webster reroute edge (`Webster` → `Plan-Burler-Review`) keeps its episode rule; its round N+1 report makes the Plan-Review settle record no longer live (D5), and the Bouncer judges that round as any round.
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
Only the `OutcomeGoto` branch goes; a reroute into a segment still ends its episode, as do a Done by the row or a row of its segment, a budget-capped entry not followed directly by a goto entry (D3) and a generation-start entry (D6), which is itself counted.
Every reader of the count follows, the Bouncer's `Bounces` seam included, since it counts through `Routing.Bounces`.
A goto still resets what the target row needs to run: `error`, `transient`, `pending_by`, and for `Goto` alone the pause conditions; it moves `current_producer` and writes `paused`.

Rationale: the operator decided on 2026-10-19 that taking another round does not reset the review budget.

### D2. Goto never re-runs a settled review, bar the verify retry

A goto onto a review or fix row of a segment whose Bouncer holds a live settle record (D5), settled or interrupted, is refused, with the status file unchanged.
Its way forward names the segment's Bouncer row, which re-settles at no review cost (D5), and the writer row, after which a changed subject starts a new generation (D6).
A segment whose record is not live, an amendment generation under way included, admits the goto as today.
A settled segment's review row admits gotos in one case: when the run's Publish failure record names that row as its target and its write time is later than the live settle record's settle time (D5), which is loom's verify retry onto `Webster-Burler-Review`.
That round runs as today, as round N+1 of the settled generation; its report on disk makes the settle record no longer live, so the Bouncer judges it, and the generation's next settle, on the round-by-round path, writes a record whose settle time is later than the failure record's write time.
Shed knows no review state, so the check is a told seam on `GotoRequest` and `RelaunchRequest`, filled by loom and darn; batten tells none.
The tackler and the arbiter run the same seam where they validate a verdict's goto with `CheckGotoTarget`, so a verdict naming a refused row is rejected as invalid there.
A refusal inside the arbiter's or tackler's re-launch hands back to the parent with the refusal text.

Bound: the seam only refuses; it never moves a run or writes a file, and a seam error refuses with the error and a way forward to re-run the goto.
An interrupted settle admits what a settled one admits: the Bouncer row, which re-settles or starts a new generation, and every row outside the segment's review and fix rows.
The verify retry bypasses the refusal only for the one row the failure record names, and only until that generation settles again; after that settle, whose settle time is later than the failure record's write time, a goto onto the row is refused for every caller.
A case 1 re-settle keeps the stored settle time, so a goto onto the Bouncer after a Publish failure leaves the bypass open.
A failure record with no write time, as an older binary writes it, admits the goto as today.

### D3. A budget-exhausted halt is lifted by the parent's resume

The Stuck entry of a bounce-budget-exhausted halt is marked `budget_capped`, and `episodeStuckCount` stops counting at it, as `rerouteCount` does at a `reroute_capped` entry, but only when the entry right after it is not an `OutcomeGoto` entry.
A resume writes no history entry, so after a resume the entry right after the capped entry is the row's own next run, and resuming that halt grants the row one more budget.
A goto onto the halted run writes its entry right after the capped entry, so the count continues past it, over the spent Stucks, and the row blocks again at its next counted Stuck; no goto grants a fresh budget, and a resume after such a goto grants none either.
A later goto, after the row ran on a resume's grant, leaves the capped entry's stop in place, so it keeps that grant's spent count as D1 keeps any spent count.
The halt's reason keeps its `ReasonBounceBudgetExhausted` prefix; its way forward is a told clause naming the recipe's resume verb, as the reroute cap's `RerouteCapWayForward` is.
The Go driver gets a rule ahead of the hand-over rule and of every rule that spawns the tackler or the arbiter: a blocked stop whose reason, or for a stop built from the status its status error, starts with `ReasonBounceBudgetExhausted` hands back to the parent.
The rule lives in `internal/sheddriver`, so it covers every shed-driven recipe.
A Bouncer whose latest round settled never reaches this halt, since D5 re-settles it without returning a Stuck; the board's converged-round case is covered that way.

Bound: one resume of one budget-exhausted halt that no goto directly followed grants exactly one more budget to that row; the driver never resumes it on its own, and never hands it over to a fresh driver, whose first step would resume it.

### D4. Every write of paused records a reason

The status file gains `pause_reason`, Shed-owned like `state`: written by every Shed persist, `Goto` and `Relaunch` that records `paused`, cleared by every one of them that moves the run out of `paused`, and carried through unchanged by every outside writer (the pause verb, `RecordResumer`).
A write that leaves the state unchanged, such as `Relaunch` with no target or with a target already in effect, leaves `pause_reason` unchanged.
The reasons:

- A pause requested through the pause verb: `pause requested` followed by the verb's caller label.
  The pause verbs (`lyx loom pause`, `lyx shed pause`, `lyx batten pause`) take a visible `--by` free-text caller label, such as `operator` or the orch's strand name; it is a label only and gates or skips nothing, and it refuses `shed`, the label reserved for lyx's own automatic steps.
  It is stored beside the pause request it accompanies (`pause_requested`, `pause_before` or `pause_after`) and cleared with it.
  Loom's bare pause stays refused by its `BarePause` hook, so a loom pause's reason is its fired condition's reason followed by the label.
- A cancelled step: the existing `reasonCancelled`.
- A fired `pause_before` or `pause_after` condition: its condition reason, followed by the label stored with it when one is set.
- `Goto`: `goto to <row>`, followed by the goto's `by` label when one is set.
  Goto's `--by` stays hidden and admits only `shed`, so an operator's or the orch's goto records `goto to <row>` alone.
- `Relaunch` with a target: `re-launched onto <row>`.

`lyx loom status`, `lyx shed status` and the drive report show the reason.
The Go driver's paused rule reads the reason from the stop or, when the stop came from the status, from `pause_reason`.
A paused status with no reason, as an older binary writes it, is labelled "paused with no recorded reason" in the hand-back report and keeps the existing friction note.

Rationale: a paused run must say what paused it and, when a pause verb asked, who asked.

### D5. A Bouncer's settle is recorded with its subject, and a re-entry compares against it

The Bouncer writes a settle record in its run directory through `internal/state` whenever it settles a generation approved: at a harvested CONVERGED verdict or a pending accept, before carry-over, Approve and Commit, and on every `approveWithoutReview` path (the skip seam, a zero-round perch, a zero allowance).
The record holds the subject's identity, the amendment base of D6, the generation and round it settled and its settle time; it is marked settled once Commit returns.
A document segment's base is written with the record as the weft HEAD read with the identity, and is replaced by the weft HEAD read after Commit returns when the record is marked settled.
The settle time is the time the record was written by a settle on the round-by-round path, a round N+1 after loom's verify retry or the Webster reroute included, or by an `approveWithoutReview` path; a re-settle in case 1 keeps the stored time.
The Bouncer also keeps a generation record in its run directory: the generation's number, whether it is an amendment, an amendment's base, and the subject identity and base of the latest judge spawn (case 3).
It writes the record at every seed, generation 1 included, and on every `approveWithoutReview` path of a directory that holds none, so a directory without one is an older binary's.
The subject's identity comes from a told seam per segment:

- a document segment (Discussion, Plan): the reviewed artifact files, minus what Approve itself writes, and minus every block Go or `lyx loom decision add` writes into the decision record: the `<!-- lyx:carry-over … -->` blocks and the `Added after Discussion` entries;
- a code segment (Webster-Review, Darn-Review): the task's own change, the diff of the warp working tree against the task branch's merge base with its parent branch, read through the told merge-base seam and a new `internal/fabricengine` patch reader, since `Fabric.Diff` returns changed paths only;
  a standalone run, which records no parent branch, has no code identity, so its code segments write neither a settle record nor a generation record and a re-entry takes case 4.
  An uncommitted change to a tracked file counts as a change, and so does an untracked file that is not ignored, unless it is in the exclusion set:
  the run's own state (webster's run directory, plan directory and scratch directory), as `UncommittedPaths` in `internal/websterengine` excludes it, plus a told list of files the reader skips, which loom and darn fill with the harness's own files, `.scratch_task.json` among them.
  The told list names exact worktree-relative paths of files the harness itself writes, never a pattern or a directory, and exempts a path only while it is untracked, so a tracked file of that name still counts toward the identity;
  a change confined to listed files reads as an unchanged subject, bounded by the verify tiers named below.
  The record keeps the merge base, the commit HEAD stood at, the hash of that full diff, git's stable patch-id of its zero-context form, and the full diff itself as a patch file beside the record in the run directory.
  The subject is unchanged when the merge base is unchanged and the full diff's hash is equal, or when the merge base moved, every first-parent commit since the settle commit came from a Merge-In, and the patch-id is equal.
  A commit came from a Merge-In when it is a merge commit whose second parent is an ancestor of the recorded parent branch's tip, or is itself such an ancestor, as a fast-forward Merge-In leaves it.
  A task commit made after the settle fails that test even when it commits only work the settle already hashed uncommitted, so a Merge-In after it reads as changed; that false amendment is accepted, and it costs at most the allowance.
  The patch-id ignores hunk line numbers and a zero-context diff carries no parent lines, so a Merge-In whose parent changes touch no line the task changed leaves the subject unchanged, while a task-side move of code, which the patch-id alone cannot see, changes the full diff and so is an amendment.
  The patch-id rule lets through, unreviewed by this segment, a parent change that interacts with the task's change without sharing a line with it;
  the verify tiers bound it, at Publish in loom and at darn's Merge-In, which verifies the merged tree, and in loom so does the Merge-Review segment, which reviews the merge-in unless its merge-overlap skip seam or a merge budget of 0 settles it with no round; darn has no Merge-Review perch.
  The run's own records (status commits, review run directories, the settle record, carry-over) live in `_lyx` on the weft side, so no warp diff holds them;
- Merge-Review: the merge-in it reviewed.

A settle record is live when it names the run directory's current generation and no round report numbered past its settled round is on disk.
For this test alone, a directory with a settle record and no generation record, as a generation an older binary seeded and this binary settled leaves it, is at generation 1.
At Call entry, at any round including 0, the Bouncer reads the settle record after the skip seam, and compares only a live one:

1. A live record whose identity equals the live subject: nothing changed since the verdict, or the settle was interrupted (a failed commit, a crash).
   The Bouncer runs carry-over, Approve and Commit again, marks the record settled and returns Done; no seed, judge or review spawns, and no Stuck is returned, so no budget is spent.
   A failing step returns an error, as today.
2. A live record whose identity differs: a new generation, handled by D6, an amendment for every segment but Merge-Review.
3. No settle record, a generation record, and a settled verdict on disk: a verdict the judge wrote while no Call was attached to harvest it, as a driver stopped or crashed during the judge leaves it.
   The Bouncer writes the subject's identity and base into the generation record at every judge spawn, so it can tell whether the subject changed after the verdict.
   When the live subject equals that identity, the Bouncer settles the verdict through its harvest path, which writes the settle record before carry-over, Approve and Commit.
   When it differs, as a goto onto the writer admitted with no live settle record leaves it, the Bouncer handles it as case 2, a new generation by D6, whose base is the one the judge spawn recorded.
4. No settle record, no generation record, and a settled verdict on disk: a generation an older binary settled; today's clear and full re-seed.

A record that is not live, because a later generation began after it (an amendment, or Merge-Review's full generation, D6) or a round N+1 report followed it (loom's verify retry, the Webster reroute), takes today's round-by-round path with no comparison, and that generation's next settle writes a new record.
On that path a settled verdict at the current round is settled, never cleared: the clear is case 4's alone.

Every archive of a review run directory keeps the settle record, its stored patch and the generation record in the new directory: the Bouncer's own generation archive and PR-Rework's archive of the review run directories.
The CarryOver, Approve and Commit seams must be idempotent: a re-run over an approved, committed subject succeeds and commits nothing.

Rationale: the operator decided on 2026-10-19 that once an artifact is approved, no further review runs on it, and a failed mechanical step after a converged verdict re-runs only that step.

### D6. An amendment generation runs under a small allowance, scoped to the change

When D5 finds the subject changed, the Bouncer archives the generation as today and, in every segment but Merge-Review, starts an amendment generation, recording its number, its amendment mark and its base in the generation record.
Its soft budget is the allowance of D7, clamped to the perch's soft cap, and its hard cap is one round past that allowance, clamped to the perch's hard cap.
An allowance of 0 settles the amendment approved with no review round, through `approveWithoutReview`, and writes its settle record.
The Stuck a successful seed of the amendment returns carries a `generation_start` mark, which Shed records on its history entry; a seed succeeds once per generation, so neither a re-bounce Stuck nor a degraded seed carries it, and a Call that failed between the archive and the seed leaves the next Call's seed to carry it; `episodeStuckCount` counts that entry and stops after it, so the seed consumes one unit as any generation's seed does.
A Stuck carrying `generation_start` skips Shed's budget comparison, as a `BudgetExempt` Stuck does, while it is still recorded as counted;
so it never takes `budget_capped`, and a seed over a spent episode, such as one an interrupted settle at the perch's last round leaves, starts the generation instead of halting the run.
Shed's count and the `Bounces` seam read 1 once the seed is recorded, whatever the previous generation spent.
The amendment's seed, reviewers and judge are told the base the settle record holds, or after D5 case 3 the base the judge spawn recorded, so they review the task's change since the reviewed subject, and, when the re-entry follows a PR-Gate reject, the reject's review file.
A document segment's base is the weft commit that holds the settled artifact: a told seam reads the weft HEAD after the Commit seam returns, since that seam returns only an error, and the Bouncer writes it into the settle record when it marks the record settled.
An interrupted settle has no such commit, so its base stays the weft HEAD the record was written with, and an amendment after it reviews, under the allowance, the artifact's change since that commit: the whole uncommitted artifact when no earlier commit holds it.
A code segment's base is the settle record's merge base and stored patch.
Its reviewers are told the task's change since the settle as the settled side, the stored working-tree diff the settle hashed, compared against the current side, the working-tree diff against the current merge base as the identity reads it, untracked files outside the exclusion set included;
so an uncommitted change at the settle is part of the settled side and never shown as new, and the parent's changes a Merge-In brought are left out.
Every route back into the segment reaches this path through the Bouncer, PR-Gate's reject routes included: loom's PR-Rework into Plan-Bouncer and darn's Darn-Write into Darn-Bouncer.
PR-Rework's exempt-generation skip (`PlanReviewSkippable`) runs ahead of it, skips the review entirely and, through `approveWithoutReview`, writes the settle record the next re-entry compares against.
Merge-Review takes no amendment: a changed merge-in is a new subject, so case 2 records the next generation number in the generation record, with no amendment mark and no base, before its seed, and starts a full generation under the perch's own budget, its seed Stuck marked `generation_start` as well, and the allowance never applies to it.
The settle record kept across the archive then names an older generation and is no longer live, so the new generation's later Calls take the round-by-round path.

Bound: an amendment of any size gets the allowance, a plan the writer rewrote whole after a goto to Plan-Write included.
The judge and the soft-cap rule work as in any generation: with allowance A, the soft-cap rule runs at round A, where it auto-accepts, auto-continues one round or escalates, and a CONTINUE at round A+1, the hard cap, escalates with accept alone, as at any perch's hard cap.
Where the `Bounces` seam is unwired or errors, Shed's generic block applies the perch's own budget; loom and darn always wire the seam.

### D7. One run-wide amendment allowance key

`reviewbudget` declares `review_amend_max_bounces`, read from `loom.yaml` and `darn.yaml` through the same config stack as the other review budget keys, a task's `run_settings` included.
Its template default is 1, and 0 is the off switch D6 describes.
It joins `reviewbudget.CheckLocal`'s bounds with a minimum of 0 and no null, `reviewbudget.CheckResolved` refuses its resolved value below 0 with a way forward routed to the key, and it is not a guard key, under reviewbudget's rule that a budget of 0 is the off switch.
At 0, every amendment generation settles with no review, a PR-Gate reject's rework and a rewrite after a goto to the writer included.
There is no per-perch amendment key.

### D8. No friction session per stop

A `blocked` or `failed` halt still writes its halt note (`loom-halt`) through `friction.NotePath`, but spawns no reflection: `reflectHalt` and its callers under step, the loop and arm write the note and report the friction status `skipped`.
The run's Friction-Reflect row reflects every note in the friction directory, as it does today.
So no stop holds the driver lock for a friction session, and `lyx loom resume` never waits on one.
The `friction_timeout_min` key stays, bounding the Friction-Reflect row's reflection and lock wait in `internal/loomcli/run.go`.
The `awaitRunLockHalted` arm in `internal/loomcli/bootstrap.go` stays too, since it also fires on the first poll of every resume of a halted run; only its comments that name a reflection after a halt are rewritten.

Rationale: the operator's view on 2026-10-19 that friction need not run at every stop.

### D9. A review round with the wrong fork count is retried, and its failure says why the chair ended

In the Burler review row's attempt loop, a review that fails with `ErrClusterForksMissing` on attempt 1 is retried: its outputs, the chair's review file included, are archived and a fresh attempt 2 runs, as after a died or timed-out attempt 1.
The sentinel covers fewer forks than requested, more, and no fork audit at all, and each is retried alike.
A resumed reviewer whose review fails with the sentinel falls through to a fresh attempt 1, as a died or timed-out resumed reviewer does.
The mismatch arrives as an error from the review run, so this is a new branch on the error path.
A mismatch on attempt 2 is a hard failure as today, and so is any other fork violation on either attempt.
The failure's error, and so the halt note, names the requested and spawned fork counts, the chair's strand, and the session states shuttle recorded for the chair (for example an interrupt before `done`); the review result carries those states to burler.

Rationale: the chair's `done` with no forks in issue #756 was a stalled round, not a design fault, and the halt gave no cause.

## Constraints

- `PATTERN-tolerant-records`: `pause_reason`, the pause label, `budget_capped`, `generation_start`, the settle record and the generation record decode from files that lack them; every write, the settle record's stored patch included, goes through `internal/state`.
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

- An in-flight run deployed onto this binary has no settle records, so its next re-entry into a settled segment re-reviews in full (D5 case 4); a run directory an older binary seeded has no generation record, so a verdict its judge writes unharvested is cleared as case 4 too.
- A budget-exhausted halt an older binary wrote has no `budget_capped` entry, so under D1 neither its resume nor a goto grants that row a fresh budget.
- Halt notes are reflected only at the run's end; `run-process-ownership`'s planned `lyx batten abandon`, which removes a run's records, must reflect or keep a run's unreflected notes to stay within `PATTERN-friction-capture`.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d3-goto-after-halt-grants-budget (class design, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-3-review.md; fixer report: _lyx/reviews/plan/round-3-fixer-report.md.
  - new-keys-break-exact-set-pins-and-docs (class design, severity BLOCKING, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- A goto into a segment with Stucks in its current episode leaves the count where it stood, for Shed's block and for the Bouncer's `Bounces` seam alike; a reroute still ends its target segment's episode.
- A goto onto a review or fix row of a settled segment is refused with the file unchanged and a way forward naming the Bouncer and the writer row; a goto onto the Bouncer itself is admitted, and so is loom's verify retry onto the row its Publish failure record names.
- A goto onto the review row of an interrupted settle is refused; a goto onto a review row inside an amendment generation that has not settled is admitted.
- After the verify retry's generation settles again, a goto onto the failure record's row is refused.
- A tackler or arbiter verdict naming a row D2 refuses is rejected at validation, and an arbiter or tackler re-launch refused by D2 hands back with the refusal text.
- A budget-exhausted halt names the recipe's resume verb, not a goto; its resume, with no goto directly after the halt, grants exactly one more budget to that row, through the `Bounces` seam too; the Go driver hands that halt back without spawning the tackler or the arbiter, also after a tackler resume returned to it, also for a stop read from the status, and also when the on-disk build changed, never handing it over.
- A goto onto a budget-exhausted halted row, or onto a row of its segment, followed by a run leaves that row's count at the spent budget, so its next counted Stuck blocks again, while a resume of the same halt reads 0, and a goto after the row ran on that resume's grant keeps the grant's spent count, not the count before the halt.
- Every path that writes `paused` writes a non-empty `pause_reason`: a pause verb's bare request with its `--by` label, a fired condition with the label stored with it, and loom's bare pause still refused; a later Shed write that moves the run out of `paused` clears it, a write that leaves the state unchanged keeps it, and the pause verb's own writes carry it through.
- A driver re-launch onto a target after a failed review row leaves a non-empty `pause_reason`, and a paused status with an empty one is labelled in the drive report as having no recorded reason.
- A Bouncer whose CONVERGED settle failed at Commit, resumed with an unchanged subject, re-runs only the settle and returns Done; no seed or judge spawns and the budget count is unchanged.
- A Discussion-Bouncer whose CONVERGED settle with open findings failed at Commit re-settles on resume with no spawn, and a later segment's carry-over or a `lyx loom decision add` leaves the Discussion identity unchanged.
- A status commit and a carry-over commit between a code segment's settle and its re-entry leave the code identity unchanged.
- A Bouncer whose CONVERGED settle failed at Commit, followed by a goto to the writer row that changed the subject, runs an amendment generation, never an unreviewed approve, and the amendment's count reads 1 once its seed is recorded and it gets its allowance.
- A Bouncer re-entered with an unchanged subject returns Done with no spawn, also after a skip-seam or zero-allowance approval and after a Merge-In whose parent changes touch no line the task changed.
- A Bouncer re-entered after the subject changed runs an amendment generation whose seed is told the settle record's base, whose soft cap is `review_amend_max_bounces` clamped to the perch's soft cap, and whose second Call, after its round producer wrote round 1, judges round 1 without re-archiving.
- With allowance A, an amendment's round A meets the soft-cap rule as any generation's round at its soft cap does, and a CONTINUE at round A+1 escalates with accept alone.
- An amendment whose Call failed between the archive and the seed marks the next Call's successful seed Stuck `generation_start`, and a re-bounce Stuck is never marked.
- A settle interrupted at the perch's last round, with the count equal to the budget, followed by a changed subject, runs the amendment's seed and does not block.
- An untracked file outside the exclusion set changes the code identity, and one inside it does not.
- A Plan-Bouncer amendment after a PR-Gate reject is told a base that contains the settled plan.
- Loom's verify retry's round N+1 and the Webster reroute's round N+1 are judged as any round.
- With `review_amend_max_bounces: 0`, an amendment settles approved with no review spawn and writes a settle record.
- A new merge-in after a settled Merge-Review runs a full generation under the perch's own budget, also with `review_amend_max_bounces: 0`, and its second Call, after its round producer wrote round 1, judges round 1 without a second archive.
- After a Publish failure record names `Webster-Burler-Review`, and before the verify retry runs, a goto onto the Bouncer re-settles with the stored settle time, so a goto onto the failure record's row is still admitted.
- A code segment's amendment reviewers are told the task's change since the settle without the parent's changes a Merge-In brought.
- `review_amend_max_bounces` below 0, or null, in `run_settings` is refused by `reviewbudget.CheckLocal`, and a resolved value below 0 from `loom.yaml` or `darn.yaml` by `reviewbudget.CheckResolved`.
- After a PR-Gate reject, the re-entered review segment (loom's Plan-Review after PR-Rework's archive, darn's Darn-Review via Darn-Write) runs an amendment generation whose seed names the reject's review file, unless PR-Rework's exempt skip approves the generation without review.
- A settled run directory with neither a settle record nor a generation record is cleared and re-seeded as today.
- A CONVERGED verdict the judge wrote while no Call was attached, in a run directory with a generation record and no settle record, is settled on the next Call through the harvest path, with a settle record written and no clear, seed or review spawn, when the subject equals the identity its judge spawn recorded, and starts an amendment against that identity when the subject changed after the verdict.
- A `blocked` and a `failed` halt write a `loom-halt` note and start no reflection; the Friction-Reflect row reflects the note at the run's end.
- A Burler review round whose attempt 1 spawns a fork count other than the one requested, or leaves no fork audit, runs attempt 2; a mismatch on both fails with an error naming the counts, the chair's strand and its session states.
- No refusal, halt reason, help text or stencil names a goto as the way to a fresh budget; the goto help texts say that a goto keeps the segment's verdicts and spent budget, what it resets, and that it records itself as the pause reason.

## PATTERN changes

- `PATTERN-friction-capture` — changes: Halting a loom run or refusing in webster: Go writes a friction note, reflected by the run's Friction-Reflect row rather than at the stop, and archived only once a reflection covers it.

## Notes for the plan writer

- The `settledGeneration` branch of `Bouncer.Call` is where D5 and D6 attach; its Warn log already names the commit-seam-failure case D5 fixes.
- In `episodeStuckCount`, D1 removes the `OutcomeGoto` call of `gotoEndsEpisode` and keeps the reroute call; `rerouteCount`'s stop at a `reroute_capped` entry is the model for D3's `budget_capped` and D6's `generation_start`, except that `generation_start` is counted before the stop and `budget_capped` stops the count only when the entry right after it is not an `OutcomeGoto` entry.
- D6's budget-comparison skip for a `generation_start` Stuck attaches at the `!output.BudgetExempt && episodeStuckCount(...) >= effectiveMaxBounces(...)` case of `stepLocked` in `internal/shedengine/run.go`.
