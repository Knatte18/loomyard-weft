# Decision record — logging-cleanup

## Goal

Bring lyx's logs to the level policy in `internal/logger/logger.go` ("Level policy"):
an ordinary state is logged at Info or Debug once per change, nothing is logged per tick or per poll, and no test leaks WARN lines into verify logs.
Fix the reed resize re-apply failure that produces most of today's log volume.
Keep a torn-down pair's logs in a git-ignored hub archive with a retention period.
Send the orch a batten notice for every landing.

## Scope

In:

- **reed-resize-pane-count**: the hub watchdog daemon retires and respawns itself when the `lyx` binary it was started from is replaced (D1, D2, D18).
  The resize re-apply failure logs once per failure episode (D3).
- **log-level-hygiene**: the call sites the board inventory names (D3 to D10), an audit of every Info-or-above log line, Warn included, in a loop or polled path (D11), and the `Level policy` text with a new PATTERN entry (D12).
- **Test noise**: the logger discards its stderr half under `go test` (D13).
- **run-log-archive**: pair teardown copies the pair's trace logs and verify logs into a hub archive and prunes entries older than 30 days (D14, D15).
- **batten-land-notice**: every landing sends `<slug>: landed`, with `; newly ready: …` appended when a task became ready, and the orch stencil handles it (D16, D17).

Out:

- `gitrepo: push refused, retrying`: its level fits the policy (a recoverable retry), and its cause is `run-window-bugs`' `weft-push-refused-retry` section.
- A configurable trace-file level (`logger.yaml` `level`): `spawn-audit-log`'s `logger-file-level` section.
- Retiring other long-lived processes on a binary change (orch watcher, batten loops): only the reed watchdog is shown to misbehave.
- Archiving the prime's batten per-slug logs (`<prime>/.lyx/batten/*.log`): the prime is never torn down, so they survive already.
- Archiving shuttle run directories (`.lyx/shuttle/<run>/`).

## Decisions

### D1 — The resize failure's cause is a stale watchdog binary

The hub watchdog daemon (`lyx reed watchdog`) keeps running the binary it was started from after a deploy replaces that binary on disk.
The running daemon predates `AnchorTop`, so its `render.partitionByAnchor` leaves the top-anchored `loom-status` strand out of the layout string.
tmux then refuses a layout that names fewer panes than the window holds.
The current render code places `AnchorTop` strands, and reed's recorded pane ids match the live window, so neither render nor reconcile changes.

### D2 — The watchdog retires on a binary change and respawns itself

At startup the daemon records the identity of the file at its own executable path (`os.Executable()`), as the file's identity and modification time.
On its existing discovery tick it re-stats that path.
When the identity differs, it:

1. stops its loops and releases the hub watchdog lock;
2. calls `reedengine.SpawnWatchdog` with its own `--hub-path`, `--tmux` and `--shell` values and a suppress value computed at that call site, the same not-under-`go test` guard every other caller passes;
3. logs the retirement at Info and exits 0.

The fresh daemon runs the new binary, takes the lock, and its layouts include every strand.
Bound: the check adds no wakeup of its own (`PATTERN-daemon-wakeups`), and each daemon process retires at most once.
A deploy that rewrites the file in several steps can chain retirements, one per observed change, and that chain ends when the file stops changing.
The respawn is the existing best-effort detached spawn, logged as `PATTERN-spawn-observability` requires.
A failed stat counts as unchanged and logs at Debug, so a binary missing mid-build never retires the daemon.
A failed respawn leaves the hub with no daemon until the next `reed up`, `attach` or `resume` spawns one, the same recovery that path has today.
The check only retires the process and never touches config, so `PATTERN-binary-change-reconcile` is unaffected.

### D18 — The one live pre-task daemon is ended once by hand

A daemon started before this task lands has no D2 check and holds the hub lock, so every new spawn exits and the failure goes on.
After this task's deploy, the operator ends each running pre-task `lyx reed watchdog` process once.
The next `reed up`, `attach` or `resume` then spawns a daemon that carries D2.
The landing report names this step.
Bound: a one-time act per hub at this deploy; every later deploy is handled by D2.

### D3 — A repeated resize failure logs once per episode

