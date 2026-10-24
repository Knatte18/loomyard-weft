# Decision record — gate-runs-past-failing-tier

## Goal

A tier that fails in `lyx gate test` no longer hides the tiers and the guard step after it.
A failure the change under test did not cause, such as one already present at the base, leaves every later tier and guard run checked and reported.

## Scope

- `internal/gatecli/run.go`: the loop over the planned runs, and how a run tells a test failure apart from a reason to stop the call.
- `internal/gatecli/cli.go`: the `test` verb's Long text and the `--guards` flag usage, which say the call stops at the first failing tier.
- `internal/gatecli/doc.go` and `docs/overview.md`: the verb and guard-step contract.
- `internal/gatecli/test_integration_test.go`: the tests that pin stopping at the first failing tier and the guard step skipped after a failure.

Out of scope: the scope computation, the green record's carry rules, the slot pool, and the gate keys in `gate.yaml`.

## Decisions

### A failing run no longer stops the call

Every listed tier runs, and then every guard run of the plan, whatever the earlier runs' outcome.
A run that started its command and exited non-zero counts as failed, a timed-out tier and an empty-tier failure included.
Rationale: the issue asks that a failure the batch did not cause never blinds the gate to later tiers and the guard step, and running them is the only way to check them.
Bound: the call takes longer only when a run fails, at most the later tiers' own `timeout_min` each, which the passing case already spends.

### The call still stops on a refusal, a busy hub or a signal

A refusal (its stdout must stay one JSON envelope), a slot wait that ends with `slot_free: false` or any other acquire failure, and a cancelled context from a terminating signal end the call at once, as before.
Rationale: none of these is a test outcome; a refusal and a signal mean the later runs cannot run as asked, and a busy hub is answered by running the same command again.

### The exit code is the first failing run's own

With failures, the call exits with the code of the first run that failed, in plan order.
Rationale: it keeps the documented exit-code contract, the failing tier's own code, for a call where one tier fails, which is the case every caller meets today.

### A closing line names every failed run

After the last run, when any run failed, the verb prints one line to stderr naming each failed run, listed tier and guard run alike, with its exit code.
Rationale: each tier's summary line already says it failed, but a reader of a long log needs the whole verdict in one place, and the issue asks the output to say what failed.

## Constraints

- `PATTERN-gate-slot`: each tier and guard run still holds its own tier's slots, released before the next run acquires.
- `PATTERN-test-tiers` and `PATTERN-gate-carry`: the green record still records only passes, so a failed tier never carries.
- `PATTERN-refusal-way-forward`: no new refusal is added.
- The `test` verb's Long text stays within the CLI-form test's byte cap.

## Auto-mode assumptions

- The issue offers two fixes; running past the failure covers both its asks, since every step is then reached, so no "not reached" listing is added.
- A refusal mid-run, such as a prebuild failure in a later tier, keeps stopping the call, since its output contract is one JSON envelope.

## Open risks

- A caller that reads the gate's output for the first failing tier's lines now also meets later tiers' output; the summary lines and run records are per tier, so a reader keyed on them is unaffected.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - cancelled-context-halt-returns-green (class design, severity MEDIUM, open in rounds 1)
  - failedtier-last-vs-first-failure (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- A call whose first tier fails still runs its later tier, and exits with the first failing tier's code while a later tier fails with another code.
- A call with `--guards` whose listed tier fails still runs the guard step, and exits with the listed tier's code.
- The closing stderr line names every failed run with its exit code, and a call where every run passes prints none.
- A refusal, a busy hub and a terminating signal still end the call at once, as the existing tests pin.

## Notes for the plan writer

- The plan's guard runs come last, so one loop over the plan in order covers listed tiers before guard runs.
