---
format: 5
approved: true
language: go
---

# Plan: Go driver per task, with the tackler and the arbiter behind it

The Go driver becomes a full alternative to the LLM loom driver for a task's own loom run, chosen per task through `run_settings`' `loom.driver_kind`, with the LLM driver as the default.
A hidden `lyx loom drive` verb steps the run through the shared `--until-stop` loop body, settles the mechanical stops by a closed rule table, and spawns a cold tackler only at a stop no rule settles.
The arbiter's existing spawn path is called in-process when a rule or the tackler's verdict calls for it, and everything needing a parent choice is handed back to the orch through a Go-authored stop report, the park marker and the orch notice queue.
Start, resume and batten read the driver's own lock as liveness, batten re-steps a halted driverless go child after a deploy, and shed gives every `paused` step stop a reason.
The cards land the records and reads first, then the tackler and the driver's pure decisions, then the loop and its verb, then the call sites that spawn it, then batten.

## Card Index

1 — shed-paused-reasons — every `paused` step stop carries a reason: `pause requested` or `cancelled`
2 — loom-config-keys — `driver_kind` and the three tackler keys, open to the local layer, validated by load and by `PrepareRunSettings`
3 — run-record-paths — shedrun's driver lock, tackler and settle paths; loomengine's tackler and settle records; the tackler directory in the status commit
4 — on-disk-build-and-verb-bodies — `buildvcs.OfFile`, `LastStepBuildChanged`, and the exported `RunUntilStop` and `RunGoto` bodies
5 — tackler-stencil — the recipe-blind tackler stencil, its registry rows and its prompt and spec composers
6 — arbiter-case-brief — the arbiter stencil's case-brief marker and check-the-facts rule, threaded through prompt, input and attempt
7 — tackler-call — spawn, wait, verdict validation and causes, paused-time deadline, per-stop-key attempts and the tamper check
8 — tackler-consume — a tackler `resume` through `shedengine.Relaunch` with an Admit and the applied mark
9 — batten-retry-exports — `RecordMatchesHalt` exported and `PublishRetrySpent` added for the Go driver's verify retry
10 — drive-rules — the closed rule table, the stop parse, the repair cap and binary eligibility, as pure functions
11 — drive-handback — the stop report, park through the commit-records code, the two notify rules and the watched-marker format
12 — drive-loop — the settle loop for Go's own rules: re-step, verify goto, moved runs, pause hold, done, busy and hand-back
13 — drive-agents — the tackler and the arbiter in the loop, the restart reuse, the applied-mark escalation and tampering
14 — drive-handover — a hand-over to a fresh driver of the on-disk build at a binary-eligible stop
15 — drive-verb — the hidden `lyx loom drive` verb, its driver lock and refusal, the real wiring and the docs
16 — start-resume-go-driver — start spawns `drive` and reads the driver lock; resume spawns a Go driver for a halted driverless go run
17 — loom-seed-driver-kind — loom's seed writer resolves an unseeded run's driver from `driver_kind`
18 — batten-child-driver — `child_driver` recorded only when typed, the adopted-seed refusal, Seed-Child resolving `driver_kind`
19 — batten-liveness-hold-retry — batten reads the driver lock, holds its notice for a tackler, and honors a Go verify retry
20 — batten-binary-restep — batten re-steps a halted driverless go child once per episode and build after a deploy
21 — batten-wiring — batten's new seams wired to the task worktree, and the watched marker naming the prime
22 — orch-go-driver-line — the orch's role stencil says a go-driven run has no driver to message

## Shared Decisions

### Decision: one package for the Go driver and the tackler

The Go driver and the tackler live in `internal/loomcli` beside the arbiter (A1, A2): `driverules.go` holds the pure decisions, `drivehandback.go` the hand-back, `drive.go` the loop, the verb and its tests, and `tackler.go`, `tacklerprompt.go` and `tacklerconsume.go` the tackler.
The records batten also reads (`TacklerMarker`, `SettleRecord`) sit in `internal/loomengine`, as the arbiter's do, so `internal/battencli` never imports another CLI package.
Every path is declared in `internal/shedrun`.

### Decision: seams, not processes, in every Tier 1 test

`goDriverDeps` is the Go driver's whole surface to the world; cards 12 to 14 test the loop over fakes of it, and card 15 wires the real seams.
Every spawn (tackler, arbiter, successor driver) goes through a seam a test fakes, and every spawn logs through `internal/logger` (`PATTERN-spawn-observability`).
The only unbounded waits are the `--until-stop` waiter, shuttle's `Wait` and the pause hold, all on `PATTERN-daemon-wakeups`' list.

### Decision: the stop key and the settle records carry every cap

The stop key is `loomengine.ArbiterStopKey`.
Every Go action appends one settle record to `shedrun.SettleRecordsFile` in the friction directory, a non-`.md` file the reflection never archives.
The repair cap, the one verify retry and the one hand-over per build are counted from those records, so a restarted driver keeps every cap.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
