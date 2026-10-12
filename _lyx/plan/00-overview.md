---
format: 5
approved: false
language: go
---

# Plan: lyx-halt

Give the operator one ordered pause that makes the hub safe to suspend within seconds: `lyx pause [<slug>]` records a pause, waits a short grace, interrupts every busy covered Claude session with Esc, and holds every path that would start agent or network work; `lyx resume [<slug>]` lifts it and gives each interrupted session one line to carry on.
Every reader that reacts to an unexpected stop or to idle time consults the record, so no failure, idle or crash path fires for a pause; the bare loom and webster boundary pause goes; and an unplanned sleep stops counting as idle time or killing work on wake, with issue #567's watchdog kill naming itself and stopping its step's agents.
The cards run in dependency order: the record and its shared clocks and holds, the geometry that tells its path, the network and lock holds, shuttle's holds and readers, reed and the restart refusals, batten's and loom's readers, the watchdog, the orch, the engine and CLI, then the boundary-pause retirement.

## Card Index

1 — pause-record — the pause record leaf: entries, coverage, intervals, the resume line and the covered refusal text
2 — sleep-watch — the shared sleep detection and awake-since reading
3 — network-gate — the shared wait at entry, network operation markers and the process network gate
4 — pause-geometry — the record path in hubgeom and standalonegeom
5 — gitexec-network-hold — network git waits at entry while a pause covers the process
6 — github-network-hold — every GitHub call waits at entry while a pause covers the process
7 — parentlock-covered — a covered run yields the parent lock queue and is never granted while covered
8 — shuttle-pause — shuttle starts wait, hold and dialog notices hold, and the deadline counts awake unpaused time
9 — shuttle-api-continue — one automatic continue line for an API-error stall across a detected sleep
10 — shuttle-step-tie — run.json names its step, and a stop verb ends one step's runs
11 — shuttle-pending-prompt — burler's live-fix resume delivers a task prompt whose turn never started
12 — runner-pause-wiring — every agent-starting runner is told its pause geometry
13 — shuttle-send-refusal — lyx shuttle send refuses into a covered strand
14 — reed-resume-hold — reed resume and batten's revive leave a covered pair's strands down
15 — start-refusals — lyx loom start, lyx loom resume and lyx batten run refuse on a covered slug
16 — batten-sleep-clock — batten's idle clocks use the shared sleep watch, fixing the eleven-hour notice
17 — batten-pause-reader — batten's notices, idle clocks, lock idle release and spawns honour the pause
18 — loom-idle-release-pause — loom's loop keeps a covered holder's parent lock
19 — watchdog-awake — the step watchdog measures awake unpaused idle time and never kills while covered
20 — watchdog-kill-record — a watchdog kill names itself and stops its step's shuttle runs
21 — orch-pause — the orch watcher interrupts, holds and resumes the orch under a hub pause
22 — pause-engine — the pause, resume and status sequences
23 — pause-cli — the pause module, lyx resume, the caller rules and the network gate install
24 — status-paused-since — loom and batten status carry paused_since
25 — loom-bare-pause-refusal — a bare loom or shed pause of a loom run refuses with both ways forward
26 — shed-pause-way-forward — busy and goto refusals name the recipe's pause, and the arbiter clause goes
27 — lock-wait-pause-after — pause --after naming a waiting row ends its parent lock wait
28 — webster-pause-retire — webster's pause verb, flag and paused outcome go

## Shared Decisions

### Decision: record-api

`internal/pauserecord` (cards 1 to 3) is the one leaf every reader imports: `Read`, `Update`, `Record.Covering`, `Record.CoverageOf` and `ReadCoverage` for coverage, `Excluded` and `AwakeSince` for clocks, `SleepWatch` for sleep, and `WaitUncovered` and `Network` for holds.
Every reader is told a `pauserecord.Geometry`, constructed only by `hubgeom.PauseGeometry` and `standalonegeom.PauseGeometry` (card 4), so no bound engine derives the path.
An unreadable record holds nothing and refuses nothing: each reader logs it and reads uncovered, so a corrupt file never freezes the hub.

### Decision: awake-unpaused-time

Every idle clock and deadline the pause touches (batten's notices, the step watchdog, shuttle's deadline) measures awake, unpaused time as `pauserecord.AwakeSince` over the union of the detected sleeps and the coverage's paused intervals.
A detected sleep is the interval ending at the reading that saw it, so an activity time keeps exactly the slept time after it; this replaces batten's running total, whose drop on a moved stamp caused the eleven-hour notice (card 16).

### Decision: network-hold-chokepoints

Network git is held in `internal/gitexec`'s `runCore` for its remote subcommands (card 5), the one runner beneath every fabricengine remote call, and GitHub in `githubclient`'s `authRT.RoundTrip` (card 6); no caller of either holds anything.
Both go through the process gate `pauserecord.Network`, installed once per `lyx` process by the CLI root through `pausecli.InstallNetworkGate` (card 23), the way `gitexec.SetKillReporter` is installed; with no gate, as in every test binary, `Network` is a no-op.

### Decision: pause-hub-resolution

`lyx pause` and `lyx resume` resolve the hub with `lyxcwd.Resolve`, as `cleanupcli` does.
D1 names `internal/cliwire`, but cliwire's fixed dependency set bars `lyxcwd` and `fabricengine` and it serves the standalone-capable CLIs only, so the binding constraint (`PATTERN-cliwire-sole-wiring` and cliwire's own import guard) wins over that wording.

### Decision: go-producer-reading

The pause report's "covered runs whose current producer is a Go producer" are covered runs whose status reads running while their pair has no live shuttle session: the run advances through local Go work until it reaches a held path (D4's out-of-scope note).

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
