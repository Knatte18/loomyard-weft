# Decision record — orch-wake-policy

## Goal

Batten runs the landing routine itself after a child run ends done, and handles the halts whose way forward is mechanical, so the hub orch is woken only for parent review, review escalations a perch does not settle, design calls, PR-Gate, a land or teardown failure, halts whose cause or way forward is unknown and, until `orch-split`'s run driver starts ready tasks, a landing that made a task ready.
The orch's landing routine (pull, deploy, config reconcile, intake, prune, the teardown check, the fabric commit) leaves its per-landing calls, and its `lyx loom start` retry loop after an approval goes away.

## Scope

In:

- A new batten row that lands the run in the prime after Run-Shed returns Done: pull, deploy, prune, and a notice for tasks the landing made ready.
- The `done` state-changed notice goes away; batten notifies the orch only when a land or teardown step fails, or when the landing left a board task newly ready.
- An automatic goto and resume for the first Publish verify failure of a batten run; the second wakes the orch.
- Fabric's bounded push retry covers a remote "cannot lock ref" refusal, and a bare "(failed)" refusal beyond Add's two branch pushes, on the plain, never-forced pushes (D6).
- The orch role stencil, `CLAUDE.md` and the refusal spec move each step to its new owner, and the orch's `lyx loom start` retry loop after an approval is dropped.

Out:

- Starting a newly ready task: note `orch-split` owns the mechanical run driver; this task only tells the orch which tasks a landing made ready (D4).
- The per-worktree `loom.yaml` landing warning and the hand-set `review_max_bounces`: task `config-layers-run-settings` removes both.
- Per-perch auto-accept of a spent review budget: task `burler-split-budget`.
- Any other halt class: besides the push refusals D6's retry covers, only a Publish verify failure is automated.
- An alignment check at parent review or PR-Gate: note `alignment-agent`.

## Decisions

### D1. A Land row between Run-Shed and Worktree-Teardown

The batten recipe gains a row (working name `Land`): `Run-Shed on_done: Land`, `Land on_done: Worktree-Teardown`, `Land` with no `on_stuck`, so a failed land step halts batten blocked at `Land` with the pair still present.
Worktree-Teardown stays the only terminal and stays unreachable from any failure path.
An in-flight batten status seeded before this change routes into `Land` when its Run-Shed returns Done.

Serialization: the row holds a hub-level land lock for its whole body, so two landings never pull, deploy or prune at once.
The land lock is waited on without a time limit, polling at most once a second and backing off (PATTERN-daemon-wakeups), and the wait honours `lyx batten pause` and cancellation.
The prime lock is taken, with the existing bounded wait (`primelockwait.go`), only around the pull itself, so a deploy never holds it and never starves Worktree-Create or Worktree-Teardown.

Steps, in order, each idempotent so a resume re-runs the row from the top:

1. Prime check: skip pull and deploy when the child's recorded parent branch is not the prime's checked-out branch; a stacked task that landed into another task branch deploys nothing.
   When the pull will run, a prime with uncommitted tracked changes is waited on, re-checked each poll, for at most ten minutes, then Stuck naming the paths.
   A landing that skips pull and deploy never waits on the prime's tree.
2. Pull the prime through fabric's pull.
   A diverged prime, or any pull refusal, is Stuck with fabric's reason.
   Batten only pulls the prime's code branch and never pushes it (PATTERN-push-both-sides).
3. Deploy: run landing config's `deploy` command (D2) in the prime, when it is non-empty and the child's parent branch is in landing config's `require_pr_to_base`.
   A landing into a prime branch outside that list pulls but skips the deploy, logging the reason at Info.
4. Ready snapshot: read the set of ready tasks (kind `task`, unclaimed, stored `depends_on` empty) and persist it in a marker under the row's scratch directory, unless a marker for this run already holds one.
5. Prune the board, the same operation as `lyx board prune --apply`.
6. Read the ready set again; the tasks in it and not in the persisted snapshot are this landing's newly ready tasks (D4).

Persisting the snapshot before the prune keeps the newly ready set correct across a restart after the prune, and when an earlier landing's prune already removed this run's done entry, the dependents it freed appear in that earlier landing's set instead, so none is lost.

Every outside effect goes through a seam on the battenshed deps, wired in `internal/battencli`: the pull, the deploy run, the board reads and prune, the lock acquisitions.
battenshed's import allowlist (`seam_enforcement_test.go`) does not grow.

