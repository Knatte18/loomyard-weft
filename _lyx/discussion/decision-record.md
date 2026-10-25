# Decision record — umbrella-daemon

## Goal

Every beam, the hub's prime included, gets dobby: a Go daemon with no LLM that starts the beam's ready tasks, drives their batten runs, lands them and cleans up.
It replaces the one waiting `lyx batten run` process per run.
It escalates only what needs judgement: first to the arbiter, then to the nearest live orch.
The orch stops starting or cleaning up runs, and resumes a halted or paused one only by a hand pass (D4); one `lyx orch start` after a machine restart brings every run back.
In the same task, the arbiter becomes part of every shed run's stop chain, batten included.

## Scope

In:

- dobby, the beam module's daemon, and `lyx beam start`, called by `lyx orch start`, `lyx beam create`, `lyx beam promote` and `lyx adopt`.
- `lyx adopt`'s refusals of a task or beam without its parent owned elsewhere, and of a beam without its members owned elsewhere (D2).
- Batten in short passes: the Run-Shed row's InnerRun checks its child once per pass instead of waiting in one long process, and dobby schedules the passes.
- `draft` as a board task status, the default for a new task, and readiness computed by dobby from the board.
- The landing notice's `newly ready` part and the orch's start-on-notice steps go; every kind of batten notice is revised (D9).
- Automatic revive of a session stalled on an API error.
- The board-tidy Sonnet agent every N landings, spawned by dobby.
- Key-level reporting in Finalize's per-worktree config notice.
- The arbiter seam moved into shed beside the tackler, wired for every shed-driven recipe, with its keys in `shed.yaml` and per-module overrides, and a description of batten's rows (parent decision, 2026-10-25).
- The orch role stencil, the `lyx batten run` help, the module docs, `docs/overview.md` and `PATTERN.md`, with "umbrella" renamed to "beam" in every text this task touches.

Out:

- Moving batten's logic into dobby: batten keeps its rows and their code, and dobby only schedules passes.
- Changing `lyx tender`: it stays the hub-wide brake, apart from dobby (operator decision, 2026-10-25), and `lyx tender revive` stays a manual verb.
- Noticing a live strand left idle with a background fork after `lyx tender resume`: the quiet notice covers an idle run, and nothing more is built for it.
- A per-beam anchor (`umbrella-anchor`), and the darn pieces inside a beam (`loom-beam-darns`), which builds on this task's dobby.
- Note `landing-escalate-arbiter`: D8 leaves loom's landing-halt rule as it is, so that note's change stays its own.
- Starting dobby when the machine starts.
- Removing the orch's hub-wide rule section "Board tidy after a landing": that is a Bolt write through `lyx orch rules`, which the parent does once this lands; D9's dropped `landed` notice leaves the rule with no trigger, so it cannot double a tidy meanwhile.

## Decisions

### D1 — dobby is the beam module's daemon, one per beam worktree

Dobby lives in the beam module: the verbs in `internal/beamcli` and the daemon in a new told-geometry engine package (`internal/dobbyengine`), run by a hidden `lyx beam dobby` verb, the way `lyx orch watch` runs the watcher.
There is one dobby per beam worktree on its owning machine: the hub's prime, which is the root beam, on the recorded prime owner's machine, and each local pair whose origin record carries the beam mark, on its owner's machine.
It holds a lock file and a state record under its worktree's `.lyx/beam/`, both named through `internal/lyxdirs`; the state is a tolerant JSON record written through `internal/state`.
A second dobby for the same worktree exits at once on the held lock.
A dobby that reads its beam owned by another machine, at start and on each probe tick, exits.
Its creel sender is the program `DOBBY`: `ly:DOBBY` for the prime and `ly:<slug>:DOBBY` for another beam, formed through `internal/agentname`.
Its working directory is the hub root, never a worktree, and it is told absolute paths, so the teardown of a pair never runs inside it.
It registers in `internal/runproc` with a new role, `RoleDobby`, under the beam's slug, or `runproc.HubOwner` for the prime, so the pair teardown of a landed beam ends that beam's dobby.
Bound: `runproc.End` of the beam's own slug, from a cancel or owner pause of the beam's landing run, also ends its dobby; by then `BeamLandRefusal` has left the beam no members, so the dobby drives nothing, and `lyx beam start` brings it back if the beam stays.
`lyx reed down` of a beam's session, through `pairteardown`'s `EndBeforeSessionDown` (reached from `internal/reedcli/up.go`), spares a `RoleDobby` record and ends the slug's other processes: dobby runs from the hub root outside the session, and its member runs still need it.
On a changed `lyx` binary it re-execs itself between ticks, as the orch watcher does.

Rationale: the operator put beam and dobby into one module (2026-10-18), and the orch watcher is the existing shape for a detached daemon.

### D2 — `lyx beam start`, and every beam has a dobby from birth

`lyx beam start` starts, detached, the dobby of every beam worktree this machine owns whose dobby is not live, and reports for each whether it started it or found it live.
With no prime owner recorded, it records this machine first, as a batten resume does; with another machine recorded, it starts no prime dobby here and says so.
It is idempotent and runs from the prime or any beam.
`lyx orch start`, with or without `--continue`, calls it first; a failure there is a Warn on orch's envelope naming `lyx beam start`, and never stops the orch's start.
`lyx beam create` and `lyx beam promote` start the new beam's dobby as their last step, and `lyx adopt` ends with `lyx beam start`, so a dobby runs wherever the adopted runs' parent now is.
`lyx adopt` of any entry it records an owner for, a task or a beam, whose parent reads owned by another machine refuses, naming `lyx adopt <parent>`, or `lyx adopt --prime` when the parent is the prime, unless the same call adopts the parent; otherwise no dobby here would drive the adopted run, a beam's landing run included, since this machine's dobby drives only runs whose parent is its own worktree (D3).
For the same reason, `lyx adopt` of a beam, the prime included, refuses while a member that is not done records another machine as its owner, naming those members, unless the same call adopts them.
Both refusals are decided before the owner write, against the entries that passed step 1 and go into it, so a parent or member the same call stops at step 1, as a diverged or live pair, does not count as adopted.
`adoptcli`'s `parentWarning` already judges every entry the parent way, but after the write; its judgement moves ahead of the write as the refusal.
Each refusal names one call that meets both rules, `lyx adopt` with every slug the two rules reach from the named entries, `--prime` included when the prime is among them, so following it meets no second refusal.
Accepted bound: a parent the batched owner write loses to a race while it records the child is not seen by the refusal; the child keeps today's warning naming `lyx adopt <parent>`.