The watch loop logs `reed: resize re-apply failed` at Warn on a session's first failure after a success or after the session's watcher starts.
While the failures go on it logs them at Debug, and on the next success it logs one Info line saying the re-apply recovered, with the count of failed ticks.
The episode state lives in the session's watcher, in memory only.
It ends with the watcher, and no recovery line is owed for a session that ends mid-episode; a respawned daemon starts with no episodes.
Rationale: the policy's hard rule bars Warn inside a loop that repeats without a state change, and this line is most of today's log volume.

### D4 — The watchdog's lock contention exits at Debug

`reed: watchdog lock already held, exiting` moves from Info to Debug.
At the default level the durable sink then writes nothing, and because it opens lazily and a zero exit never forces it open, the short-lived process creates no trace file.
Rationale: losing the lock race is the expected outcome of every `up`, `attach` and `resume` while a daemon runs.

### D5 — Batten waits are Info, once per state; Warn means a failed step

`reportStuck` keeps writing the reason file on every call, but its log line takes its level from the caller's cause:

- Warn: a producer step that failed, such as a Worktree-Create, Seed-Child, Land or Worktree-Teardown failure, or a child status file that cannot be read or is gone.
- Info: an ordinary wait or move, such as the child running, awaiting, paused, blocked or halted for the operator, a state change (`child running → awaiting at PR-Gate`), a requested pause, or a Publish-retry action or wait.

A blocked or halted child is Info because batten itself is not stuck: it is waiting, and the child's stop reaches the orch through its own notice.
The wait report's log dedupe keys on the reason without its changing parts, such as elapsed seconds, a grace countdown or a retried command's error tail.
The wait report writes the reason file whenever the reason text changes and logs only when the dedupe key changes.
Each caller passes that key beside the text, the reason with its counters, countdowns and error tails left out, rather than having the key parsed back out of the formatted text.
A reason whose text only counts up then logs once per state of the child, while the reason file still carries the full current text for `lyx batten status`.
Rationale: most WARN lines in the prime's traces recorded ordinary waits, and the operator ruled them an action log, not warnings.

### D6 — A halted child waiting for the operator is Info

`battenshed: inner shed run halted; waiting for the operator to resume it` moves to Info, still once per halt episode through the existing halt-warned marker.

### D7 — The notice's session-state line is Debug

`battenshed: session state at notice` moves from Info to Debug: it shadows state the notice itself carries.

### D8 — Orch: benign idle-probe combinations agree, and the rest log once per episode

Two state and probe combinations count as agreement:

- a `busy` state with cause `background` beside an idle-looking pane, since the provider shows an empty prompt while a background task runs;
- an `asking` state beside a pane that is not idle, since a question picker shows no empty input box.

A remaining disagreement logs at Info, once per episode, where an episode is keyed on the idle answer, state, cause and the state's `Since`.
A flip back to agreement and then to the same disagreement with the same `Since` does not log again.
Rationale: the line drives no decision, nearly all its occurrences were the two benign combinations, and a disagreement is a diagnostic, not a recoverable failure.

### D9 — Orch hold reasons carry no draft text

Draft text reaches the orch by two routes, and both carry a draft kind apart from their reason text:

- shuttle's idle probe, when its readiness reading is held by a draft;
- shuttle's busy send error (`ErrSessionBusy`), when the box holds text that is not the send's own, or when the send's hold came from a draft reading.

The orch watcher logs and stores the constant reason "the input box holds a draft" for that kind, in `orch: injection held`, `orch: injection held by the session` and `State.Stuck`.
No orch log line and no `State.Stuck` value carries the operator's draft text, and the once-per-change dedupe holds while the operator types.
Loom's `driver_busy` refusal still names the draft, since the operator reads it there to clear the pane.
Rationale: the changing draft defeated the dedupe and wrote what the operator was typing into the trace.

### D10 — Two smaller lines

- `claudeengine: could not read the session name`: a transcript that does not exist yet logs at Debug.
  Any other read failure logs at Warn once per session id for as long as the provider value lives, through a mutex-guarded seen-set on it.
- `battencli: create worktree` drops its `mutations` field, since `fabric: mutation` already logs the same record.

### D11 — Audit every Info-or-above line, Warn included, in a loop or polled path against the policy

