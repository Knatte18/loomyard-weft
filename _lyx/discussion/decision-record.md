# Decision record — gate-wait-cache-cap

## Goal

`lyx gate test`'s wait for a free gate slot never blocks an agent's Bash call past the prompt-cache lifetime.
Once the shared keep-cache-warm cap passes with every slot held, the verb answers that no slot is free yet and the caller runs it again.

## Scope

- `internal/gatecli`: the slot wait is bounded by `shuttleengine.DefaultAwaitCap`, the cap `lyx shuttle await` defaults to, and the busy answer becomes a re-run answer.
- `internal/gateslot`: `cli_wait_sec` leaves the config, the template and `Limits`, and becomes a retired key.
- `cmd/testtiming`: its slot wait takes the same shared cap, since `Limits` no longer carries a wait.
- `internal/shellguard`: the gate rule agents get says to re-run on the busy answer, in the await help's words.
- The webster implementer and verify-fix stencils describe the busy answer's new shape.
- Docs: `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `internal/shuttleengine` cap comment and `docs/overview.md`.
- Out of scope: a long test run after the slot is taken; the test budget governs it.
- Out of scope: resetting the hub's `cli_wait_sec: 180`; the orch does that after landing, and reconcile prunes the retired key.

## Decisions

### D1: the gate imports the shared cap

The slot wait is bounded by `shuttleengine.DefaultAwaitCap` itself, never a copy.
Rationale: `gatecli` already depends on `shuttleengine` transitively, so the import adds no edge to the build graph, and one definition keeps both waits under the cache lifetime together.
The cap's doc comment widens from the `--cap` default to every keep-cache-warm wait.

### D2: `cli_wait_sec` is retired

`cli_wait_sec` leaves `Config`, the template and `Limits`, and the gate spec lists it in `RetiredKeys` as `CategoryCrossPair`.
Rationale: a bound the operator could raise past the cap would break the rule, and the cap has no tunable reason.
Every load drops a still-set key, and `lyx config reconcile` prunes it from the hub file because it is no longer a template key, so a hub that still sets it keeps loading.

### D3: the busy answer is not a failure

With every slot held at the cap, the verb prints an ok envelope with `slot_free: false`, the holders by worktree and site, and a message telling the caller to run the same command again, and exits `SlotBusyExit` (75).
Rationale: 75 is already the verb's own code, distinct from go test's codes and the signal codes, and the stencils already treat it as no failure;
an ok envelope stops it reading as an error, the shape `lyx shuttle await`'s ready false answer has.

### D4: the gate rule names the re-run

The shell guard's gate-rule text adds that a no-slot-free answer is re-run in a turn that does nothing else, so the prompt cache stays warm between calls, the words the await help uses.

### D5: the wait stays injectable for tests

`newCommand` takes the slot-wait cap as a parameter: `Command` passes the shared cap, and the integration helper passes a short one.
Rationale: the busy-hub test cannot wait out 250 s, and a package variable would break parallel tests.

## Constraints

- PATTERN-test-speed: the busy-hub test shortens the wait through injection, never waiting out the real cap.
- PATTERN-refusal-way-forward: the busy answer names re-running the same command.
- PATTERN-no-denied-recovery: the re-run step names no denied command.
- PATTERN-tolerant-records: a hub file that still carries `cli_wait_sec` loads.

## Auto-mode assumptions

- Exit code 75 stays: "an exit code of its own that is not a failure" is read as the verb's distinct code with an envelope that is not an error, not a new number.
- `cmd/testtiming` takes the shared cap instead of an unbounded wait, so a stuck holder still ends the measurement.

## Open risks

- A caller that parses the busy answer's old `ok: false` error envelope breaks; none in Go reads it, and the stencils change in this task.

## Acceptance criteria

- With every slot held, `lyx gate test` returns after the injected cap with exit 75, an ok envelope carrying `slot_free: false`, the holders and a re-run message, and runs no go test.
- `Command` bounds the slot wait by `shuttleengine.DefaultAwaitCap`.
- A gate.yaml that still sets `cli_wait_sec` loads, and its other keys still apply.
- The gate config's resolved `Config` and `Limits` carry no CLI wait.
- The gate rule's text carries the re-run instruction.

## Notes for the plan writer

- `hubgeom`'s gate-limits test pins `Limits` and needs its `CLIWait` dropped.
