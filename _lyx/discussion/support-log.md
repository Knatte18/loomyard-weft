## Interview

### Turn 1 — exploration

Read the board entry `twb-d0-hc` and the board list.
The entries that touch this task are `pause-background-work` (a hub pause leaves background work running, so a hibernate confirmation never passes) and `orch-split` (the orch's future split, which changes nothing here).
`twb-d0-ha` and `twb-d0-hf` are copies of this entry and are not separate work.
Read `internal/tenderengine/doc.go` (pause, resume, status and its session causes), `internal/tendercli/cli.go` (pause flags and the hub-wide scope), `internal/shortcutcli/shortcuts.yaml`, `contracts/recipes/batten-recipe.yaml` (Land under the hub's land lock, then `Worktree-Teardown`), `internal/landingshed/doc.go` (parent locks held from Merge-In through Finalize, including PR-Gate), `internal/orchengine/doc.go` and `notice.go` (`QueueNotice`), and `internal/orchcli/spawn.go` (the detached watcher spawn).
No hibernate code exists at HEAD.
Only one `-LYXHUB` hub exists under the operator's code directory, and lyx has no registry of hubs.

### Turn 2 — design, self-picked

No operator answers in autonomous mode. Every pick is in the question ledger.
Settled on a detached Go watcher that alone runs the hibernate, triggered now or after named runs settle, behind a confirmed hub pause, lifting only its own pause after wake.
While writing, the parent-lock condition in the confirmation was dropped: a run holds its parent lock through the whole PR-Gate wait, so that condition would block a hibernate for as long as a pull request waits.

### Turn 3 — lens review

Six forks returned reports: premises, goal-scope, board-fit, pattern-fit, scenarios and leanness. Every fork returned a report. Duplicates are merged and listed once.

- premises+scenarios, BLOCKING/MEDIUM: tender status does not report the land lock (`tenderengine.StatusReport` has parent-lock holds only). Fixed as proposed: D5 reads battenshed's land lock through its own seam.
- premises+scenarios, MEDIUM: a pause halts a Land waiting on the land lock (`internal/battenshed/doc.go`), and a resume does not restart it. Fixed differently: the proposed change was only to report it, but the watcher now also waits for the land lock before pausing (D5), and D7's notice names any run blocked at Land with its way back. Acceptance lines were added.
- premises, MEDIUM: a deploy can outlast the 10-minute bound. Fixed as proposed: D5 waits for the land lock first, bounded at 35 minutes, longer than the 30-minute deploy timeout.
- premises+scenarios, MEDIUM: the settled states were wrong. Shed has no `halted` state, and a halted child leaves batten `running` (`internal/battenshed/doc.go`). Fixed as proposed: D4 reads the child's state too and names `blocked`, `failed`, `paused` and `awaiting`, and a pre-paused run is stated to hold the request.
- premises+pattern-fit, MEDIUM/LOW: the watcher has no strand, so its pause caller is `EXTERNAL` (`internal/pauserecord/record.go`), and no single-entry resume exists. Fixed as proposed: D5 states `EXTERNAL` and keeps entry ownership in the request record, and Scope adds the `tenderengine` single-entry resume.
- premises+goal-scope+scenarios, LOW/MEDIUM: wake detection assumed `systemctl` returns first, and `pauserecord.SleepWatch` already exists. Fixed as proposed: D6 uses `SleepWatch` from the command's start, and the window assumption is in Open risks.
- board-fit, MEDIUM: a driver can stall after a host sleep (`orch-split`). Fixed as proposed: an Open risk was added, and the notice names `lyx tender revive`.
- board-fit, MEDIUM: the orch's turn withholds with no orch watcher live, and the relation to `pause-background-work` was not explicit. Fixed as proposed: D5 states the orch case and its bound, and Scope Out states the split with that entry.
- board-fit, LOW: name `creel` and `spawn-audit-log` as later owners. Fixed as proposed, in Notes.
- goal-scope, MEDIUM: no deny of the sleep commands exists at HEAD (`internal/shuttleengine/claudeengine/settings.go` `standingDenies` has none), and the orch is not pointed at the verb. Fixed as proposed: D13 and the Scope lines were added, and D1's false claim was removed.
- goal-scope, MEDIUM: "every run in flight" was missing from the record. Fixed as proposed: `--all` was added to D2, with the late-run bound.
- goal-scope, LOW: the departure from "batten fires it" was unstated. Fixed as proposed, in D3.
- goal-scope, LOW: counting a halted run widens "lands". Fixed as proposed: D4 states the widening.
- pattern-fit, MEDIUM: the watcher's wait must join the daemon-wakeups guard list. Fixed as proposed, in D4.
- pattern-fit, MEDIUM: the non-Linux way forward was a denied step. Fixed as proposed: D9 addresses it to the operator.
- pattern-fit, LOW: `cliwire` was the wrong entry to cite. Fixed as proposed: Constraints cites `PATTERN-cwd-resolution`.
- pattern-fit, LOW: injected spawner in tests. Fixed as proposed, in D2.
- pattern-fit, LOW: the guard for the new PATTERN entry. Fixed as proposed: D3 makes it test-guarded, with a scan for `systemctl hibernate`.
- pattern-fit, LOW: the clock seam must yield wall and awake readings. Fixed differently: the proposal was to name a two-reading seam in D6, but D6 uses `SleepWatch`, whose awake clock is already injectable.
- pattern-fit, LOW: share `hubDeps`. Fixed as proposed, in Notes.
- pattern-fit, LOW: test budget. Fixed as proposed, in Notes.
- scenarios, MEDIUM: a crash or a cold boot after the pause leaves the hub paused silently. Fixed as proposed: an Open risk names where it shows and how it is lifted.
- scenarios, MEDIUM: two racing requests. Fixed as proposed: `request` writes under the request's lock with a request id, and `watch` refuses a mismatched id (D1, D2).
- scenarios, MEDIUM: a cancel or an outside resume could land just before the command. Fixed as proposed: D6 re-reads under the lock, and D8 serialises cancel on the `hibernating` phase.
- scenarios, LOW: the `watch` bound. Fixed as proposed, in D1.
- scenarios, LOW: duplicate, already-settled and done slugs. Fixed as proposed, in D2 and the acceptance criteria.
- scenarios, LOW: notice delivery under a live pause. Fixed as proposed: D7 states that the orch watcher holds notices under a hub entry (`internal/orchengine/doc.go`, Hub pause).
- leanness, LOW: D5's duplicate of Scope Out, the doc list repeated in Scope and D12, Constraints bullets repeating Decisions, Auto-mode assumptions repeating Decisions, D10's deliberation, and Notes restating D5/D6. Fixed as proposed for each.
- leanness, LOW: the verify-timeout risk was unbacked. Fixed as proposed: `internal/verifytree/verify.go` uses `context.WithTimeout`, whose runtime clock does not count suspend, so the risk was removed.

## Rejected alternatives

- **Batten's Land row fires the hibernate** (the board entry's "who fires it"). Rejected: it puts a machine-wide action inside one run's row, cannot serve a "now" request or several slugs, and makes the run that fires it the one mid-write when the machine goes down. The watcher waits for `Worktree-Teardown` to finish, which covers "after Land and its deploy".
- **The orch runs `systemctl hibernate`.** Rejected by the operator's 2026-10-14 direction: the session is left hanging mid-call.
- **`request` blocks until the hibernate.** Rejected: the orch's turn would stay open across the hibernate, which is the failure the direction rules out.
- **A machine-wide flag outside any hub, covering every hub.** Rejected: there is no hub registry to find the hubs to pause, and the operator runs one hub per machine.
- **Background work withholds confirmation until it ends.** Rejected: it is what made the request fail in `pause-background-work`, and the operator already hibernates with runs live.
- **Settled means landed only.** Rejected: a halted run would keep the machine up indefinitely.
- **Always resume the hub after wake.** Rejected: it would lift a pause the operator set on purpose.
- **Leave the hub paused after wake.** Rejected: every run would stay stalled until someone noticed.
- **Express "every run in flight" by naming each run.** Rejected after the lens review: the orch could miss a run, so `--all` takes them at request time.
- **A wall-versus-monotonic wake rule of the module's own.** Rejected: `pauserecord.SleepWatch` already defines a machine sleep.
- **Parent locks withhold confirmation.** Rejected: a run holds its parent lock through the whole PR-Gate wait.
- **A `hibernate` config module for the bounds.** Rejected under YAGNI: nobody asked to tune them.
- **No shortcuts, or longer names.** `p` and `r` follow the `a` beside `attach` pattern and are free.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Who runs the hibernate? A (auto-pick): only `hibernateengine`'s detached watcher, pinned by a new PATTERN entry.
- Q: Which triggers? A (auto-pick): now, after one or more `--after <slug>` batten runs settle, or `--all` for every run not settled at request time.
- Q: When has a run settled? A (auto-pick): batten done at `Worktree-Teardown`, or batten or its child `blocked` or `failed`. A run that is `running`, `paused` or `awaiting` has not.
- Q: What confirms the pause? A (auto-pick): no session busy with cause `turn`, no network operation in flight, and no land lock held. Background work and parent locks do not withhold. The bound is 10 minutes, after a land-lock wait of at most 35 minutes before the pause.
- Q: Add a deny of the machine sleep commands? A (auto-pick): yes, a standing guardrail row for every run, since none exists at HEAD and the board entry's incident was the orch running one.
- Q: Machine-wide or hub-wide? A (auto-pick): hub-wide, since there is no hub registry and one hub per machine. This departs from the board entry's wording, and D10 says so.
- Q: Resume after wake? A (auto-pick): only the hub entry the watcher's own pause added.
- Q: How is a wake detected? A (auto-pick): `pauserecord.SleepWatch` from the command's start, with a 2-minute window after a zero exit.
- Q: Add the pause and resume shortcuts, and under which names? A (auto-pick): yes, `p` and `r`.
- Q: Where can the verbs run? A (auto-pick): only from the prime, as `orch` and `batten` do.
- Q: Non-Linux? A (auto-pick): `request` refuses; `status` and `cancel` work.