The audit checks every Info-or-above line, Warn included, inside a loop or a polled path against the policy.
Each site either stays as it is, moves to Info or Debug, or gains a once-per-change or once-per-episode guard.
The test for a site: Warn only for a recoverable failure (a retry, an unconfirmed teardown, an error swallowed on a fallback path); nothing at Info or above repeats without a state change.
The plan lists every Info-or-above line, Warn included, that sits in a loop or polled path, each with a verdict (keep, demote or guard), and every changed site has its change in a card.
A line at Info or above that sits in no loop or polled path needs no per-site verdict: such a site is out of this audit.
Rationale: the board asks for the other modules' Warn calls to be checked, and the audit narrows that to the Warn calls in a loop or polled path and adds the Info and Error lines there, where an Info-or-above line can repeat without a state change; the inventory ranked only the sites that fired in four days of traces.

### D12 — The policy text and a PATTERN entry

`logger.go`'s `Level policy` gains three rules:

- an ordinary state (a wait, a pause, a halt for the operator, an ordinary retry iteration) is Info at its change and never Warn;
- a failure that repeats each tick logs Warn once per episode, Debug while it persists, and Info at recovery;
- a log line's dedupe key holds no text that changes while the state stays the same.

`PATTERN.md` gains `PATTERN-log-levels`, a one-line entry pointing at that policy, in the same commit.
It goes under `## Daemons and waits`, where the per-tick floods it prevents arise, and it carries no `(test)` tag because review enforces it.
Rationale: spawned agents get `PATTERN.md` inlined and not `logger.go`, which is how the current violations accrued unseen.

### D13 — The logger discards its stderr half under `go test`

Under `testing.Testing()` the logger's stderr half defaults to `io.Discard` instead of `os.Stderr`, the same test-time switch its sink already applies to the durable half.
A test that asserts on log output still calls `logger.SetOutput` itself, and the durable sink is untouched.
The trade-off: a developer running a failing test with `-v` sees no logger lines on stderr unless that test redirects the logger.
Rationale: the default lives in the logger, so it covers every test binary that links it and adds no import to any test kit.

### D14 — Pair teardown archives the pair's logs before removal

`internal/pairteardown`'s composite copies, after the session end and the run-lock wait and right before `Topology.Remove`:

- the pair's trace directory (`logger.LogsDirAt` for the pair's anchor);
- the pair's verify logs, `verify-*.log` in `verifytree.Dir` for the pair.

The archive root is one accessor in `fabricengine`, beside `HubLogsDir`, resolving to `<hub>/_board/.lyx/log-archive`.
The copy and the prune (D15) both resolve the root through it.
Each copy goes to `<root>/<slug>/<UTC timestamp>/`, with the two sources in their own subdirectories.
The timestamp directory is created exclusively; on a name collision the step adds a numeric suffix and never writes into an existing entry.
The step copies and never moves: a teardown the removal then refuses leaves the pair whole, and the next teardown writes a fresh entry.
A pair with neither source creates no archive entry.
A copy failure removes the step's own partial entry, logs Warn, and never fails or delays the teardown.
A completed archive logs one Info line with its path and file count.
Rationale: `pairteardown` is the one route every teardown takes (`PATTERN-pair-teardown`), and after the run-lock wait no step writes into the pair.
The archive is never tracked, so it lives under `.lyx` (`PATTERN-durable-vs-ephemeral-state`), and it sits under `_board`, which is reachable from the hub only (`PATTERN-hub-containment`).

### D15 — The archive keeps 30 days, as a constant

Each archive write first prunes the timestamp directories, in any slug, whose name's timestamp is older than 30 days, then removes any slug directory left empty.
An entry whose name does not parse as an archive timestamp, and any plain file, is left alone.
The retention is a Go constant, not a config key.
A prune failure logs Warn, and an entry another teardown removed first counts as removed.
Rationale: the logs are needed for weeks, and a hub-wide key would take a `HubWide` config module with no one asking to tune it.
Pruning sits outside `package fabricengine`, so `PATTERN-fabric-destruction-chokepoint` does not apply to it.

### D16 — Every landing sends a notice

The Land row's last step queues `<slug>: landed` after the board prune, appending `; newly ready: a, b` when the landing made tasks ready.
It is sent once per batten run, guarded by the row's marker line.
The existing ready marker literal keeps meaning "the landing notice was queued", so a row resumed across the upgrade sends no second notice.
A notice that cannot be queued, because the orch state is unreadable or no orch strand is recorded, logs Warn naming the slug and is not retried: the row is done; Finalize already marked the board entry `done`, and the Land row's prune removed it.
The line is still appended to the hub notice log before the hold-back check, and the orch learns of the landing from D17's board checks and of newly ready tasks from that notice-log line at its next wake.
`origin/prod` is the baseline for the deploy only, never for the landing.
A Land failure still sends its failure notice, and a landing that fails before the ready read sends no landed notice.
Rationale: the alignment agent lands tasks with no orch turn, so the notice is the orch's only prompt that a task landed.