Rationale: the routine is mechanical, and running it before teardown keeps the pair available when a step fails.
Bound: the row runs only after Run-Shed's Done, which batten returns only for a child in state done; it never runs for a halted, failed or awaiting child.

### D2. The deploy command is a hub-wide landing key

`landing.yaml` gains `deploy`: a shell command batten's Land row runs in the prime after a landing into the prime's branch.
Empty or absent runs nothing and logs nothing; the template default is empty, so lyx stays repo-generic.
The hub sets it to `./update-plugins.sh`.
The key is hub-wide like the rest of `landing.yaml`: read from the board dir, never from a pair, and `lyx config` refuses a write to it from a pair, a guard against a mistaken write rather than a determined agent.
It is the same trust class as `publish_verify`.
The command executes code from the landed tip (`./update-plugins.sh` runs `go run ./tools/deploy`), which a task can edit, and it may push outward: the hub's deploy moves `prod`.
It is trusted because a branch in `require_pr_to_base` receives code on its remote only through PR-Gate-reviewed landings or the operator; that review, not the config key, is what keeps task-authored code from reaching the prime unreviewed.
PR-Gate reviews only a parent branch in `require_pr_to_base` and passes any other with no review, so the deploy runs only for a parent in that list when the Land row runs (D1 step 3).
The row does not re-check what PR-Gate did: an edit to the list between PR-Gate and Land, made by the orch in its own hub config, can deploy a tip PR-Gate passed unreviewed.
The command runs with a timeout (Plan picks the value), its output to a log file under the row's scratch directory, its spawn and wait logged through `internal/logger`.
A non-zero exit, a timeout or a shell that cannot start is Stuck naming the exit, the log path and the way forward.
Rationale: `update-plugins.sh` exists only in this repository, and a hub-wide key keeps the command line itself out of a task's config.
Bound: it runs only from the Land row, only in the prime, only for a parent branch in `require_pr_to_base` when the row runs, only after a pull that left the prime at that branch's remote head, and only under the land lock; what it can do is whatever that head's code does.

### D3. Config reconcile is left to `hubreconcile.Ensure`

The Land row runs no `lyx config reconcile`.
The deployed binary reconciles every worktree at its first start verb through `hubreconcile.Ensure`, which is already the only sanctioned reconcile after a binary change.
The batten process running the Land row is the old binary, so a reconcile from it would compare against the old templates.

### D4. Notices from the Land and Worktree-Teardown rows

Run-Shed's `done` state-changed notice is removed.
The Land and Worktree-Teardown rows get the same Notify seam the InnerRun row has, with its orch-strand lookup and delivery retries, and queue one notice in exactly these cases:

- A Land step or Worktree-Teardown returns Stuck or a hard error: the notice names the row, the step, the error cut to one line and the way forward (`lyx batten run <slug>` from the prime once the cause is fixed).
  One notice per failure episode, recorded in a marker under the row's scratch directory.
  An episode is one entry of the row from its predecessor's Done or from a halt at that row: a deliberate re-run after a halt (`lyx batten run <slug>`) that fails again starts a new episode and sends a new notice.
  A batten that died while the row was still running, before it halted, continues the same episode on restart, so it never repeats a notice already queued.
- The landing made at least one task newly ready (D1 step 6): the notice names the landed slug and the ready slugs, once per run, recorded in the same marker.

A landing that succeeds and readies nothing sends no notice.
A notice that delivery drops after its retries leaves a failed row halted, which `lyx batten status` still shows; a dropped ready notice is logged as a Warn.
The ready notice is a deliberate, temporary departure from the board entry's list of what wakes the orch: until `orch-split`'s run driver starts ready tasks, the orch starts them, and `orch-split` replaces this notice rather than adding to it.
Bound: a notice is informational and never changes a row's outcome.

### D5. One automatic retry of a Publish verify failure

When the child halts blocked with Publish as its current producer and a Publish failure record that belongs to this halt, batten sends the child to the record's target producer and resumes it, and sends no notice for the halt or for the child's paused state at the target while the retry is unfinished, that is while its marker (below) holds `started` or `goto-done`.

- Publish writes the record's write time into it (`written_at`), as it writes the target field, in the RFC 3339 whole-second UTC form shed's history `at` uses, so both compare at one precision.
  The record belongs to this halt when its HEAD equals the task worktree's HEAD and its write time falls inside the Publish attempt that halted: strictly later than the child's last history entry before that attempt and at or before the halting Publish entry.
  A record from a verify that failed within the same second as that earlier entry is therefore not retried, which fails safe to the blocked notice.
  The blocked state's start is never the reference, since Publish writes the record before it returns Stuck.
  Any other record, including one left at the same HEAD by an earlier Publish halt, is stale and the halt is handled as today.