Rationale: the operator decided `lyx beam start` and that `lyx orch start` calls it (2026-10-18), and that a run's parent always has a running dobby (2026-10-25).

### D3 — which runs a dobby drives

A dobby drives every run on this machine whose parent worktree is its own.
It finds them from the local batten status files of the hub, never from the board, so a run whose task Land already pruned is still driven through Worktree-Teardown; each run's parent is the one its pair's origin record names, per `PATTERN-pair-parent`.
It watches the batten runs directory, so a run another verb starts is seen at once.
It drives only runs this machine owns, per `PATTERN-owner-machine`.
A run another machine takes over is found by the origin reconcile of a full pass (D4), after a dobby start or a machine wake, or while the run's origin-owed marker stands, and dobby then handles it as owned elsewhere, as D4 says.
Between full passes, InnerRun's owner check keeps its pause-lift and owned-elsewhere triggers across passes through the memory record (D4).
A beam's own landing run is a run whose parent is the beam's parent, so the parent's dobby drives it.

### D4 — batten runs in short passes that dobby schedules

`lyx batten run <slug>` becomes one pass: it steps the lifecycle through the Go driver as today, and the Run-Shed row's InnerRun does one check of its child per Call instead of waiting.
That one check is `check()`'s full body, never `wait()`'s first round: `coverage`, `foldOwnerTrigger`, `checkChild`, `ownerStep`, `refreshWatched` and `lockStep` on a due probe, `noticeStep`, the arm's step and the pause-flag read, with the probe judged due from the persisted `lastProbe`.

The yield is a new shed outcome, `Yield`: InnerRun returns it when its check ends nothing.
Shed persists nothing on a yield, so batten's status file, history and activity stay as they were, and the step-until-stop loop ends.
The Go driver reads a yield as no stop: no rule-table row, no stop report, no repair, bounce or tackler budget, and the pass exits 0.
Bound: a yield leaves the run exactly as it was, at the same row and `running`; only a producer at a wait that its own next pass re-checks may return it, and InnerRun is the only one.

`battenPreRun` in `internal/battencli/arm.go` refuses a pass first when a pause entry covers the slug (`refuseCovered`), which would refuse every pass under a pause; so `--continue-pass` skips that refusal, and a continue pass runs under a covering pause.
Its one check then keeps releasing an idle owner's parent lock and sending the child's notices under the pause, as today's long wait does.
Dobby runs no full pass of a run while a pause entry covers the hub or the run's slug, read from the same pause record it reads for D5 and D12; the full pass at the lift stays as below and in D12.
A full pass refused at pre-run for a covering pause, as when a pause begins between dobby's read and the pass's, exits with a code of its own, and dobby reads it as covered: never a failing pass, counted toward neither the three-failures notice nor D7's failing-pass marker.
A hand pass under a covering pause keeps today's refusal naming `lyx resume`.
Accepted bound: a run owed a full pass while a pause covers it, after a dobby start or a wake under the pause, gets no pass until the lift, so its idle parent lock and its notices wait for the lift.
InnerRun's spawn hold under a covering pause or a standing owner (`holdSpawnWhileCovered` in `internal/battenshed/innerrun_pause.go`) returns `Yield` instead of sleeping, after writing the hold reason to the reason file as today, and the pass prints that reason as it exits.
Before yielding on a standing owner, the hold runs the owner check when a notice probe is due by the persisted `lastProbe`; an owned-here verdict clears `ownedBy`, and the pass goes on to the spawn in the same pass.
The other in-pass waits stay, each with its existing bound: the prime-lock wait of Worktree-Create and Worktree-Teardown (`primelockwait.go`, ten minutes), Land's wait for a clean prime (ten minutes), Land's deploy run (`deployTimeout`, 30 minutes), the row notice's spacing between delivery attempts (30 seconds, at most two), and a stop's tackler and arbiter sessions under their own timeouts.
Three in-pass waits last as long as what they wait on.
InnerRun's spawn runs the child's `lyx loom start --no-attach` bootstrap synchronously, with no timeout of its own, so it lasts as long as that bootstrap.
The Go driver's pause hold (`holdPause` in `internal/sheddriver/drive.go`) sits inside stop handling, so it stays: it lasts while a pause covers the slug, and the pass goes on to the stop's next settler at the lift.
Land's land-lock poll has no time limit of its own: it lasts while another landing holds the land lock, and it ends at batten's own pause flag, never at a tender pause entry.
Bound: a pass stays in flight only through those waits.