### D17 — The orch stencil handles every landing notice

`contracts/stencils/orch/orch-template-role.md` routes every `landed` notice to `## After landing`, which then:

- treats the task as landed, no longer running;
- checks the board intake;
- starts any tasks the notice names as newly ready;
- runs `update-plugins.sh` by hand when the hub's landing `deploy` key is empty.

A dropped notice is caught at the orch's next wake by two board checks, neither of which depends on the `deploy` key:

- any task the orch counts as running whose board entry is `done` or gone from the board, or whose pair is gone, is treated as landed and no longer running, and the newly-ready check below runs for it;
- for a task the first check finds landed, the orch reads that task's `landed; newly ready:` line in the hub notice log D16 appends to, and starts each task it names that the board still shows ready and that has not started; any other ready task waits for the operator's approval, as the orch's board policy requires.

Bound: the fallback starts only tasks a landing named as newly ready, so it never starts a task the operator has not released.

A separate deploy-recovery check runs at each wake only while the landing `deploy` key is empty, and covers only landings into a `require_pr_to_base` branch: the orch compares `origin/prod`, which `update-plugins.sh` moves to the commit it deploys, against `origin/main`, and runs `update-plugins.sh` when `main` holds a commit `prod` lacks.
All three checks read durable state, so they hold across a refresh, a compact or a handoff of the orch.
The `contracts/stencils/claims_test.go` rows that pin the old sentences change with it, and `docs/overview.md`'s line on Land notifying only on a failure or a newly ready task is updated.

### Added after Discussion (parent, 2026-10-13): D11 audit bounded to the listed and review-named sites

Decision: D11's per-site verdict list is complete when it covers the sites the plan lists plus every site named in the Plan-Review rounds 1-3 reviews (round 3: reed's state-load Warns, shuttle's record readers reached through loop callbacks, battencli's session-state reader, parentlock's release Warn). Per-tick lines found later, below further callees or behind callbacks, are not a plan defect: they are caught from traces after deploy and fixed as follow-ups. A review finding that only names more unlisted sites beyond these is not BLOCKING.

Rationale: Three rounds each added the rows the review named and the next review found more one call level down; an exhaustive static call-graph sweep does not converge through review rounds, while the design itself (policy, verdict per site, guards) is sound and the trace inventory already ranks the sites that fire.

### Added after Discussion (parent, 2026-10-13): Plan-Review hard cap: F1 settled at Webster

