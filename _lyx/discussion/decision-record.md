# Decision record — merge-review

## Goal

When a run merges its parent branch in, a cold-start agent reviews how the task's code interacts with what the parent gained while the task ran, and no other run can move that parent until the run lands or stops making progress.
The parent lock follows its owner process, never wall-clock time, so a host that sleeps keeps it and a crashed owner frees it at once.
A run that lost the lock, or whose parent moved outside lyx, merges the parent in again by itself before it can land.
Everything works for any parent branch, never a fixed one.

## Scope

In:

- The `Merge-In` row (D1) and the `Merge-Review` perch with its skip seam and off switch (D2, D3, D6).
- The parent lock: record, owner, release paths, verbs and visibility (D4, D5, D7, D8, D11, D12).
- The moved-parent reroute and batten's resume of a stale PR-Gate child (D9, D10).
- The darn recipe gains `Merge-In`, the lock and the reroute, without the merge perch (D1).
- The orch stencil's PR-Gate section and its stale lines (D13).
- Docs in the same commit: landingshed, battenshed, shedengine and shedverbs `doc.go`, the new lock package's `doc.go`, `docs/overview.md`'s module table, `PATTERN.md` and its touched background files (D14), the loom status spec and the refusal spec rows.

Out:

- `lyx pause` itself (note `lyx-halt`): this task provides the release-all function and verb it calls (D11).
- Detecting a live owner that hangs (task `driver-arbiter`).
- A lock verb for a hand landing outside a run: a hand landing goes through the run's own rows (`lyx loom goto --to Finalize`), and a raw push to the parent is caught as a moved parent (D9).
- A parent lock at batten's Land row: Finalize has already landed and released the lock before Land, and Land only pulls the prime, deploys and prunes under its existing land lock.
- Glyph-level overlap: the seam judges overlap by file and Go package, a superset of the glyphs the board names (D6).
- A `lyx cleanup` hook for lock records: a stale record is replaced by the next acquire on its branch and shown as stale by `lyx batten lock list`.

## Decisions

### D1: Merge-In is its own landing row, ahead of Publish, for every parent

Decision: Publish's merge-in of the parent, its clearing of a parked merge-in and of live conflict-session strands, its clean-tree checks, its conflict session and its post-merge plan verify move into a new `landingshed` producer, `Merge-In`, which runs for every parent branch, including one outside `require_pr_to_base`.
Publish keeps `publish_verify`, the push and the pull request, and stays a no-op for a parent outside `require_pr_to_base`.
Loom's order becomes `Webster-Bouncer → Describe → Merge-In → Merge-Bouncer → Publish → PR-Gate → Finalize`; darn's becomes `… → Merge-In → Publish → PR-Gate → Finalize`.
Once it holds the parent lock (D5), and before it merges, Merge-In fetches the parent's upstream and fast-forwards the local parent branch to the upstream tip, so a hand push to the upstream reaches the local parent at the next Merge-In and needs no other actor; it then merges the local parent branch, which now equals the upstream tip, and records that tip.
The local parent branch is checked out in the parent pair's own worktree, the one Finalize opens, so the fast-forward pulls that pair through `fabricengine`'s pull; running it under the lock keeps it from racing another run's Finalize, which merges into the same worktree under the lock.
A parent with no upstream is merged and recorded as its local tip.
Under the told push skip (`Deps.PushSkipped`, from `FABRIC_SKIP_PUSH`), Finalize never pushes the parent, so Merge-In skips the fetch and the fast-forward and merges and records the local parent tip, as for a parent with no upstream.
A local parent ahead of its upstream, or diverged from it, is Stuck at Merge-In with a way forward naming the push or reconcile of the parent branch in its own pair, then `lyx loom resume`; Merge-In never pushes or rewrites the parent.
The fast-forward is refused, Stuck with a way forward, when the parent pair's worktree is dirty (commit or stash the changes in the parent pair), when a merge is in progress there (finish or abort that merge in the parent pair), or when no live parent pair exists (re-create the parent pair); each names the step in the parent pair and then `lyx loom resume`.
Bound: the fast-forward only moves the parent pair to its upstream tip, only when that move is a fast-forward, and never touches local commits; the ahead and diverged states are Stuck before the pull runs.
Merge-In writes a Go-owned merge record in the run's ephemeral directory: the parent tip it merged, the parent tip the last finished merge review reached (the fork point before any), the task head before the merge, and the overlap D6 computes.
The Publish verify failure record and batten's Publish-retry name the row that ran the failing verify: `Merge-In` for the plan verify, `Publish` for `publish_verify`.
A run parked on a row whose name or edges changed is restarted, not migrated, as the recipe headers already state for earlier row changes.