A pass dobby schedules into a running batten run at Run-Shed is a continue pass (a hidden `--continue-pass` flag): it skips the resume's origin reconcile and the Open-Window re-run.
Every other pass is a full pass that runs both: the first pass of each run after a dobby starts, the first pass of each run after dobby sees a machine wake, every pass of a run with a full pass owed, and every hand pass, the one that resumes a halted or paused batten run included.
A continue pass is not a resume under `PATTERN-owner-machine`, whose change below says so: it goes on with a run the same dobby already drives, so it needs no origin verdict, while each full pass is a resume and gets one.
A re-exec on a binary change is not a start: the next passes stay continue passes, and a full pass owed to a wake is kept in the state record across it.
InnerRun's owner check is due today only after a sleep the same process watched, a pause lift, or a probe check while the run stands owned elsewhere (`foldOwnerTrigger` in `internal/battenshed/innerrun_owner.go`), so it never fires on a sleep in a short pass.
Dobby therefore holds the sleep watch: it pairs its wall clock with an awake clock on each tick, as InnerRun's `pauserecord.SleepWatch` does, records each sleep it sees in its state record, and owes every run it drives a full pass.
A dobby start owes every run a full pass the same way; an owed full pass is kept in the state record and cleared when a full pass of that run exits 0 with no origin-owed marker standing, so a failed one is retried as a full pass.
A full pass whose origin was unreachable judges the owner against the local Bolt, as `refuseResumeUnreachable` does today, and when it resumes it first writes a per-run origin-owed marker, a tolerant record under the run's scratch directory named through `internal/lyxdirs`; a full pass that reaches origin and exits 0 removes it.
The pass itself records the episode, so a hand pass, the detached first pass of `lyx beam land` or `lyx adopt`, and a pass dobby spawned all leave the same marker.
Dobby owes a run a full pass while its marker exists, and runs each such pass with the hidden `--no-reopen` flag, which skips only the Open-Window re-run, so an hour offline opens no window per probe tick.

Every refusal of a pass that reads another machine as the run's owner exits with a code of its own, distinct from the `--continue-pass` refusal's, through one typed error that `ownedElsewhereRefusal` and the owned arm of `refuseResumeUnreachable` both return.
Examples are `resumeRefusal`'s `EntryOwnedElsewhere` arm, `resumeOwnerClaim`'s lost owner claim, `refuseResumeUnreachable`'s refusal on a local Bolt showing another owner, and `refuseStartOwnedElsewhere`, which `ownerPreflight` reaches for a found status still at Worktree-Create with no pair on this machine.
The code is set only for a refusal before the step loop, never from a row's error.
A first pass with no status file is not yet a run dobby drives: when `refuseStartOwnedElsewhere` refuses it as owned elsewhere it exits with the same code and stays a refused start under D5, with no owner pause, since there is no run to pause.
A claim `claimBoardEntry` loses inside the Worktree-Create row fails that row with the refusal's text naming `lyx adopt <slug>`, so the stop goes through D7's shed chain like any row failure;
dobby passes no failed run, and a hand pass of it reaches `refuseStartOwnedElsewhere` at the found Worktree-Create status and exits with the code.
Dobby reads that code from a run it drives, one with a status file, as owned elsewhere, never as a failing pass.
It then keeps the run's full pass owed, records the run as owned elsewhere in its state record, and runs today's owned-elsewhere handling: `tendercli.PauseOwnedElsewhere`, the same parent-lock release InnerRun's `ownedElsewhere` makes, and the owner notice naming `lyx adopt`, once per owner episode.
A failed owner pause warns once per episode and is never retried blind: while the mark stands and no pause entry covers the run's slug, dobby runs a full pass of the run on each probe tick instead, so the owner is judged again before any pause.
That pass is either refused as owned elsewhere again, and dobby runs the owned-elsewhere handling again, its pause included, with the episode's marker keeping the notice to once, or it exits 0.
While the mark stands and a pause entry covers the slug, dobby runs no pass of that run, a dobby start's and a wake's included, but the full pass at the lift of the covering pause; `lyx adopt` reaches it the same way, since adopt lifts the owner pause.
A full pass of a marked run clears the mark when it exits 0, and a further owned-elsewhere refusal leaves it standing.
An owner InnerRun's own check finds inside a continue pass sets no such mark: InnerRun pauses the run itself, dobby goes on passing it in continue passes, whether or not that owner pause established, and its spawn hold re-checks the owner on each due probe; the lift of the owner pause, as by `lyx adopt`, gets the run its full pass.
The pause-lift and owned-elsewhere triggers persist in the memory record below.
Bound: the pass refuses `--continue-pass`, naming a plain `lyx batten run <slug>`, unless batten's status reads `running` at Run-Shed, the state record of the parent's dobby names the run as one it drives with no full pass owed, and no origin-owed marker of the run exists;
the flag skips only the origin reconcile, the Open-Window re-run and the pre-run pause refusal, never the owner check's triggers.
The refusal exits with a code of its own, and dobby reads it as a full pass owed rather than a failing pass.

InnerRun's memory across checks moves to a per-run tolerant record under the row's scratch directory, so its once-per-episode, backoff, idle-bound and grace rules keep their meaning across passes.
The inventory, from `internal/battenshed/innerrun.go`, `notice.go`, `innerrun_binaryrestep.go` and `innerrun_parentlock.go`:

- The producer's notice episode, `noticeEpisode`: its state, its since, `arbiterOffAt` and each notice's delivery (attempts, last try, strand warned, dropped), so each notice goes once per episode, its retries stay spaced and capped, and the arbiter-off grace counts from the first sight.
- `driverBusyNoticed`, so a busy driver pane is noticed once per stall.
- The owner trigger's `ownedBy`, `ownerNoticed`, `ownerDue` and `ownerCovered`, so the owner check's retry and pause-lift triggers fire across passes and the owner notice goes once per owner episode.
- The `failures` episode set, so a failing seam warns once per failure episode and logs its recovery.
- `restepWatch`, so the binary re-step keeps its build-check interval, its attempt count and its logged dirty-path list per halt.
- From the per-Call `childWait`, which today restarts with every Call: the parent lock's `lockWatch` (its idle watch and the owner it counts for), so an idle owner's lock is released after the idle bound;
  `resumeRetryAt` and `notParkedLogged`, so the awaiting arm keeps its resume retry spacing and logs a not-parked refusal once;
  `busyLogged`, `busyReason` and `busyKey`, so a busy-pane refusal logs once;
  `lastProbe`, so the costly probe work runs at most once per `notice_probe_s`;
  `reason`, `reasonKeys`, `pauseWarned` and `coverageWarned`, so the reason file is rewritten only on a change, a wait reason logs at Info and a persistent read error warns once per child state;
  `revival`'s failed-revive mark (`driverRevival.startFallback`), so the halted reason keeps its `lyx loom start` fallback for the rest of the episode.
