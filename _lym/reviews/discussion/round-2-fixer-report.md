# Round 2 fixer-report — discussion

Every finding's premise was checked against the code at HEAD before its fix; all thirteen hold.

## Changes

- F1 (MEDIUM), fixed as proposed: Decision 3 splits `placeDobby`'s refusals.
  One made after `lymcwd.ResolveWorktree` succeeds appends its line to the resolved worktree's `.lym/beam/dobby.log`, derived through one `hubgeom` function `DobbyGeometry` shares, and posts the failed-re-exec notice on a re-exec.
  Only `filepath.Abs` and `ResolveWorktree` failures keep stdout as their only trace; Open risks narrowed and an Acceptance criterion added.
  Premise checked at `internal/beamcli/cli.go` `placeDobby` and `internal/hubgeom/dobbygeom.go`.
- F2 (MEDIUM), fixed as proposed and then refined by the self-check: Decision 2 makes `recordStart` use the start-ticks test, `tick`'s pid refill writes the ticks, and a ticks-less record is back-filled.
  Criteria added for a pid-matching, ticks-differing fresh dobby and for the back-fill.
  Premise checked at `internal/dobbyengine/loop.go` `recordStart` and `tick`.
- F3 (MEDIUM), fixed as proposed: Decision 5 has `runDobby` reopen fd 1 and 2 onto `dobby.out` after `placeDobby` returns, where `selfexec` execs; criterion added.
- F4 (MEDIUM), fixed as proposed: Decision 1 sends the worktree-gone line to the prime's `dobby.log` through a new told Geometry path, appended without rotating, and no append creates a missing directory; Decision 3's line list and the criterion follow.
- F5 (MEDIUM), fixed as proposed: Decision 3 posts a fatal `Run` error after a re-exec through `Deps.Notice`, and keeps a line only when the state read itself failed; the exit-path criterion extended.
- F6 (MEDIUM), fixed as proposed: Decision 10 spawns the check with stdout and stderr on `check.out`; named in Decision 11, the durable-vs-ephemeral constraint and the second-check criterion.
- F7 (LOW), fixed as proposed: Decision 10's did-not-start warning names `lym fabric reconcile` from the prime before `lym gate test --gate adopt`; criterion extended.
- F8 (LOW), fixed differently: Decision 8 empties `LandedSinceTidy` once, at the first count on this binary with no tidy in flight, and sets a tolerant flag; criterion added.
  Not made: the proposed drop, at every count, of each slug the machine-filtered landings no longer return.
  `removeRunRecords` (`internal/battencli/cleanup.go:277`) removes a run's directory, so that drop would also lose this machine's own landings cleaned up before their tidy.
  Every pre-change entry carries no machine, so the one-time emptying gives the same first-tidy result without that loss; the bound is stated.
- F9 (LOW), fixed as proposed: Decision 1's worktree-gone exit stops a tidy in flight through `TidyShuttle.StopStrand` from the in-memory `State.Tidy`, and Decision 4 points to it.
- F10 (LOW), fixed as proposed (the second option), then split by platform in the self-check: the lock holder rotates `dobby.out` where `selfexec` execs, and the spawner no longer rotates it there.
- F11 (LOW), fixed as proposed: Decision 2 has a refused re-exec reread the record and post nothing when another pid holds it; the refused criterion extended.
- F12 (NIT), fixed as proposed: Decision 10's bound says either envelope's pid can name the losing check.
- F13 (NIT), fixed as proposed, then corrected by the self-check: Decision 6's rationale names the heartbeat bound.
- `support-log.md`: a Round 2 entry under Review rounds and the orch-notice ledger row brought in line with Decision 3.

## Self-check

- fix-consistency, MEDIUM, a refused `placeDobby` keeps inherited fds on `dobby.log`: fixed as proposed; Decision 3 states the stdout trace per spawn origin, and the criterion is scoped to a process this binary spawned.
- fix-consistency, LOW, Constraints miss the prime log path: fixed as proposed.
- fix-consistency, LOW, the ledger row on orch notices is stale: fixed as proposed.
- fix-consistency, LOW, Decision 3's refused-re-exec post ignores the second read: fixed as proposed.
- fix-consistency, LOW, the worktree-gone writer is outside Decision 5's rotation rule: fixed as proposed; the rule now reads "the holder of a file's own dobby lock".
- fix-way-forward MEDIUM and fix-proposal MEDIUM, merged, Windows holds `dobby.out` open, so the holder's rotation always fails: fixed as proposed; on Windows the spawner rotates only when `Live` reads the lock free, the two-spawner race is a stated bound, and the criterion follows.
- fix-way-forward, MEDIUM, the tidy stop's runner is built over the removed worktree: fixed as proposed; the runner is built over the prime, `StopStrand` removes a strand with no run record (`internal/shuttleengine/stop.go`), and a failed stop names `lym reed remove <guid>`.
- fix-way-forward, MEDIUM, the fd-reopen criterion cannot run at tier0: fixed as proposed; it rides the tier1 binary-replacing test, and the test-speed constraint says so.
- fix-claims, MEDIUM, `lym fabric reconcile` may not run the gate migration: disputed in part, premise false; `internal/gateslot/doc.go:78` and `internal/gateslot/config.go:344` name `lym fabric reconcile` from the prime as the step that runs `MigrateConfig`.
  The loose "no reconcile ran" was fixed to "no reconcile of the hub's config ran", since `lym adopt` does run the origin reconcile.
- fix-claims LOW and fix-proposal LOW, merged, eleven minutes holds only while `notice_probe_s` is at most 60: fixed as proposed; the gap is the larger of the idle ceiling and the probe interval.
- fix-proposal, LOW, a ticks-less record read as a re-exec skips a pid-reusing reboot's full pass: fixed differently; the ticks-less reading now applies to the lock wait only, `recordStart` gives such a record a full pass, and a full pass after a true re-exec is only extra work.

## Disputed

- Self-check fix-claims MEDIUM, `lym fabric reconcile` running the gate migration: premise false, evidence above. No review finding was disputed.

## Deferred

None.