Rationale: the merge review's fixes must be verified and pushed before PR-Gate, so the push has to follow the review; one merge row serves both recipes and both kinds of parent.

### D2: Merge-Review perch reuses the Bouncer, BurlerReview and BurlerFix rows

Decision: loom's recipe gains segment `Merge-Review`: `Merge-Bouncer` (on_stuck `Merge-Burler-Review`, on_done `Publish`), `Merge-Burler-Review` and `Merge-Burler-Fix`, in the same ring shape as Webster-Review, with a segment colour of its own.
The review target is the merge record's overlap: the merged-in parent commits and the overlapping paths, read through git as Webster-Review reads its diff.
Its fasit is `PATTERN.md`, the `pattern/` files the overlap touches, and the task's decision record.
A new rubric stencil, `loom-rubric-merge-review`, asks the reviewer to judge only the interaction: invariants the parent's commits added or changed against the task's code, and the task's code against the parent's new code.
The fix row runs `fix-scope: source` with the verify gate, as Webster-Burler-Fix does; no cluster fan.
loom.yaml gains `merge_max_bounces: 1`, `merge_auto_accept: true` and `skip_merge_review: false`; the first two are read like the other per-perch keys, and `skip_merge_review: true` rewrites Merge-In's on_done to Publish, as darn's `review_max_bounces: 0` rewrites past its perch.
A pair whose `loom.yaml` lacks the new keys reads their zero values until `lyx config reconcile` adds them: the review runs, with `review_max_bounces` as its budget and no auto-accept, the stricter side.

Rationale: the perch machinery, budgets and escalation already exist; a cold reviewer on a narrow subject is the cheapest review the gap needs, and the off switch's zero value keeps the review on.
Bound: `skip_merge_review` skips only the review; Merge-In, the lock and the reroute still run.

### D3: Merge-Bouncer's auto-accept escalates over an open BLOCKING finding

Decision: a Bouncer row config key, set only on `Merge-Bouncer`, makes an auto-accept on a spent budget escalate with cause `budget` instead when the round's ledger still holds an open BLOCKING finding.
Every other Bouncer row and every other auto-accept keeps today's behaviour.
Non-BLOCKING open findings are accepted and carried into `## Open risks` under `Merge-Review`, as any auto-accept does.
The escalation is an awaiting stop at a Bouncer row the auto-accept did not settle, which `driver-arbiter` makes eligible for its arbiter.

Rationale: the board asks for one review and one fix then on, yet a finding the fix cannot settle escalates; BLOCKING is the line between the two.
Bound: the key only turns some auto-accepts into escalations; it never accepts what today escalates.

### D4: The parent lock is a record file per parent branch at hub level

