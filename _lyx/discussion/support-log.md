# Support log — orch-context-overview

## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

- Read the board entry and the related entries `orch-split`, `viewer-off-shell`, `reed-status-buttons`, `run-pane-timing` and `orch-any-worktree`.
  None plans an orch line in the overview; `orch-split` may later split the orch in two, which would then want two lines, but designing for that now is YAGNI.
- Read `internal/battencli/overview.go`, `internal/orchengine/state.go`, `internal/orchcli/status.go`, `internal/orchcli/paths.go`, `internal/orchcli/runs.go`, the orch config's threshold accessors, `internal/state`'s read and write paths and `statuspane.StatusCache`.
- Found that `internal/battencli` already imports `orchcli` and `orchengine` (`wire.go`'s `orchStrandRecorded` and the notice queue), which settles the board entry's dependency question.
- Found that `orchengine.LoadState` takes a read lock and creates the state directory, which the read-only overview should not do.

### Turn 2 — design questions settled

- Layering, line content, lockless read, config load timing, empty and failure states: see ledger Q1–Q6.

### Turn 3 — lens review by forks

Every lens fork returned a report.

- goal-scope, `watcher down` is unrequested: rejected, D4 names it as the one extra cell with its reason.
- goal-scope + premises, first-line consumer risk can be settled: fixed, Open risks now names only the render tests.
- goal-scope, D3 bound softer than reality (runs strand replaced only on build change or dead strand): fixed, D3's bound restated.
- goal-scope, name that `docs/overview.md` is untouched: fixed, added to Scope Out.
- board-fit, `run-pane-timing` edits `overview.go` concurrently: fixed, Open risk added and run-line criteria now read "as the parent renders them".
- board-fit, `orch-token-spend` from `orch-split` unchecked: rejected, the slug is not on the board and the board entry states `lyx orch status` is the only display today.
- board-fit, orch assumed in the prime vs `orch-any-worktree`: fixed, Auto-mode assumption and Open risk added.
- premises + scenarios, `watcher down` misses a hard-killed watcher: fixed, D4 bounds the marker to watcher-recorded exits and Open risks names the gap; a lockless liveness signal does not exist without new writes.
- scenarios, one-strike `state unreadable` vs the run lines' two-strike tolerance: fixed, D2 adopts the `StatusCache` tolerance and a criterion pins it.
- scenarios, stopped orch state unnamed: fixed, D4 rule added.
- scenarios, state present with no runs: fixed, criterion added.
- scenarios, mid-cycle reading next to the new phase: fixed, one clause in D4.
- pattern-fit, config failure log level unstated: fixed, D3 says Warn.
- pattern-fit, pin the lockless read to `orchengine` not `internal/state`: fixed, D2 says so.
- pattern-fit, docs rule not cited in Constraints: rejected, Scope carries the doc updates.
- leanness, D1 and D3 rationale argue rejected alternatives: fixed, trimmed.
- leanness, D2 `LoadState` clause: kept, it tells the plan writer not to reuse `LoadState`.
- leanness, D4 padding note belongs in the rules list: fixed, moved.
- leanness, Auto-mode threshold assumption repeats D3: fixed, cut.
- leanness, `PATTERN-daemon-wakeups` and "reads files only" constraint lines repeat: fixed, cut.
- leanness, test-budget note repeats CLAUDE.md: rejected, it names the packages this task is likely to grow.

## Rejected alternatives

- A provider the orch module registers with batten's overview: no layer is crossed today, so a registry adds indirection with no gain.
- A composed view in the runs strand (a second pane process for the orch line): two processes for one window, and the runs strand's spec would change.
- Reading the orch state through `orchengine.LoadState`: takes a lock and creates files from a pane that must write nothing.
- Decoding the orch state file inside `battencli`: a second owner of the state's shape.
- Loading the orch config on every frame: repeats `LoadConfig`'s below-floor warning per frame, while the watcher it describes loads once anyway.
- Refusing the overview on an orch config error: takes the whole runs view down over a display detail.
- Inferring watcher liveness from the state file's modification time: an idle watcher may not rewrite the file, so the age is no liveness signal.
- Showing `orch  state unreadable` on the first failed read: the run lines in the same frame tolerate one failure, so the orch line would flicker where they do not.
- Showing strand liveness through reed or watcher liveness through `orchengine.WatcherLive`: a reed query or a lock probe per frame; `WatcherExit` from the state already says the watcher is down.
- Padding the orch line into the run columns: its cells differ in kind from the run cells.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): does batten's overview reading orch state cross a layer? No: `battencli` already imports `orchcli` and `orchengine`; read directly (D1).
- Q2 (resolved, auto-pick): how is the state read? A new lockless, non-creating reader in `orchengine`, safe because `internal/state` writes atomically (D2).
- Q3 (resolved, auto-pick): where do thresholds come from and when? `orchengine.LoadConfig` once at overview start; failure degrades to `-` (D3).
- Q4 (resolved, auto-pick): what does the line show? Tokens, soft and hard thresholds, phase, cycle count, plus `watcher down` when `WatcherExit` is set (D4).
- Q5 (resolved, auto-pick): empty state? No state file draws no orch line; an unknown reading shows `-` (D4).
- Q6 (resolved, auto-pick): placement? First line of the frame, above the runs (Auto-mode assumptions).