- New cross-pass counters: the respawn count of a running child and the child status file's modification time at the last respawn, so the respawn cap below trips and resets on a status write;
  the api-error episode whose revive line was sent, so D9's revive goes once per episode;
  and the count of revived api-error episodes in a row with the child status modification time it last saw, so D9's third-episode notice trips and a status write resets the count.
- The idle checks' sleep watch, and the owner trigger's sleep watch (`ownerSleep`), are replaced by the sleeps dobby records, which every pass reads as excluded time, and by the owed full pass, so idle clocks still measure awake time and a wake still reaches the owner verdict.

The rest of `childWait`, including `revival`'s strand reading, is each check's fresh reading and stays per pass.
Two rules stay per pass on purpose: the spawn confirmation, so each pass re-checks the child's strand and locks and respawns a driverless running child, and the batten-watched marker's write, which each pass makes with the pid of the dobby that schedules it, read from that dobby's state record.
The respawn of one running child is capped at three in one run of respawns with no progress between them; a write of the child's status file since the last respawn resets the count, and a live driver strand or a held lock alone does not, so a driver that dies minutes after each respawn still reaches the cap.
Past the cap the pass sends the driver-dead notice and respawns no more until the count resets or the child's state changes.
Every other rule stated per batten process today becomes per episode, the revive of a halted child's dead driver strand included.
Bound: a file that ends a wait between passes, such as a stop report or a strand's end, is seen at the next pass, so each such wait can end up to one `notice_probe_s` later than today, and none is lost.

Dobby runs a pass of a run it drives only while batten's own state is `running`, never while the slug's cancel marker exists, only continue passes and the full pass at the lift while a pause entry covers the hub or the slug, and, for a run its state record names owned elsewhere, only the full passes named above.
A run so named is not in flight for D5's `max_runs` count, and is in flight for D13's backoff only while no pause entry covers its slug, since dobby then passes it on each probe tick.
A halted or paused batten run is resumed by a hand pass, `lyx batten run <slug>`, run by the operator, or by the orch on the operator's word or after judging the stop; dobby never resumes one.
It runs one:

- when the child's status file is written, seen through `internal/fswatch`;
- when batten's own status file is written while no pass of the slug is in flight and with a modification time later than the exit of the slug's last pass, so a pass's own writes never trigger the next, even when their event arrives after the pass exits;
- on a probe tick, once per batten's `notice_probe_s`;
- at once when the dobby starts, and when a tender pause entry covering the run is lifted.

It never runs two passes of one slug at once, and reads a held driver lock (`shedrun.DriverLock`) as a pass in flight, so a pass left by a dead dobby is waited for and never doubled.
A pass runs in its own process group, so a dobby that dies leaves it to finish.
Bound: per run, at most one pass per `notice_probe_s` plus one per write of the child's status file, and at most one in flight.
A pass that exits non-zero, bar the refusal codes named above, is retried on the next probe tick; three failures in a row with the same error text post one notice per error text to the nearest orch, and the retries go on.

`pairteardown`'s liveness read itself (`battenAlive`) reads a run as having a live lifecycle owner when a registered batten lives, or when a live dobby drives it and batten reads `running` with no cancel marker, the runs dobby passes.
So every reader of it, the claim settlement, the session end and `driverChecksSkipped` included, never reads the gap between passes as a gone batten, and still reads a halted run as settled.

`--window` and the hidden `--open-terminal` are retired, which the entry did not ask for but follows from dobby starting every run; the open_window setting alone decides the Open-Window row.
`battencli.ResumeInWindow`'s callers, `lyx adopt` and `lyx beam land`, start the run's first pass detached instead and then make sure the parent's dobby is live.
A hand-run `lyx batten run <slug>` runs one pass in the foreground and then starts the parent's dobby when it is not live.
The run removes note `batten-window-default` with `lyx board remove`, since its idea is moot.

Rationale: the operator agreed that one daemon replaces one waiting batten process per run, and that batten drives one task's lifecycle as today and knows nothing of its owner.
Short passes keep every row and notice in batten's code; persisting InnerRun's memory keeps its once-per-episode rules true when each pass is a new process.

### D5 — `draft` is a task status, and dobby starts ready tasks

`draft` is one more value of a task's `status`: not a run status, so it locks nothing.
An empty status means a nil status or an empty string, in the start condition, the migration and the release paths alike.
One rule, applied where `boardengine`'s store commits an entry, gives `draft`: a write that leaves an entry of kind task with an empty status gives it `draft`, unless the entry was already a task before the write and the write is a release or leaves the status untouched.
So a create, a batch or merge create, a note or project turned into a task by an upsert, a note promoted by `lyx board promote`, and an empty-string status written to any task all give `draft`, whatever the verb, and a later edit of a released task keeps it released.
`lyx batten cancel`, with or without `--remove`, leaves the task `draft` rather than clearing its status, so a cancelled task is not started again.
Pair teardown's claim settlement does the same: its `claimClear` arm (`settleClaim` in `internal/pairteardown/claim.go`, for an unseeded, paused, blocked, failed or awaiting run) writes `draft` through `writeBoardClaim` instead of a nil status, so a task whose pair `lyx fabric remove` removed is not started again.
A release is a write that sets `status` to null on an entry that was already a task: `lyx board set-status '{"slug":"<slug>","status":null}'`, which is how the operator, or the orch on the operator's word, releases a task, an upsert whose payload sets `status` to null, and a merge whose `set_status` step sets a null status.
An empty-string status is never a release, and a write that makes an entry a task gives it `draft` even when its payload sets `status` to null.
Since the store cannot tell a release from any other null write, no lyx code path writes a null task status of its own: each one that clears a task's status writes `draft` instead, so a null status reaches the store only from a payload the operator or the orch writes.
Batten's start guard refuses a fresh start of a draft task, naming `lyx board set-status '{"slug":"<slug>","status":null}'` with the slug filled in, so a hand-run start cannot skip the release and the way forward runs as typed.
The README keeps a draft task in its Ready or layer section with its After column, marked `draft` before its title.