Decision: a new told-geometry engine package (working name `parentlock`) owns the lock.
One record per parent branch lives under the board dir's ephemeral tree (`<BoardDir>/.lyx/…`, built through `lyxdirs` and told to the engine), visible to every worktree on the hub; the branch name is encoded into a filename so `/` and other path characters cannot escape the directory, and the first acquire creates the directory.
The record names the owner run (slug, pair path, run id), the owner process's pid and start time, the parent branch and the acquire time.
A read-modify-write runs under a short OS file lock beside the record, so two acquirers cannot both win.
A record whose owner process is dead, or whose pid now names a process with another start time, is stale and is replaced by the next acquirer.
An acquire by the run the record already names is granted and rewrites the owner process, so a new loop of the same run takes the lock back from a dead loop or from batten (D8).
Waiters queue: each waiter writes a ticket with its pid and start time, and the lock goes to the oldest live ticket, so no waiter starves; a dead waiter's ticket is skipped and removed.
An unreadable or undecodable record is Stuck at the acquiring row with a way forward naming `lyx batten lock release <branch>` (D11).

Rationale: an OS file lock dies with the short-lived step process that takes it, and the lock must outlive single steps; a pid-and-start-time record, as `loop.pid` keeps, is valid exactly while its owner lives, whatever the host's clock does during sleep.

### D5: The owner is the run's detached shed loop

Decision: `Merge-In` acquires the lock in the name of the run's detached `--until-stop` loop process, read from the loop's pid record; a step run outside a loop names its own step process, so its lock lapses when that step exits.
`internal/shedverbs` gains a told exit hook on its loop spec, filled by the arming modules (loom and darn), and keeps its import set; the hook releases the run's lock when the loop exits, which it does at every stop: awaiting, blocked, failed, paused, a stop condition and the terminal.
The release is a compare-and-release on the loop's own pid and start time, as D8's is: a record that names the run but is owned by another process, batten or a newer loop, is left alone.
The one exception is an awaiting stop at PR-Gate with a live batten watching the run: the hook hands the record to that batten's process instead (D8), so the lock is kept while the orch reviews.
Finalize releases the lock on its Done, before Friction-Reflect runs.
A lock-holding row whose verdict takes an edge out of the window from Merge-In to Finalize releases the run's lock on that edge: today that is PR-Gate's rejection Stuck, to PR-Rework in loom and to Darn-Write in darn.
The rework comes back around through Merge-In, which reacquires.
Merge-In, and each row of D9 that finds the lock not held, waits for it with no time limit in the queue of D4, polling from one second doubling to thirty; the wait counts as activity for the loop's idle watchdog.
A pause request or cancellation seen during the wait halts the row blocked, as the land-lock wait does, with the way forward `lyx loom goto --to <row>` then `lyx loom resume`.

Rationale: the loop lives exactly while the run makes progress on its own, so validity by its pid gives release-on-stop and keeps sleep from mattering; the orch's PR-Gate review is part of the run's progress, not a human stop.

### D6: Merge-Bouncer's skip seam runs the round only on overlap

Decision: a new `skip_seam` value on `Merge-Bouncer` skips the judge, with no agent spawned, unless the parent commits since the last finished merge review overlap the task's own change.
The parent side is the commits from the parent tip the last finished merge review reached (the fork point before any) to the parent tip just merged.
The task side is the files changed between the parent tip last merged into the task (the fork point before any merge-in) and the task head before this merge-in, so earlier merged parent changes never count as the task's.
They overlap when a file is changed on both sides, when a Go package directory holds files changed on both sides, or when the parent side changes `PATTERN.md` or a file under `pattern/`.
An already-up-to-date merge-in skips.
Merge-In computes the overlap and writes it into the merge record; the seam reads only that record.
Merge-Bouncer's Done, a skip included, records the merged parent tip as the last finished review in the run's ephemeral directory; a review that ends without Merge-Bouncer's Done, such as an escalation followed by a goto past it, records nothing, so the next range still covers its commits.
A missing or unreadable merge record falls back to a real review whose subject is the whole merge-in diff.

Rationale: the reviewer's subject is the interaction, which is empty when the two sides touch nothing in common; a PATTERN change binds every package, so it always reviews.
Bound: the seam skips only a merge-in whose git-computed sides share no file, no Go package and no PATTERN change.

### D7: A waiting run past a bound releases the lock

