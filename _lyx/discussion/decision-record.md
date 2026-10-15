# Decision record — orch-context-overview

## Goal

The prime's overview window, drawn by `lyx batten overview` in the `runs` strand, shows one line for the orch session: its latest context reading against the soft and hard thresholds, the watcher's cycle phase and the cycle count.
The operator then sees the orch's context size beside the batten runs without running `lyx orch status`.

## Scope

In:

- One orch line in the batten overview frame, above the run lines, drawn on every frame whether or not runs are in flight.
- A lockless read of the orch state file, exposed by `internal/orchengine`.
- The orch config's thresholds, loaded by the overview verb.
- The overview verb's help text and the orch package doc's mention of the overview window, updated to name the orch line.

Out:

- `lyx orch status`, the watcher, the orch state's fields and the orch config: unchanged.
- Any statusline, reed status bar or tmux change (`reed-status-buttons` owns status-bar actions).
- Any change to the run lines' columns or ordering.
- `docs/overview.md`: no module or execution-stack change.

## Decisions

### D1 — The orch line lives in `internal/battencli`'s overview, reading orch files directly

Decision: the overview source reads the orch state through `orchcli.PrimePaths` and `orchengine`, as `battencli`'s wiring already does for the notice queue and `orchStrandRecorded`.
Rationale: `battencli` already imports `orchcli` and `orchengine`, so the read adds no import edge and crosses no new layer.

### D2 — The orch state is read without its lock, through a new `orchengine` reader

Decision: `orchengine` gains a reader that decodes the state file at `Paths.StatePath` without taking `StateLockPath` and without creating any directory or file; an absent file reads as not found.
A present file that fails to decode is a failed read, counted by the same tolerance as a failed stat or read.
The overview reads it once per frame with the same failure tolerance as the run lines' `statuspane.StatusCache`: a first failed read keeps the last good state (none drawn when there was none), and only a second failure in a row, or a repeat of the same failed modification time, draws `orch  state unreadable`.
Rationale: the overview reads files only, writes nothing and never blocks the process it watches.
`LoadState` and `state.ReadJSONNoMkdir` both take a read lock, which creates the lock file, so neither fits the pane.
`internal/state` writes atomically, so a lockless read never sees a torn file; the two-strike tolerance covers a transient stat or read failure during the rename, as it does for the run lines.
Keeping the decode in `orchengine` leaves the orch state with one owner of its shape, and leaves `internal/state`'s locking contract untouched.
Bound: the reader is read-only and changes no locking of any writer.

### D3 — Thresholds come from the orch config, loaded once when the overview starts

Decision: the overview verb loads `orchengine.LoadConfig(fabricengine.ConfigDirs(location), "orch")` once at start and draws `SoftThreshold()` and `Threshold()` on every frame.
A config load failure does not refuse the verb: it is logged at Warn, and the orch line draws the context reading with `-` for both thresholds.
Rationale: refusing would take the whole runs view down over a display detail.
Bound: the overview's thresholds refresh only when the runs strand is replaced, which `lyx orch start` does when the strand is dead or its sidecar does not record it as launched by the running build; a threshold edit alone does not show until then.
The watcher, which reloads on its own restart, may act on a newer value meanwhile.

### D4 — The orch line's content and form

Decision: one line, first in the frame, of the form `orch  <tokens> / soft <soft> / hard <hard>  <phase>  cycles <n>`, with these rules:

- Token counts render in thousands with a `k` suffix, rounded down (`212k`); a count under 1000 renders as the plain number.
- An unknown reading (`LastContextKnown` false, as after a fresh launch before the first turn end) renders the tokens as `-`.
- The phase is the persisted `Phase` value as it is (`idle`, `compacting` and the rest); the tokens are the persisted reading as it is, which during a non-idle phase may predate the cycle; the phase cell is what tells the operator a cycle is in progress.
- When the state's `WatcherExit` is non-empty, the line appends `watcher down`.
  This covers a watcher that exited through its own code: a signal, the tick-failure cap, or a gone strand, which includes `lyx orch stop`.
  A watcher killed outright (SIGKILL, a host crash) records no exit, so the line then shows the last reading with no marker.