A one-time board migration marks every open task with an empty status `draft` and records a top-level `draft_status` key in `board.json` in the same write, so tasks made by an older binary before this lands wait for a release; a board holding the key is never migrated again, and an older binary's rewrite keeps the key, per `PATTERN-tolerant-records`.
Accepted bound: a task an older binary on another machine creates after the migration has no status, so it starts without a release until every machine runs the new binary.

A dobby starts a task when all hold: kind task; status empty; every `depends_on` done; its parent is the dobby's worktree; its owner is this machine or none; no tender pause entry covers the hub or the slug; and fewer than `max_runs` runs it drives are in flight, halted ones included and ones named owned elsewhere (D4) not.
Starting is the run's first pass, which claims the task as batten does today.
`max_runs` is a new key of the beam module's config, `beam.yaml`, default 3, settable per worktree layer; 0 starts nothing.
A first pass refused at pre-run for a covering pause (D4) is not a refused start: it posts no notice, sets no marker and starts no backoff, and the start waits for the lift.
Any other first pass that batten refuses before the claim posts one notice per refusal text to the nearest orch, and the start is retried from one probe interval, doubling up to an hour, and at once on any change of the task's board entry; the backoff is kept in the dobby's state record.
Dobby reads the local board on a write to `board.json` and on its probe tick; a release written on another machine reaches it when this machine's board is next pulled.
Bound: a task is started only from an empty status, by its parent's dobby or a hand-run `lyx batten run`, and never twice while a pass of it is in flight; across machines, only the owner machine's dobby of the parent runs.
Since the draft rule sits at the store's commit point rather than in each verb, a task is left with an empty status only by a release, the older-binary bound above aside.

Rationale: the operator decided readiness without batten notices (2026-10-19) and draft as a status (2026-10-25); "a new task is a draft by default" (2026-10-18) carries over to the status.

### D6 — the landing notice names no ready task

The Land row's ready snapshot, its `newly ready` routes (`LandDeps.ReadyRoutes`) and their marker lines go.
The orch role stencil drops the start steps (`lyx batten run <slug> --window`, start-on-notice and the creel-log fallback) and the window wording of its landing sections.
The orch never starts, stops or cleans up a batten run, and resumes a halted or paused one only by a hand pass (D4).
The stencil says so: the orch runs `lyx batten run <slug>` on a halted or paused run only on the operator's word or after judging the stop.

### D7 — stops: shed's chain for runs, the orch for dobby's own failures

Each batten pass steps through the Go driver, so a batten stop meets the rule table, the tackler and the arbiter (D8) before anything reaches the orch, and the child run's own driver does the same for its stops.
Dobby adds no rule table: the mechanical stops are settled where they are today.
Dobby's own failures, a refused start (D5) and a pass that keeps failing (D4), are not shed stops, and dobby posts them to the nearest live orch up its beam chain, the existing notice routing, each with its marker under `.lyx/beam/`:
one notice per run and error text for a failing pass, cleared when a pass of that run exits 0,
and one per task and refusal text for a refused start, cleared when a pass claims the task or the task's board entry changes.
A full pass of a run dobby drives refused as owned elsewhere is neither: dobby sends today's owner notice naming `lyx adopt` instead (D4), never the failing-pass notice.
Nor is a pass refused at pre-run for a covering pause: it sends no notice and sets no failing-pass marker (D4).

### D8 — the arbiter for every shed run (parent decision, ly:orch, 2026-10-25)

The arbiter seam moves into shed beside the tackler: its spawn and its prompt move out of `internal/loomcli` into a shed-layer package that batten imports without linking tree-sitter, and `sheddriver`'s `Arbiter` seam is wired for every shed-driven recipe, loom, darn and batten.
Its keys, `arbiter`, `arbiter_timeout_min` and `arbiter_max_spawns`, move to `shed.yaml` with today's loom defaults.
`loom.yaml`, `darn.yaml` and `batten.yaml` each get same-named override keys in the tackler overrides' shape: an empty model-spec and a 0 timeout inherit `shed.yaml`, `arbiter_max_spawns: ~` inherits, and a set 0 turns the arbiter off for that module.
A hub layer that set a key in `loom.yaml` keeps its meaning as a loom override, an explicit 0 included.
Darn reads loom's keys today, so the config migration copies each arbiter key a hub layer sets in `loom.yaml` into `darn.yaml` where that layer's `darn.yaml` sets none, keeping darn's behaviour.
Only hub-layer values carry over: a value set in a worktree layer, the local layer or a task's `run_settings.loom` now applies to loom alone, and darn reads `shed.yaml` or its own override there.
The arbiter prompt names the recipe's rows through a per-recipe description, as the tackler's prompt does, and batten's description names its rows and what each does.

`internal/arbiterstop`'s eligibility gains a rule per recipe: for batten, a stop the tackler escalated is eligible unless it is a hand-back (a landing halt, a spent budget or an unmet precondition), which goes to the run's parent as today.
Loom's and darn's rules are unchanged.