Decision: through a second told loop-spec seam, filled by the arming module, the owning loop reads its run's live agents on its existing watch wakes: a held turn end through shuttle's wait markers, `asking` and `idle-stalled` through shuttle's session states.
It releases the lock when no live agent of the run is busy and at least one has been waiting for longer than `parent_lock_idle_release_min`.
An unreadable state counts as not waiting, so a failed read releases nothing.
The bound is measured on the loop process's monotonic clock, which does not advance during suspend on Linux, so a sleep never counts as waiting.
The key lives in the hub-wide `landing.yaml`, default 10, added to existing hubs through `hubreconcile.Ensure`; a value below 1, an absent key included, reads as 10 with one Warn.
The run keeps running; the next lock-taking row reacquires (D9).
A `lyx pause` that keeps loops alive frees locks through release-all (D11), never through this bound.

Rationale: a run stalled on a human or an error must not hold every other run on its parent.
Bound: release frees the lock only; it never stops or changes the run, and the run cannot land before D9's moved-parent check runs under a reacquired lock.

### D8: Batten holds a PR-Gate lock and backstops releases

Decision: batten's InnerRun wait, at its notice-probe cadence, acts on a lock record that names its child's run:
- a record handed to batten at a PR-Gate awaiting (D5) is released once the awaiting has lasted `parent_lock_idle_release_min` on batten's monotonic clock with no decision, or once the child leaves awaiting for any stop but a resume, or once a resumed child stops again without having taken the record back;
- a record whose owner is the child's loop is released when that same owner, by pid and start time, has met D7's rule; the release is a compare-and-release on the owner, so a fresh loop's record is never freed.
It logs one Warn per release.

Rationale: the orch's PR-Gate review is quick and must not see the parent move under it, and the board asks batten to backstop an owning loop that failed to release.
Bound: batten releases only records naming its own child's run, and only under the bound or a matched stalled owner.

### D9: Publish, PR-Gate and Finalize reroute to Merge-In on a moved parent

Decision: `shedengine` gains a recipe-declared reroute edge: a `ProducerDef` field (recipe key `on_reroute`) naming one row, and an output flag a producer sets on a Stuck verdict to take that edge instead of `on_stuck`.
A reroute writes a new history-only outcome, `reroute`, which every history reader accepts; it ends the bounce episode of its target's segment, spends no bounce budget, and is allowed across segments.
`PATTERN-shed-verb-set`'s goto clause is unchanged: `OutcomeGoto` stays the goto verb's alone.
A row's reroutes are capped at three, counted from the row's last Done or from the last `lyx loom resume` of that row's reroute-cap halt, whichever is later; the fourth is a blocked halt for a human whose reason names the moving parent and `lyx loom resume`, and that resume resets the row's count, so each resume grants three more reroutes.
`Publish`, `PR-Gate` and `Finalize` in loom and darn declare `on_reroute: Merge-In`.
Each first ensures the lock (D5) and then reads two code-side parent tips: the local parent branch tip, which Finalize's catch-up merge-in and parent-side merge use, and its upstream tip after a fetch, which a hand push reaches.
A parent with no upstream, or a hub under the told push skip (D1), reads the local tip alone.
The parent has moved when either tip differs from the tip the merge record says was merged and the parent does not already contain the task's work; a parent that contains it, after a pull request merged on GitHub or an earlier Finalize that landed, takes Finalize's already-landed path.
A local parent ahead of or diverged from its upstream at Publish, PR-Gate or Finalize therefore reroutes to Merge-In, where D1's Stuck applies.
Finalize's catch-up merge-in stays, after this check; with the local tip checked equal to the merged tip under the lock, it brings no parent code the merge review has not covered, and still syncs the records side; its parent-records commit keeps its place after it.
Finalize's retry after a parent-side merge that reports merge-in-required runs only when both code-side tips, read the same two-ref way, still equal the merge record's merged tip, which covers a parent moved by run records alone; otherwise Finalize reroutes to Merge-In as its first check does.