- After `lyx orch stop`, the state file remains, so the line keeps the stopped session's last reading, with `watcher down`.
- An absent state file draws no orch line, as before any orch has started in the prime.
- The orch line is not padded into the run columns.

The run lines below it are rendered as the parent renders them at Plan time, including `no runs in flight`.
Rationale: the board entry asks for tokens against both thresholds plus phase and cycle count; `watcher down` is the one extra cell, kept so a cleanly stopped watcher's stale reading does not look current.

## Constraints

- `PATTERN-told-geometry`: `orchengine` stays told its paths; the new reader takes `Paths` and derives nothing.
  `battencli` gets the paths from `orchcli.PrimePaths`, as its wiring already does.
- `PATTERN-tolerant-records`: the lockless reader decodes into `orchengine.State` and never refuses an unknown key.
- `PATTERN-config-strictness`: `orchengine` is a degrading caller (`LoadOrTemplate`); the overview adopts `LoadConfig` as is.
- `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer`: no new file is created; paths come from `orchcli.PrimePaths`.
- `PATTERN-log-levels`: a per-frame read failure logs at Debug, as the overview's other per-frame failures do.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing overview render tests; pure render tests with an injected clock.

## Auto-mode assumptions

- The orch line goes above the run lines, not below: the orch is one fixed line and the runs list grows downward.
- `watcher down` is shown; `stuck`, `last_abort_reason` and strand liveness are not, to keep one short line.
- No colour or highlight marks a reading past the soft threshold; the numbers side by side are enough.
- The orch line assumes the one orch in the prime, which both `lyx orch` and `lyx batten` require today.

## Open risks

- `run-pane-timing` (running, at Webster) also edits `internal/battencli/overview.go`: it numbers the stage cells of the run lines.
  Whichever lands second rebases on the other; the plan builds on the `renderOverview` on the parent at Plan time, and leaves the run lines as that version renders them.
- The existing overview render tests expect `no runs in flight` as the whole frame; only frames with an orch state gain a line, so they hold, and new cases cover the orch line.
- `orch-any-worktree` would let an orch live outside the prime; the orch line would then need the home worktree's orch paths.
- A hub whose prime never ran `lyx orch start` has no state file and so no orch line; this is the intended empty state.
- A watcher killed outright leaves a stale reading unmarked (D4).

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - testtiming-keep-directive-detached (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- With an orch state holding a known reading, the overview frame's first line shows the tokens, both thresholds, the phase and the cycle count, and the run lines follow as the parent renders them.
- With an orch state and no run in flight, the frame is the orch line, then `no runs in flight`.
- With `LastContextKnown` false, the tokens cell shows `-`.
- With `WatcherExit` set, the line ends in `watcher down`.
- With no orch state file, the frame equals the parent's frame with no orch line.
- One failed state read, a decode failure of a present file included, after a good one keeps drawing the last good orch line; a second failure in a row draws `orch  state unreadable`, and the run lines still draw.
- With a config load failure, the thresholds show `-` and the verb still runs.
- Reading the orch state for the overview creates no file or directory (no lock file, no state directory) and takes no lock.
- Token formatting: counts under 1000 as plain numbers, larger ones in rounded-down thousands with `k`.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass.

## Notes for the plan writer

- `internal/battencli/overview.go` holds the render (`renderOverview`) and the source (`overviewSource.frame`); the orch line fits as a prefix to the frame's lines.
- `internal/battencli/wire.go`'s `orchStrandRecorded` shows the existing `orchcli.PrimePaths` plus `orchengine` read.
- `internal/orchcli/status.go` already renders the same fields as JSON; `internal/orchengine/config.go` holds `Threshold` and `SoftThreshold`.
- If new top-level tests push `internal/battencli` or `internal/orchengine` past their row in `cmd/lyx/testdata/test-budget.yaml`, list that file under `Edit:`.