Batten's Land and Worktree-Teardown failure notices now wait for the stop chain: the row notifier posts once the batten run's Go driver hands the stop back, with its stop report, instead of on the row's failure, so the orch is told only what the tackler and the arbiter did not settle.

Rationale: every shed run must be able to spawn the arbiter, to take load off the orch (operator).

### D9 — batten's notices, revised now that dobby runs

- `landed`: dropped; a landing that succeeds needs nothing from the orch, and the board tidy (D10) takes over what the orch did after a landing.
- A child that halts and whose driver handed the stop back, an awaiting child with a parent notice, the busy-pane draft stall, the owner-elsewhere notice, and Land and Worktree-Teardown failures (held per D8): kept, since each needs judgement.
- driver-dead: sent when the respawn cap (D4) is spent, or a revive of a halted child's strand failed.
- api-error: the pass first sends tender's revive line to the stalled session through the conditioned send, once per episode; the notice goes when the same episode still stands at the next probe check, or when the third revived episode in a row comes with no write of the child's status file between them.
- quiet: kept, informational.

Every kept notice keeps its text, routing and once-per-episode marker.

### D10 — the board tidy every N landings

Dobby counts a landing from the batten history of every run it drives or has driven, every run D3's status files name with its worktree as parent whatever the run's state, read on each tick, as a Land Done keyed by the run in its state record.
So a landing made by a hand pass or by a pass a dead dobby left running is counted, and a restart double counts none.
Accepted bound: a landing whose run records `lyx batten cleanup --apply` removed before dobby read them is not counted, so a tidy may come later than every `board_tidy_every` landings.
A dobby that starts with no state record records every Land Done already there as counted, so run records left from before it count nothing.
At every `board_tidy_every` landings (a new `beam.yaml` key, default 5, 0 off) it spawns the board-tidy agent with the landed slugs since the last tidy: a fresh Sonnet session at medium effort, an interactive shuttle run in a window of the beam's reed session, briefed from a stencil read at call time through `internal/stencilstore`.
The brief is today's orch rule: read each landed commit and its run's decision record against the board's notes, through `lyx board` only, make each obvious amendment, and report each note a landing superseded or changed in scope or design.
Its parent directive names the nearest live orch, so a question goes to an LLM that can answer it.
When its report exists, dobby posts its path to that orch, which puts each removal or scope change to the operator.
The tidy in flight is recorded in the state record, so a restart or re-exec spawns no second one.
Bound: one tidy agent at a time per dobby, ended through the shuttle stop verb after its report or after `board_tidy_timeout_min` (a new `beam.yaml` key, default 30).

### D11 — key-level config notice at Finalize

Finalize's per-worktree config notice compares keys, through `internal/configengine`, at the fork base, the task tip and the parent tip.
It is silent when the tip changes no key, as for an empty file or comments only, and when the parent already holds the tip's value.
Otherwise it names each changed key with its base, tip and parent values, an absent key or file reading `absent`.

### D12 — tender stays the brake

Dobby starts no task while a pause entry covers the hub or the slug.
Inside a pass, batten's existing pause handling holds every spawn, resume and revive, and InnerRun yields rather than returning Done for a done child while a pause covers the slug, so no run enters Land under a pause.
When an entry is lifted, dobby runs a pass of every running run it covered at once (D4), whose respawn, revive and driver-dead checks show whether the run came back.
A pass held at a stop by the Go driver's pause hold goes on by itself at the lift (D4), so dobby runs it no lift pass; a pass in flight at a lift in another of D4's in-pass waits gets its lift pass as soon as it exits.

### D13 — dobby's wakeups

A dobby wakes on an `internal/fswatch` event and on its probe timer, and coalesces events to at most one tick a second.
With no run in flight, its timer backs off by doubling up to sixty seconds, and any event resets it.
In flight here means a run dobby may pass on its next probe tick under D4, apart from D5's `max_runs` count, so a dobby holding only halted runs, or only runs that wait for a pause's lift, backs off.
Where fswatch fails to open, as on WSL or through a junction, the timer alone drives it at the probe cadence.

### Added after Discussion (parent, 2026-10-25): The prime is the root beam, on the same code path as every other beam

