---
format: 5
approved: true
language: go
---

# Plan: daemon-wakeups

lyx's long-lived processes wait cheaply and wake on time.
Every unbounded wait blocks on a file event through a new `internal/fswatch` leaf where its substrate offers one and otherwise polls at a second-scale, backing-off cadence pinned by a guard; the reed watchdog's three loops convert first.
The orch's resume pointer rides the provider's session-start hook after every compaction, and the orch watcher gates every injection on a readiness reading built from the hook-derived session state instead of the pane probe, logging why each step waited.
The run seed carries its start time, which the spawn order reads in place of the file's mtime.

## Card Index

1 — fswatch-leaf — the `internal/fswatch` leaf over fsnotify, its dependency, leaf PATTERN entry and docs
2 — poll-floors — orch, shuttle and batten floor a configured poll interval at one second; shuttle's template default moves to 1000 ms
3 — seed-started-at — the seed carries `started_at`, stamped by `WriteSeed`, and `hubgeom` reads it for the spawn order
4 — reed-resize-events — the resize loop's signal mode waits on `fswatch` through an opener seam, with one-shot debounce and retry timers
5 — reed-wake-cadence — the shared backoff rule, the discover signal touched at a cold boot and told on the geometry, and the name-repair backoff
6 — watchdog-discovery-backoff — the discovery loop backs off across unchanged cycles and wakes on the discover signal
7 — session-start-signal — the spec's context-after-compaction command, the Claude `SessionStart` hook, the `session_start` signal and the hook's context JSON
8 — readiness-reading — one readiness reading over the session state gates `Runner.SessionIdle` and the verified send's idle wait
9 — orch-held-injection-log — the watcher records and logs why each injection waited, when it was typed and when a boundary was found
10 — shell-changedir — the shell seam gains `ChangeDir`
11 — orch-resume-context-engine — the pointer render, the delivery mark, the atomic role render and the watcher's skip of a hook-delivered pointer
12 — orch-resume-context-verb — the hidden `lyx orch resume-context` verb and the hook command on the orch spec
13 — daemon-wakeups-guard — `PATTERN-daemon-wakeups`, its background file, the literal and ticker guard, and the one-shot annotation
14 — idle-cpu-benchmark — the before-and-after idle CPU report under `docs/benchmarks/`

## Shared Decisions

### Decision: floors-at-load-and-in-the-accessor

- **Decision:** Each floored interval is floored in two places: the loader logs the one Warn naming the key and the floor, and the accessor or constructor floors silently, so a loop that reads the accessor every tick logs once.
  Batten's floor sits in `NewInnerRun` on the told duration, since the recipe's `poll_interval_s` is in whole seconds and an absent key and an explicit zero are indistinguishable there; the recipe keeps resolving both to its default without a warning.
- **Rationale:** D9 asks for one Warn and a floor that never refuses; a Warn in an accessor read per tick would repeat forever.
- **Applies to:** card 2

### Decision: fake-clock-not-fast-config

- **Decision:** A test that drove a wait loop fast through a sub-second `PollIntervalMS` now injects a fake clock; `newFixture` in `shuttleengine` defaults to one, and a test that needs the wall clock says so.
- **Rationale:** D9's bound: a test that wants a fast loop injects a clock or a timing struct, never a config value; the floor would otherwise turn millisecond fixtures into one-second real sleeps.
- **Applies to:** card 2

### Decision: opener-seam-exported-from-reedengine

- **Decision:** The file-event source both daemon loops wait on is a `reedengine.FileWatchOpener`, with `reedengine.OpenFileWatch` wrapping `fswatch.Watch`; `reedcli`'s production files import neither `fswatch` nor fsnotify.
`reedengine`'s tier-1 tests pass an opener over a channel they feed; `reedcli`'s loop lists sessions against a live tmux server, so its event branch is proven in tier 1 at its pure seams (the wake planner and the signal's consume) and live in the `tmux` tier with the production opener.
  `reedengine.NextWakeCadence` is the one backoff rule, and `reedengine.DiscoverSignalFileName` the one spelling of the discover signal's name, which `hubgeom` joins onto `fabricengine.HubScratchDir` for the geometry and `reedcli` joins onto the same for the daemon.
- **Rationale:** `PATTERN-told-geometry` keeps `reedengine` told its paths; one declarer per rule and per literal keeps the two packages from drifting.
- **Applies to:** cards 4, 5, 6

### Decision: readiness-tested-in-shuttle

- **Decision:** The readiness reading is computed inside `shuttleengine` and reaches the orch watcher through `IdleProbe`, so it is tested against a fake reed and engine in `shuttleengine`; the orch watcher's fake `Session` scripts the `IdleProbe` the reading yields, with its reason, and the #481 scenario is a `Runner.SessionIdle` test (a hook turn end beside a pane needle reads idle) plus a watcher row whose probe reads idle on the tick after the boundary.
- **Rationale:** D7 places the reading behind `Runner.SessionIdle` and keeps `Watcher.probeIdle` the one door; the orch fake cannot show the move from pane to state, the shuttle test can.
- **Applies to:** cards 8, 9

### Decision: hook-command-is-posix

- **Decision:** The context-after-compaction command is built with `shell.Posix()`, whatever the host, because Claude Code runs its hooks under a POSIX shell on every OS, as the recording hooks in `settings.go` already assume; it chains `ChangeDir` to the prime's anchor with the bare verb, and the verb's own non-prime refusal bounds a failed change of directory.
- **Rationale:** D11 and `PATTERN-shell-mechanics-seam`; a pwsh statement would never run inside the hook shell.
- **Applies to:** cards 10, 12

### Decision: delivery-mark-beside-cycle-request

- **Decision:** The delivery mark is a JSON file under the orch dir beside `cycle-request`, holding the render time and the pointer text; the watcher reads it at the pointer step alone, keyed against `State.CompactionBaseline`, and removes it when the reload ends.
- **Rationale:** D11 keys the skip on a mark the verb writes only after a successful render, so the typed fallback stays reachable.
- **Applies to:** cards 11, 12

### Decision: carry-over-findings-covered

- **Decision:** The two carry-over findings of the decision record's `## Open risks`, the hook's cwd drift and the pointer naming a stale role file, are covered by cards 11 and 12 as the record states them: the hook command changes to the prime first, and the verb renders the role file before the pointer in every phase but `resuming`.
  Neither is a card of its own.
- **Rationale:** The record already fixed both; the plan implements the record.
- **Applies to:** cards 11, 12

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