Decision: Plan-Bouncer round 4 is accepted unconverged. Round 4's F1 is applied as its review states when card 9 is built: handedLockStep's decision read is not guarded (D11 verdict keep, since awaitingStep's own decision read ends the Call on the same check and the line never repeats), and the stale-base and notice-step decision-read rows of TestInnerRun_ReadFailuresLogOncePerEpisode run with a pause covering the slug so runArm skips awaitingStep. F2 (recovery reached by the strand-read test) is a mechanical test fix taken the same way.

Rationale: At the hard cap the segment can only be accepted. F1 faults one card's guard rows and one site's verdict, not the task's design: D11's per-site guard-or-keep rule stands, and the review spells out the exact fix, so Webster applies it and Webster-Review checks it.

## Constraints

- `PATTERN-shuttle-provider-seam`: the draft kind (D9) is exposed through shuttle's provider-neutral types, and Claude's shape stays in `claudeengine`.
- `PATTERN-test-speed` and `PATTERN-test-economy`: new tests are unit-tier with injected clocks and fakes, extending existing tables where one covers the behavior.
- Docs in the same commit: `reedengine`'s `doc.go` (D2, D3), `pairteardown`'s `doc.go` and `pattern/PATTERN-pair-teardown.md` (D14), `battenshed`'s `doc.go` (D5, D16), `logger.go` (D12) and `docs/overview.md` (D16).
- A package whose new top-level tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- No operator answered; every open board question was settled by the writer, and each pick sits in its decision and in the support log's question ledger.

## Open risks

- `go-driver-assistant` is in Webster and may touch `internal/battenshed/innerrun.go`; whichever lands second merges against the other.
  Its assistant reads logs to find a stop's cause, and if it keys on WARN lines, D5's Info waits fall outside that.
- `run-window-bugs` is in Plan-Write and changes shuttle's readiness and send paths (`shuttle-ready-blank-pane`, `shuttle-color-verified-send`), which D9 also touches.
- The audit (D11) spans many packages, and a later trace may show a site it missed.
- Verify logs were never measured: no pair holds any now, and each pair keeps up to 20, so the archive's size rests on retention alone.
- On Windows a running binary cannot be overwritten in place; D2's check fires on whatever the deploy leaves at the path, or not at all, and Windows is unverified.
- D18 depends on the operator; until it is done, the hub's old daemon keeps failing every tick.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d16-d17-dropped-landing-notice-no-recovery (class design, severity BLOCKING, open in rounds 1, 2, 3, 4)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-4-review.md; fixer report: _lyx/reviews/plan/round-4-fixer-report.md.
  - decision-read-guard-rows-unreachable (class design, severity BLOCKING, open in rounds 4)
  - recovery-branches-unreached-by-tests (class consistency, severity MEDIUM, open in rounds 1, 2, 3, 4)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- A watchdog whose executable path is replaced retires: it releases the lock, spawns one successor through `SpawnWatchdog` and exits 0. One whose stat fails or whose file is unchanged keeps running.
- Under `go test` the retirement spawns nothing.
- After the deploy and D18, the hub watchdog's trace shows no `resize re-apply failed` line for a session holding a `loom-status` strand.
- A session whose re-apply fails every tick logs one Warn, then Debug lines, then one Info on recovery; a second failure episode after the recovery logs Warn again.
- A watchdog start that loses the lock race writes no trace file at the default level.
- `reportStuck` logs Warn for a failed Worktree-Create, Seed-Child, Land or Worktree-Teardown step, and Info for a child state change, a halt, a block, a pause and a Publish-retry wait. The reason file is written in every case.
- A wait reason whose text differs only in an elapsed counter logs once while the child's state is unchanged, and the reason file holds the newest text.
- An idle probe answering idle beside `busy`/`background`, or not idle beside `asking`, logs nothing. A real disagreement logs Info once per episode, including across a flip back to agreement with the same `Since`.
- A held-by-draft probe, or a busy send refused over foreign box text, whose draft changes on every tick logs its held line once. Neither the log lines nor `State.Stuck` contain the draft text.
- Loom's `driver_busy` refusal still names the draft.
- A missing transcript in `SessionNameDrift` logs at Debug; another read error logs Warn once per session id.
- A package test run whose code logs at Warn leaves no logger stderr output, and the logger package's own tests still pass.
- No Info-or-above line the plan listed as in a loop or polled path logs more than once while its state is unchanged (D11).
- Pair teardown copies the pair's trace logs and verify logs into `<root>/<slug>/<timestamp>/` after the run-lock wait and before removal. A pair with neither source creates no entry. A copy failure leaves no partial entry, logs Warn and leaves the teardown's result unchanged. A refused removal leaves the pair's logs in place. Two archives of one slug in the same second land in two entries.
- An archive write removes timestamp entries older than 30 days and empty slug directories, keeps newer entries, leaves entries with unparsable names and plain files alone, and treats an entry already gone as removed.
- A landing with no newly ready task sends `[batten notice] <slug>: landed`; one with ready tasks sends `[batten notice] <slug>: landed; newly ready: a, b`. A Land row resumed after the notice was queued sends no second notice, a notice that cannot be queued logs Warn, and a failed Land step sends only its failure notice.
- The orch stencil's claims test pins that every landing notice goes to After landing.
- The orch stencil's claims test pins the next-wake board check: a running task whose board entry is `done` or gone from the board, or whose pair is gone, is treated as landed.
- The orch stencil's claims test pins that the next-wake fallback starts only the tasks a landed task's notice-log line names, never another ready task.
- The orch stencil's claims test pins the deploy-recovery check, scoped to an empty `deploy` key, including its baseline: `origin/main` holding a commit `origin/prod` lacks.

## Notes for the plan writer

- Draft text enters through `internal/shuttleengine/readiness.go` and `internal/shuttleengine/send.go`.
- `exemptEnd` in `internal/battenshed/innerrun.go` mixes state changes with unreadable-status failures.
- A held-back row notice is `rowNotifier.send` in `internal/battenshed/rownotice.go`.