- The target producer is a structured field the recipe wiring writes into the Publish failure record (today `Webster-Burler` for loom and `Darn-Write` for darn, the producers `internal/loomcli`'s `failedWayForward` already names), so batten names no recipe's rows.
- The goto runs the child's goto verb (the `internal/shedverbs` body); the resume goes through the InnerRun Spawn seam the awaiting arm uses, so the child's own bootstrap resumes the run.
- A goto refused while the child's driver holds the run lock, and a resume refused because the driver has not parked or its pane is busy, are retried at most once per notice probe, the way the awaiting arm retries an approval resume.
  A busy pane gets the awaiting arm's driver-busy notice.
  The retrying is capped in time (Plan picks the value), measured from the marker's `started` write so a restart does not reset it: past the cap, batten stops retrying, writes `capped` to the marker and sends the notice for the child's actual state, blocked at Publish or paused at the target, with the last refusal, so a driver that never releases its lock or never parks still wakes the orch.
- A goto refused for any other reason, such as a target the recipe lacks, is not retried: batten sends the blocked notice with the refusal.
- A marker under batten's scratch directory records the retry's progress, written before the goto (`started`), after a goto that succeeded (`goto-done`), after a resume that landed (`resumed`) and when the cap's notice is queued (`capped`), and is reset only when batten seeds a fresh status.
  `resumed` and `capped` each end the run's one retry.
  Every entry of the halted arm, not only one after a restart, finishes a retry it finds unfinished: `started` with the child still blocked at Publish repeats the goto, and `started` or `goto-done` with the child paused at the target resumes it.
  An unfinished marker with the child in any other state, such as `goto-done` with the child blocked at Publish again after a resume whose `resumed` write was lost, counts as the retry spent.
  The goto's blocked-to-paused change ends the InnerRun wait, so the paused arm's resume on `goto-done` is the retry's normal path.
- Each automatic retry is logged at Info and recorded as a batten history entry, so the repair is visible beside the run's other steps.
- A later Publish verify failure in the same batten run, once the retry is spent, halts as today and gets the blocked notice, which says the automatic retry already ran.

This changes battenshed's documented rule that batten never spawns or resumes a halted child, blocked or paused, for this one case.
Bound: once per batten run, a capped retry counting as that one; only for state blocked at Publish; only with a failure record written during the halting attempt, which Publish writes only for a verify that exited non-zero; a record without the target field or the write time is not retryable; the goto and resume retries stop at the time cap.

### D6. "cannot lock ref" joins fabric's push retry

Fabric's bounded push retry (today `pushretry.go`, only around Add's two branch pushes and only for a bare "(failed)" refusal) moves to the plain, never-forced push primitives every fabric push runs through: `internal/gitrepo`'s `Push`, `PushCoalesced` and `PushRebaseFree`, Add's branch pushes and the archive tag push.
The leased update (`UpdateRemoteBranchLeased`) and both remote-branch deletes do not retry.
It retries a remote refusal of either form on the pushed ref's own `! [remote rejected]` line: a bare "(failed)", or a reason holding "cannot lock ref".
That check runs before `rebaseRetryTriggers` and before the mapping to `ErrPushRejected`, so while attempts remain such a refusal never triggers a `pull --rebase` and never surfaces as divergence.
Once the attempts run out, the last push error is returned as a plain push failure, never as `ErrPushRejected` and never followed by a rebase.
What each caller then gets: Publish goes Stuck with `push failed: ...` instead of the divergence text, and the coalesced rebase-free path (`pushRebaseFreeLogged` in `internal/fabricengine/coalesce.go`) keeps today's outcome for this refusal, a Warn that leaves the commits unpushed for a later push, so the coalesced push, the `fabric sync` push child and the verbs on them gain no new error.
To tell it apart, the exhausted error wraps a sentinel of its own, distinct from `ErrPushRejected`.
The attempt count and backoff stay as they are.
The retry runs inside whatever lock the caller already holds, and never rebases or forces, so the push-both-sides shape is unchanged (PATTERN-push-both-sides).
The detached push child of `fabric sync` runs the same primitives and so retries too.
Rationale: the common cause of the refusal is a server-side ref-lock race that clears on retry; fixing it at the push keeps the transient mark and its closed class set unchanged (PATTERN-transient-stop).
Bound: only a remote-rejected push of those two forms, from the primitives named above, is retried; a local "cannot lock ref" and every other rejection fail as today.
A permanent remote "cannot lock ref", such as a ref name conflicting with an existing ref, is retried to the attempt cap under the caller's lock before it fails.