Rationale: losing the lock or a hand push to the parent leaves a merge nobody reviewed; sending the run back to its merge row is the one path for both.
Bound: a row can reroute only to the one target its recipe row names, only after reading a parent tip that differs from the recorded merge and does not hold the task's work, and at most three times before a human resumes it; a reroute redoes the merge and its review, never skips them.

### D10: Batten resumes a child awaiting at PR-Gate whose parent moved

Decision: batten's InnerRun wait, for a child awaiting at PR-Gate, reads the parent's upstream tip after a fetch at its notice-probe cadence, and when it differs from the child's merge record and does not hold the task's work, resumes the child as it resumes after a decision, so PR-Gate's re-call reroutes (D9).
No notice goes for that awaiting episode.

Rationale: the operator's `prgate-stale-base` idea: the orch should never review a pull request whose base moved.
Bound: batten only resumes; the reroute decision stays PR-Gate's under the lock, and the fetch runs only for a child awaiting at PR-Gate.

### D11: A lock verb group lists and force-releases locks

Decision: `lyx batten lock list` shows every parent lock with owner run, pid, liveness, age and queued waiters; `lyx batten lock release <parent-branch>` and `lyx batten lock release --all` free records.
They run from the prime, as every batten verb does, follow `PATTERN-cli-cobra`, and each refusal names its way forward.
The engine exposes the release-all function `lyx pause` (note `lyx-halt`) will call.

Rationale: the orch and the operator need a way to free a lock whose owner is alive but wedged, and `lyx-halt` needs one before a sleep.
Bound: a release frees the record only and never stops a run; the released run reacquires at its next lock-taking row and reroutes if the parent moved meanwhile.

### D12: A lock wait shows as waiting

Decision: while a row waits for the lock it writes a wait record, carrying its pid and start time, that `lyx loom status` renders as waiting for the parent lock on `<branch>` held by `<slug>`; batten counts it as a live wait, as it counts the verify marker, so no quiet notice fires during it.
A wait record whose writer is dead is ignored by both and removed by the next wait.
The row also marks the driver strand on screen through landingshed's wait-mark callback, the shape the post-merge verify uses.

Rationale: the orch must not read a run waiting its turn as stuck.

### D13: The PR-Gate fork narrows to goal and decisions

Decision: the orch stencil's PR-Gate section tells the reviewer fork to judge the run against its board goal and the operator's recorded decisions, and to leave code review to the Webster and Merge reviews; carried-over `Merge-Review` open risks are part of what it reads.
The stencil's lines on batten's Publish-verify retry and on Finalize's sync are updated to D1 and D9.
Checking the run against other runs in flight stays with note `alignment-agent`.

Rationale: with the lock held through the orch's review (D5, D8) and the merge reviewed, the remaining review is what needs the orch's context.

### D14: A new PATTERN entry for the parent lock

Decision: `PATTERN.md` gains `PATTERN-parent-lock`: merging a parent into a task branch or landing on a parent from a run happens only under that parent's lock through the lock package, and a moved parent reroutes to the merge row rather than merging new code at a later row.
`pattern/PATTERN-told-geometry.md` lists the lock package among its bound packages, and `PATTERN-wait-mark-display-only`'s caller set names the landing wait-mark callback for the lock wait beside the verify.

Rationale: three rows across two recipes, the loop and batten all take or release the lock, so one invariant names the rule they share.

### Added after Discussion (parent, 2026-10-12): Moved-parent check reads both parent refs

Decision: Accept the round-2 fixes as written: D9 reads the local parent tip and its upstream tip, D1 fast-forwards the parent pair only under the lock with Stuck exits for a dirty, mid-merge or missing parent pair, and a push-skipped hub merges and compares the local tip.

Rationale: The round-2 fixer applied F1, F2 and F3 as the review proposed and checked each premise against the code; the segment ran out of rounds, not out of agreement, and no open finding makes the design wrong.

## Constraints

