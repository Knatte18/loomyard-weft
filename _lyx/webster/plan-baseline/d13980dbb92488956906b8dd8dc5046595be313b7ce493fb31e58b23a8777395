---
format: 5
approved: true
language: go
---

# Plan: driver-context — the loom driver steps in only when something goes wrong

Go runs the shed steps in a detached loop, `lyx shed step <run-id> --until-stop`, and wakes the driver only at a stop: a halt, a step error, a hung step killed by the watchdog, or a stop condition the orchestrator recorded with `pause --before` or `--after`.
A failure stop leaves the status file non-running, the stop envelope carries what went wrong and where to read further, and the driver stencil is rewritten around the loop with a one-shot fork for the heavy work, so the driver's context stays small for a whole run.
The cards build the process primitives first, then the engine's conditions and failure write, then room in the PATTERN overview, then the loop in three steps, then loom's arming, the stencils, the teardown and the two trace-noise fixes.

## Card Index

1 — proc-tied-child — the child tie, tree kill, pid record and Windows named job in `internal/proc`
2 — gatecli-tied-child — `lyx gate test` runs its child through `proc.StartTied`
3 — shedrun-loop-files — the loop's files declared under the run's steps directory
4 — shedengine-stop-conditions — `pause_before` and `pause_after` honoured by the engine and cleared by goto
5 — shedengine-failed-stop — `WriteFailedStop` writes `failed` under the run lock while the file reads running
6 — pause-conditions — `lyx shed pause --before|--after|--clear` records and clears the conditions
7 — step-progress-compact — the step envelope's `progress` drops `remaining` and empty keys
8 — pattern-overview-room — eight PATTERN overview lines shortened, their detail in background files, so the later lines fit under the cap
9 — interrupted-kind-and-arming-refusal — the sixth kind, a `bootstrap` arming refusal with a trace on both step subtrees, and the hub-config way forward
10 — logger-child-traces — `logger.NewTraceID` mints a child's trace id and `logger.TraceFilesFor` lists a trace group's files
11 — loop-body — `runLoop`: children, stop classification, the `loop` object, the transient re-step, the failed write and the rich detail
12 — loop-detach-wait — the detached loop, the loop lock and pid file, the waiter, envelope currency, the dead-loop stop and `PATTERN-failure-stop-visible`
13 — loop-watchdog — an idle child is killed with its tree and reported `interrupted`
14 — loom-idle-timeout-key — `loom.yaml` gains `step_idle_timeout_min`
15 — loom-arms-loop — loom fills the loop spec, the trace and activity readings and the interrupted-stop friction hook
16 — start-keeps-status-strand — `lyx loom start` keeps the status strand on llm-driven runs
17 — driver-stencil-rewrite — the driver stencil around the loop, the guide stencil, the slimmer driver spec and the orch line
18 — pairteardown-loop — a live loop is busy, and the session end kills the loop and its step tree first
19 — plan-gate-findings-file — informational plan-gate findings logged as a count plus a file under the run's scratch directory
20 — stencil-drift-once — the stencil drift notice is one line per process

## Shared Decisions

### Decision: pattern-overview-cap

- **Decision:** every card that adds or edits a `PATTERN.md` overview line leaves the file at most 16200 bytes; card 8 makes room for every later line by shortening eight overview lines, their detail held in the entries' background files, and no invariant is dropped.
  Measured with `wc -c PATTERN.md`: 16375 today, about 15841 after card 8, about 15874 after card 9's shed-verb-set line, about 15902 after card 11's allowlist clause on that line, and about 16137 after card 12's `PATTERN-failure-stop-visible` line.
- **Rationale:** the overview is capped at 16384 bytes and stands at 16375 today, so the first card to touch it must make room for every later line.
- **Applies to:** cards 8, 9, 11, 12

### Decision: loop-files-told-through-spec

- **Decision:** `internal/shedrun` declares every loop file as a constructor under the run's steps directory, and `internal/shedverbs` reaches them only through `Spec.Loop`, filled by the arming module; the per-step trace copy and stderr files are reached through the told `StopFiles` closure, and a child's trace files through the told `TraceFiles` closure.
- **Rationale:** `PATTERN-shed-verb-set` keeps `shedverbs` free of derived paths and of `internal/shedrun`, whose constructors take a resolved location, and admits `logger.TraceFile` and `logger.TraceDir` only as values reported verbatim, while `PATTERN-shed-run-directory` keeps the run directory's layout in one declarer.
- **Applies to:** cards 3, 11, 12, 13, 15

### Decision: platform-split-symbols-take-no-handle

- **Decision:** a symbol with one body per GOOS keeps a single exported declaration in a shared file, and the per-GOOS bodies are unexported helpers behind it; such helpers take no `plan:` handle, since their glyph is ambiguous by construction, as `configurePlatformGroupKill` is today.
- **Rationale:** the plan gate resolves a glyph to one declaration, and `quarry` answers a platform-split symbol as ambiguous.
- **Applies to:** cards 1, 2

### Decision: windows-bodies-cross-vetted

- **Decision:** every card that adds or changes a `_windows.go` body carries a `**Verify:**` that runs `GOOS=windows go vet` over its own packages.
- **Rationale:** `go vet` on a Linux host skips `_windows.go` files, and nothing else in the plan or the repository type-checks them before landing.
- **Applies to:** cards 1, 2, 12, 18

### Decision: loop-seams-for-tier-one

- **Decision:** the loop's child runner, spawn, file watch, sleep, clock, trace listing and activity reading are fields of `LoopSpec`, nil selecting the production value where one exists, so every loop behavior is tested untagged through `stepCmd` with `--until-stop` over fakes, and one `integration` test runs a real child.
- **Rationale:** `PATTERN-test-speed` keeps tier 1 free of spawns and waits, and `PATTERN-test-economy` tests the loop's unexported body through the verb that uses it and keeps the real-process test to the boundary it pins.
- **Applies to:** cards 11, 12, 13

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