### D7. The approval paths need no orch action

After `lyx loom approve` or `lyx loom reject` at PR-Gate, batten's awaiting arm already resumes the child, retrying while the driver has not parked.
A parent-review approval inside the gate's wait needs no resume, since the Discussion row is still running; an approval after the gate already halted the run is a halted child, whose notice reaches the orch and whose way forward is `lyx loom resume`, as for any halt.
The orch role stencil states that batten resumes the child and that the orch runs no `lyx loom start` loop after either approval.
No Go change is planned for these paths; Acceptance criteria pin the PR-Gate one.

### D8. The orch's remaining landing duties

The orch role stencil's `## After landing` section is replaced: a landing needs nothing from the orch, a Land or teardown notice is a stop to investigate, and a ready-task notice is the cue to start the named tasks.
At PR-Gate, after approving, the orch amends the board entries the run superseded; an approved run that later fails to land is a halt the orch handles anyway, so amending before the landing is acceptable.
The orch takes the GitHub inbox onto the board (`lyx board intake`) at PR-Gate and at any other wake, since triage needs judgement.
The fabric commit of hub config edits is the orch's whenever it makes such an edit, not a landing step.
While the hub's `deploy` key is empty, the orch deploys by hand on a ready notice or at its next wake after an approval.
The Publish verify failure line under `## Escalations from a child` says batten retries the first one, so a notice for one means the second failure, which goes to Investigating a stop.

### D9. `CLAUDE.md` and the refusal spec

`CLAUDE.md`'s `## Production` standing exception says batten's Land row runs the hub's `deploy` command after a landing, and the hub orch runs `update-plugins.sh` itself only after its own pushed fix, on a clean `main`, and says so in its report.
`contracts/specs/refusal-spec.md`'s "Publish: verify failed" row says batten takes its way forward once on its own.

## Constraints

- PATTERN-fabric-git: the pull and every push go through `internal/fabricengine` and `internal/gitrepo`, in-process.
- PATTERN-told-geometry: `internal/battenshed` is told the prime path, the board dir, the deploy command and every seam (D1).
- PATTERN-config-strictness: `deploy` is read through landing's existing config loader.
- PATTERN-orch-pane-single-writer: every notice is queued through the existing Notify seam, never typed.
- PATTERN-spawn-observability: the deploy command's spawn and wait are logged.
- PATTERN-no-denied-recovery: every way forward a notice or Stuck names is a command the orch may run.
- PATTERN-refusal-way-forward: the Publish refusal-spec row changes with D5 (D9).
- PATTERN-lyxdirs-single-declarer and PATTERN-durable-vs-ephemeral-state: the land lock and the new markers live under `.lyx`, their paths declared through `internal/lyxdirs`.
- The batten recipe's row names are durable on-disk identities: the new row gets a `Name*` constant in `internal/battenrecipe` and its coverage guard.
- Docs in the same commit: `internal/battenshed/doc.go`, `contracts/recipes/batten-recipe.yaml`'s header, `internal/landingshed/doc.go` for `deploy` and the record field, `internal/fabricengine` and `internal/gitrepo` docs for the push retry, the orch role stencil, `CLAUDE.md`, `refusal-spec.md`.

## Auto-mode assumptions

- The orch sets the hub's `landing.yaml` `deploy: ./update-plugins.sh` from the prime during this task's own landing, which still runs the old routine; the PR description names that step.
- Deploying on every landing into the prime's branch is acceptable even when two landings come close together; the second rebuild is redundant but harmless.
- Running the Land row before Worktree-Teardown delays the deploy by the driver's exit wait, which can be minutes (its stop report includes the friction reflection); that is acceptable.
- Waiting at most ten minutes on a dirty prime, then halting with a notice, balances a prime agent mid-commit against a forgotten edit.

## Open risks