- `PATTERN-told-geometry`: the lock engine, landingshed and battenshed take the lock directory told; only CLI wiring resolves it.
- `PATTERN-shed-verb-set` and `PATTERN-shed-producer-seam`: the loop's exit hook and waiting-state reading are told seams, and the reroute stays inside `shedengine`'s stdlib-only imports.
- `PATTERN-durable-vs-ephemeral-state` and `PATTERN-hub-containment`: lock, ticket, wait and merge records live under `.lyx`; the board dir is reached by path, never junctioned into a pair.
- `PATTERN-hub-wide-config` and `PATTERN-binary-change-reconcile`: `parent_lock_idle_release_min` is read only from the board dir and added through `hubreconcile.Ensure`.
- `PATTERN-fabric-git` and `PATTERN-gitexec-checked-call`: fetches, parent tips and the overlap's diffs go through `fabricengine` or `gitexec`.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: every new Stuck and refusal names a way forward the agents may run, with a refusal-spec row and a test.
- `PATTERN-daemon-wakeups`: the lock wait wakes at most once a second and backs off.
- `PATTERN-spawn-color`: the Merge-Review segment's sessions take their colour from the launch spec.

## Auto-mode assumptions

- Merge-In as its own row (D1), file-and-package overlap (D6) and the BLOCKING-only auto-accept exception (D3) are self-made picks; see the support log's question ledger.
- The idle bound's default of 10 minutes (D7) and the reroute cap of three (D9) are self-made values.
- Batten holding the lock through the orch's PR-Gate review (D5, D8) reads the board's "PR-Gate waiting on a human" as excluding the orch.
- Darn takes the lock, Merge-In and the reroute but no merge perch, since the board scopes the review to loom and an unlocked darn landing would move a locked parent.

## Open risks