Decision: The prime worktree is a beam like every other beam: dobby and the beam code treat it as the root beam, with no special path for it. The only differences: it has no parent, so `lyx beam land` never lands it; it needs no project entry or beam mark, being a beam by being the top worktree; and the hub-wide duties stay with it (the board on Bolt, hub config, the landing deploy and the hub's prime owner).

Rationale: Operator, 2026-10-25: the top worktree is equivalent to every other beam except that it has no parent.

### Added after Discussion (parent, 2026-10-25): D5: the draft rule lives in boardCriticalSection

Decision: Take round-4 finding F1 as written: in D5, the draft rule is applied in `Board.boardCriticalSection` (`internal/boardengine/board.go`), beside `checkRunLocks` and `checkTaskCreation`, over the entries as loaded and as the mutation left them. The other two writers of `board.json`, `writeOwners` in `ownerset.go` and the replay in `replay.go`, run no draft rule: the owner write never makes an entry a task and only writes claim strings, and the replay copies entries a `boardCriticalSection` write already normalised. Add a Constraint that the rule is not placed in `Store.Save`, which holds no pre-write snapshot.

Rationale: Parent, 2026-10-25: the rule needs the pre-write entries, which only boardCriticalSection holds; checked in the code.

### Added after Discussion (parent, 2026-10-25): D2: adopt refusals are decided before any mutation

Decision: Take round-4 finding F3 as written: in D2, both adopt refusals are decided before any mutation of the call, using only the read-only part of step 1 (`ownedElsewhere`, `stoppedBranches` and `LivePair`), ahead of the `takeOverSlug` loop in `internal/adoptcli/adopt.go`, so a refused call drops no local commits and adds no pair. Reword 'the entries that passed step 1' to 'the entries step 1 does not stop', and extend the acceptance bullet on the adopt refusals with 'and mutates nothing'.

Rationale: Parent, 2026-10-25: a refused call must leave nothing behind; checked in the code.

### Added after Discussion (parent, 2026-10-25): D4: dobby lifts a stale owner pause itself

Decision: Take round-4 finding F4's second option: in D4, once its owner pause stands, dobby reads the effective owner from the local Bolt, and when it no longer reads another machine it lifts its own owner pause entry and runs a full pass. The retried pause on a probe tick needs no change.

Rationale: Parent, 2026-10-25: an adopt landing in that window must not strand a run waiting on a manual step; one owner read after the pause closes the window.

### Added after Discussion (parent, 2026-10-25): A records stop on a successful landing still notices the orch

Decision: When a landing succeeds but the prime's records did not pull, batten still notices the orch with that records-stop line: it is not the dropped `landed` notice but a failure that leaves the prime's board behind and needs judgement, like the kept Land failure notices of D9. Cards 17 and 42 keep the line and its test row, and the Shared Decision in 00-overview.md cites this entry as its rationale.

Rationale: Parent, 2026-10-25 (plan round-1 finding F6): a prime whose records did not pull is a stop someone must act on, even though the landing itself succeeded.

### Added after Discussion (parent, 2026-10-25): A pause-honoured Land stop is a hand-back under D8

Decision: A Land stop that honours a pause request is a hand-back under D8: the arbiter never relaunches over an operator's or tender's pause. Card 24 refuses a batten stop whose reason starts with `battenshed.PauseHonouredPrefix`, ending with `BattenHandBackWayForward`, with a `TestEligibility_BattenRule` row reaching it through `Check`.

Rationale: Parent, 2026-10-25 (plan round-1 finding F9): a pause is the operator's brake and no automatic step may override it.

### Added after Discussion (parent, 2026-10-25): Dobby's owner re-read uses the local Bolt, no pull

Decision: Card 34 reads the slug's effective owner from the local Bolt, as the Added-after-Discussion D4 entry says; drop the per-tick pull seam and the unreachable-origin clause.

Rationale: Parent, 2026-10-25 (plan round-1 finding F10): the adopt that closes this window runs on this machine and writes the local Bolt first, so a pull on every probe tick buys nothing.

### Added after Discussion (parent, 2026-10-25): A failed prime claim in EnsureLive is reported and skipped

Decision: Take plan round-1 finding F21 as written: in card 27, when the prime owner write fails, as with origin unreachable, EnsureLive reports it for the prime with its cause and `lyx beam start` as the re-run, starts no prime dobby, and goes on with the beams, as a batten resume's reconcileResume warns and goes on; add a TestEnsureLive row for it.

Rationale: Parent, 2026-10-25: follows D2's 'as a batten resume does'.

## Constraints

- `PATTERN-shed-driver`: the step loop stays in `internal/sheddriver`, and dobby calls no step body; it starts `lyx batten run` processes, registered in `internal/runproc` and logged through `internal/logger`, a yield at Debug.
- `PATTERN-durable-vs-ephemeral-state`: dobby's lock, state and markers sit under `.lyx/beam/`, named only in `internal/lyxdirs`.
- `PATTERN-agent-name`: `DOBBY` is the beam module's constant.
- `PATTERN-cwd-resolution`: `internal/dobbyengine` is told every path and never imports `internal/lyxcwd`.
- `PATTERN-config-layers`: `beam.yaml` and the arbiter keys are read only through `configengine`, and D8's copy into `darn.yaml` runs as a config migration through `hubreconcile`.
- Running task `discussion-scope-in-lenses` also edits the orch role stencil and renames `Merge-In` to `Parent-Align` in the landing texts `arbiterstop` and `sheddriver` hold; whichever lands second takes the other's wording in those files.
- `lyx batten run` already resumes from disk with every row idempotent, and its file notice markers already survive a restart (`internal/battenshed/doc.go`), which is what a pass relies on; the in-memory latches InnerRun holds today do not, and D4's memory record persists them.
- `boardengine`'s store accepts one on-disk version, so D5's migration records a key rather than raising the version, which an older binary on another machine would refuse to load.

## Auto-mode assumptions

- No operator answered: every choice in the question ledger was picked by the writer.
- "batten and dobby included" in the parent's arbiter decision is read as: the arbiter reaches every stop of a run dobby drives, since dobby drives batten through the Go driver; dobby's own failures are not shed stops and go to the orch (D7).

## Open risks

- A pass per probe tick per run spawns many short `lyx` processes; D4's bound caps them, at the cost of process starts (see note `spawn-audit-log`).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - note-upsert-to-task-leaves-statusless (class decision, severity MEDIUM, open in rounds 2, 3, 4)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - forced-edits-outside-cards (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - draft-migration-after-mutation (class design, severity MEDIUM, open in rounds 1)
  - lift-full-pass-unreachable (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- `lyx beam start` starts a dobby for the prime and each owned beam and reports live ones as found; a second start changes nothing; it starts no prime dobby where another machine is the prime owner; `lyx orch start` runs it first and still starts when it fails.
- A second dobby for one worktree exits on the held lock, and a dobby whose beam another machine owns exits.
- A beam made by `lyx beam create` or `promote`, and the parent of a run `lyx adopt` took, has a live dobby when the verb returns; `lyx adopt` of a member task or a beam whose parent another machine owns is refused naming `lyx adopt <parent>`, or `lyx adopt --prime` for the prime, also when the same call names the parent but stops before the parent's owner write, `lyx adopt` of a beam with a not-done member owned elsewhere is refused naming the member, each refusal names one call with every slug both rules reach, which is not refused, and both refusals are decided before the owner write.
- A task with an empty status, all dependencies done and the dobby's worktree as parent is started; a draft task, one with an open dependency, one under a covering pause, one owned elsewhere and one past `max_runs` are not.
- A new task from any creating write, a note or a project upserted to kind task, a promoted note and a task written with an empty-string status get `draft`, and an edit of a released task that leaves its status untouched keeps it released; `lyx board set-status '{"slug":"<slug>","status":null}'`, an upsert setting `status` to null and a merge with a nil `set_status` release a task; a cancelled task is left `draft`, and so is a task whose pair `lyx fabric remove` removed; a hand-run start of a draft is refused naming that `set-status` command with its slug, which runs as typed; the migration marks open tasks with a nil or empty-string status once.
- A refused start, a first pass with no status file refused as owned elsewhere included, is retried on its backoff and at once on a change of the entry, with one notice per refusal text.
- A batten pass yields at Run-Shed with the status file untouched and no cap or budget spent; the next pass resumes it; its own writes trigger no pass.
- A continue pass runs no origin reconcile and no Open-Window re-run; the first pass after a dobby start, and the first pass after a machine wake, runs both; a re-exec owes no full pass.
- A pass given `--continue-pass` for a run not `running` at Run-Shed, not named in its dobby's state record, or with an origin-owed marker standing, is refused naming `lyx batten run <slug>`.
- A run a pause entry covers gets continue passes from dobby, which skip the pre-run pause refusal and still release an idle owner's parent lock, but no full pass bar the one at the lift, and no failing-pass notice; a full pass refused at pre-run for a covering pause exits with its own code, which dobby reads as covered, and a start so refused gets no refused-start notice or backoff.
- A batten pass at Run-Shed under a covering pause or a standing owner yields at once, printing the hold reason; a pass held at a stop by a pause goes on to the stop's next settler at the lift.
- A run held at the spawn on a standing owner and then adopted back is respawned by the full pass at the owner pause's lift, or, when its owner pause failed, by the next pass with a due probe.
- A full pass owed after a start or a wake stays owed until a full pass of the run exits 0 with no origin-owed marker standing; a full pass that resumes with origin unreachable, a hand pass and a detached first pass of `lyx beam land` or `lyx adopt` included, leaves the marker, so dobby's passes stay full and run no Open-Window re-run until a full pass that reached origin exits 0 and removes it; a refused `--continue-pass` makes dobby run a full pass, never counted as a failing pass.
- A full pass of a run dobby drives, one with a status file, refused by any refusal that names another machine as owner, origin's verdict, a lost owner claim, the local Bolt with origin unreachable and the start check of a Worktree-Create status with no pair here included, exits with its own code; dobby then pauses the run under the owner, releases its parent lock and sends the owner notice naming `lyx adopt` once, sends no failing-pass notice, counts the run in neither `max_runs` nor its backoff, runs no further pass while the owner pause covers it, across a restart or a wake included, and drives the run again only after the full pass at the owner pause's lift, by hand or by `lyx adopt`, exits 0.
  When the owner pause failed, dobby runs a full pass on each probe tick instead of retrying the pause, so a run adopted back while its owner pause had failed is not paused, and its next full pass exits 0 and clears the mark.
- A halted or paused batten run gets no pass from dobby, nor does a slug with a cancel marker.
- A notice sent once per episode by today's long wait is sent once across many passes, and an idle owner's parent lock is released after the idle bound across passes; a driverless running child is respawned by a pass, at most three times with no write of the child's status file between respawns, then noticed, also when each respawned driver lives a while before it dies.
- `pairteardown`'s liveness read, and so its claim settlement, session end and quiet-wait skip, reads a `running` run between passes as owned while its dobby lives, and a halted run as settled.
- `lyx reed down` of a beam's session leaves its dobby live.
- A batten status write whose modification time is no later than the slug's last pass exit triggers no pass.
- A pass is never run twice at once for one slug, a held driver lock left by a dead dobby is waited for, and a restarted dobby runs a pass of every run in flight, a run Land already pruned included, and of a run a pause covers at its lift.
- Under a covering pause no run enters Land; after the lift, every covered running run gets a pass at once, or as soon as its pass in flight exits.
- The landing notice names no ready task, and no `landed` notice is sent on success.
- An api-error stall gets one revive line per episode, and the notice when the episode stands at the next probe or on the third revived episode in a row.
- A batten stop the tackler escalates reaches the arbiter, and a batten landing halt does not; a Land failure is noticed only after the driver hands it back; loom's and darn's eligibility is unchanged.
- The arbiter keys resolve from `shed.yaml` with module overrides; an existing `loom.yaml` value keeps its meaning for loom and, through the copy, for darn.
- Every `board_tidy_every` landings dobby spawns one tidy agent, counted once per landing across a restart and for a landing a hand pass made, and posts its report path to the nearest orch; 0 spawns none.
- Finalize's config notice is silent for a comments-only change and for a key the parent already holds, and names base, tip and parent values otherwise, `absent` included.
- A pair teardown of a landed beam ends its dobby.
- Dobby wakes at most once a second and backs off while it passes no run, halted runs only included, and falls back to its timer when fswatch fails to open.

## PATTERN changes

- `PATTERN-dobby-starts-runs` — adds: Starting or driving a board task's run: its parent's dobby, from an empty status, in batten passes; a hand `lyx batten run` is one pass.
- `PATTERN-owner-machine` — changes: a continue pass dobby schedules into a run it already drives is not a resume and needs no origin verdict; the first pass of a run after a dobby start or a machine wake, every pass while a full pass is owed, and every hand pass, is a resume and gets one.
- `PATTERN-shed-stop-chain` — adds: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; arbiter keys in shed.yaml with module overrides.