- Tasks `config-layers-run-settings` and `burler-split-budget` run concurrently and edit the orch role stencil (`## The board`, `## Escalations from a child`) and, for the former, landing config's loader; the later-landing task merges against the earlier.
- Task `burler-split-budget` renames the Webster review round's rows; the verify-failure target and the refusal-spec row then change with it, and a stale target reaches D5's not-retried refusal path.
- Notes `alignment-agent` and `merge-review` may take PR-Gate off the orch; the duties D8 puts at PR-Gate then need another trigger.
- A run's own Friction-Reflect issues are filed after its PR-Gate, so they reach the board at the orch's next wake, which may be late when no other run is in flight.
- Old-binary batten and loom processes still call `hubreconcile.Ensure` after a deploy, the stamp flip-flop `hubreconcile` already accepts; deploys inside running battens make it more frequent.
- A batten killed between Run-Shed's Done and the Land row sends nothing until it runs again, where the removed `done` notice used to go at first sight.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d5-capped-retry-no-recorded-end (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A child that ends done is pulled into the prime, deployed and pruned by the Land row, and batten reaches done with no notice sent.
- A landing into a parent other than the prime's branch skips pull and deploy and still prunes.
- A landing into a prime branch outside `require_pr_to_base` pulls, skips the deploy with an Info log, and still prunes.
- An empty or absent `deploy` runs nothing; a set one runs once per landing in the prime.
- A deploy that exits non-zero or times out halts batten blocked at `Land` with the pair intact and sends one notice naming the step, the log path and `lyx batten run <slug>`; a batten that dies after queuing the notice and before halting sends no second notice on restart, while a `lyx batten run <slug>` after the halt that fails again sends a new one.
- A prime with uncommitted tracked changes is waited on and, past the bound, halts `Land` with one notice; a prime that becomes clean within the bound lands; a landing that skips the pull does not wait on a dirty prime.
- Resuming batten after a Land failure re-runs the row; pull, deploy and prune on an already-landed prime succeed.
- A Worktree-Teardown failure sends one notice.
- Two Land rows run one after the other, and a Worktree-Create waiting on the prime lock is not held for the length of a deploy.
- A landing that empties a task's `depends_on` sends one notice naming the newly ready task; a landing that readies nothing sends none.
- A batten that dies after the prune and before the ready notice sends that notice on restart.
- When one landing's prune also removes a second, concurrent landing's done entry, the tasks it frees are named in a notice exactly once.
- An in-flight batten status from before this change reaches `Land` after its Run-Shed Done.
- A `deploy` write from a pair is refused.
- A child blocked at Publish with a failure record matching the halt is sent to the record's target producer and resumed once, with no notice for the halt or for the paused state at the target, also when the resume waits past the driver's stop-report wait; a second such halt in the same batten run sends the blocked notice saying the retry already ran.
- A child blocked at Publish without a matching record (none, a stale one including one from an earlier halt at the same HEAD, or one lacking the target or write time) is not retried and gets the blocked notice.
- A record written in the same whole second as its halting Publish entry is retried; a stale record from an earlier same-HEAD halt, written in the same second as that halt's entry, is not.
- A goto refused while the driver holds the run lock, and a resume refused as not parked, are retried until they land within the time cap; past the cap batten sends the notice for the child's actual state (blocked at Publish for a refused goto, paused at the target for a refused resume) with the last refusal; a restart does not reset the cap; after that notice and an orch resume, a second Publish verify failure gets the blocked notice and no goto.
- A goto refused for an unknown target is not retried and sends the blocked notice.
- A batten that dies after the automatic goto and before the resume resumes the child on restart; one that dies before the goto repeats it.
- A push refused with a remote "cannot lock ref" or a bare "(failed)" is retried within the existing attempt bound, with no rebase and no `ErrPushRejected` between attempts, and succeeds when the race clears; a local "cannot lock ref" fails at once; the leased update and the deletes are not retried; once attempts run out, Publish goes Stuck with the push failure and the coalesced push Warns and leaves the commits unpushed.
- After `lyx loom approve` with the driver not yet parked, batten resumes the child with no orch command.
- The orch role stencil no longer lists pull, deploy, reconcile, prune or the teardown check after a landing, and tells the orch not to run a `lyx loom start` loop after an approval.

## Notes for the plan writer

- The InnerRun notice step and its episode markers live in `internal/battenshed/notice.go`; the approval-resume retry is in `awaitingStep` in `innerrun.go`.
- The verify-failure way-forward strings are in `internal/loomcli/landingdeps.go` and `darnwiring.go`; the record is in `internal/verifytree/failure.go`.
- The orch-note rules being retired are in the prime's `.lyx/orch` note; the stencil change overrides them, and the note is never edited by hand.