- Go's monotonic clock excludes suspend on Linux and macOS; on native Windows it is not verified, and the operator's Windows use is through WSL, which runs the Linux kernel.
- A live but hung owner holds the lock until D7's bound, batten's backstop or a force release frees it; detecting the hang itself is `driver-arbiter`'s.
- `config-layers-run-settings` classifies which keys its local layer and run settings may override; `merge_max_bounces`, `merge_auto_accept`, `skip_merge_review` and `parent_lock_idle_release_min` need that classification in whichever task lands second.
- `driver-arbiter` names the Publish verify halt in its eligibility rules and edits batten's notice path; under D1 that halt is Merge-In's, and whichever task lands second updates the other's row name and notice handling.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - merge-in-and-moved-parent-check-read-different-refs (class design, severity BLOCKING, open in rounds 1, 2)
  - parent-fast-forward-moves-pair-worktree-unbounded (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Two runs on one parent: the second waits at Merge-In in queue order, shows the wait in `lyx loom status`, sends no quiet notice, and proceeds once the first's lock is released by Finalize's Done or by any release path.
- Two acquirers racing on a free lock: exactly one wins; three waiters are granted in ticket order, skipping a dead one.
- A lock whose owner process is dead, or whose pid names a process with another start time, is taken over at once by the next acquirer.
- A parent branch name containing `/` maps to a record inside the lock directory, and a first acquire creates the directory.
- A clock jump during a lock hold leaves the lock valid; the idle bound ignores time the injected monotonic clock did not see.
- Each stop kind (blocked, failed, paused, stop condition, terminal, awaiting outside PR-Gate) releases the run's lock; Finalize's Done releases it before Friction-Reflect.
- An awaiting at PR-Gate under a live batten hands the lock to batten, which keeps it until a decision resumes the child or the bound passes, and the resumed loop takes it back; a resumed child that stops before taking it back leaves it to batten, which releases it, and the stopped loop's exit hook leaves batten's record alone.
- A rejected PR-Gate releases the lock before PR-Rework (darn: Darn-Write) runs, and the reworked run reacquires at Merge-In.
- With no busy agent, a waiting agent past the bound releases the lock while the run keeps running; one below the bound, one beside a busy agent, and an unreadable state do not.
- Batten's backstop releases only a record whose owner matches the stalled loop, never one a fresh loop took after a resume.
- A moved parent at Publish, PR-Gate or Finalize reroutes to Merge-In without spending bounce budget, and the next merge review judges only the parent commits since the last finished review, with the task side excluding earlier merged parent changes.
- A review that ends without Merge-Bouncer's Done, such as an escalation followed by a goto past it, leaves its commits in the next review's range.
- The fourth reroute of a row since its last Done halts blocked with a way forward, and `lyx loom resume` of that halt lets the row reroute three more times.
- A parent that already holds the task's work, after a GitHub merge or an earlier landing, lands through Finalize's already-landed path with no reroute.
- A hand push to the parent's upstream is seen as a moved parent; the rerouted Merge-In fast-forwards the local parent, merges the pushed commits and records their tip, and the next check passes.
- A local parent ahead of or diverged from its upstream is Stuck at Merge-In with a way forward, never merged.
- A commit made on the local parent branch after Merge-In, with the upstream unchanged, is seen as a moved parent at Publish, PR-Gate and Finalize and reroutes to Merge-In, where it is Stuck as ahead of its upstream.
- Merge-In's fast-forward moves the parent pair only after the lock is held; a dirty parent pair, a parent pair with a merge in progress and a missing parent pair are each Stuck with a way forward, and a parent pair with local commits is never moved.
- Two successive landings on one parent with the push skipped both pass Merge-In, each merging and recording the local parent tip.
- Finalize's merge-in brings no parent code the merge review has not covered and still syncs the records side.
- A code-side parent commit that arrives between Finalize's check and its parent-side merge, whether a local commit on the parent branch or a push to its upstream, reroutes to Merge-In and never lands; a parent moved by run records alone takes Finalize's retry.
- A child awaiting at PR-Gate whose parent moved is resumed by batten and reroutes.
- A loop killed during Merge-In's conflict session resumes through Merge-In's clearing of the parked merge and its conflict strands.
- The skip seam skips with no agent when the sides share no file and no Go package and the parent side leaves PATTERN untouched; it runs the round on a shared file, a shared package or a PATTERN change; it skips an up-to-date merge; it reviews the whole merge-in diff when the merge record is missing.
- With budget 1 and auto-accept: a CONTINUE with only non-BLOCKING open findings settles accepted and carries them over; an open BLOCKING finding escalates.
- `skip_merge_review: true` routes Merge-In straight to Publish, and a `loom.yaml` without the new keys runs the review.
- A pause during a lock wait halts the row blocked with its way forward.
- A wait record left by a dead step is ignored by `lyx loom status` and batten.
- `lyx batten lock list` shows owner, liveness, age and waiters; `release <branch>` and `release --all` free records; an unreadable record makes the acquiring row Stuck naming the release verb.
- A no-PR parent holds the lock from Merge-In through Finalize with no reroute when nothing else moves the parent.
- Darn lands through Merge-In under the lock and reroutes on a moved parent.
- A recipe validation rejects an `on_reroute` naming a row not in the list.

## Notes for the plan writer

- Batten's Publish-retry (`internal/battenshed/publishretry.go`) is the precedent for batten acting on a parked child; its target rows change with D1.
- The rework-exempt `skip_seam` on Plan-Bouncer (`internal/shedrecipe/entries_bouncer.go`) is the precedent for D6's seam wiring.
- `internal/shedverbs/looppid.go` holds the loop pid record D5's owner identity comes from, and the loop's `Activity` seam is the precedent for D5's and D7's told seams.
- Batten's quiet-notice check reads file markers (the verify marker, shuttle's wait markers), the precedent for D12's wait record.
- Note `finalize-discrepancy-doc` describes a conflict shape at Finalize that under D9 arises only at Merge-In; note `raddle`'s fold into Finalize's parent-side merge is unaffected.
