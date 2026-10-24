# Gate test runs every tier and guard run past a failing one

`lyx gate test` used to stop at the first tier that failed.
A failure the change under test did not cause, such as one already present at the base, therefore skipped every later tier and the guard step, and the output gave no sign of what went unchecked (issue #864).

Now a failed run stops no later run: every listed tier runs, then every guard run of the plan.
The call exits with the first failing run's own code, in plan order, and closes with one stderr line naming each failed run and its code, for example:

```
gate test: failed runs: tier0 (exit 7), tier6 (exit 9); every later run still ran, and the exit code is tier0's
```

A call where every run passes prints no such line.

## What changed

- `internal/gatecli/run.go`: one loop over the plan runs listed tiers and guard runs in order.
  Each run returns a `runEnd`, its exit code and whether the call halts there.
  A refusal, a slot wait that does not end with the slots held (the `slot_free: false` answer included) and a context a terminating signal cancelled halt the call at once, as before.
  A run whose command failed, a timed-out tier included, does not.
  `guardRunOf` is gone, since the single loop replaces it.
- `internal/gatecli/doc.go`, the `test` verb's Long text and `--guards` usage, and `docs/overview.md` state the new contract and exit code.
  The Long text stays under the CLI-form test's byte cap.
- Tests: the scenario test that pinned stopping at the first failing tier now pins that a later tier still runs, failing with its own code, while the call exits with the first failure's code and prints the closing line.
  The passing two-tier row asserts no closing line.
  The `--guards` row with a failing tier now expects the guard group to run.
  The `--card` timeout row now expects the guard step after the timed-out tier, while the tier's own second invocation still never starts.

## Design decisions

- **Run past the failure rather than list what was not reached.**
  The issue offered both; running every step leaves nothing unreached, so no "not reached" listing is needed.
- **The exit code is the first failing run's.**
  This keeps the documented "failing tier's own code" for the one-failure case every caller meets today.
- **A timed-out tier is a failed run like any other.**
  It stops its own remaining invocations, but not later tiers or guard runs.
  The cost of a failing gate grows by at most the later runs' own `timeout_min`, which a passing gate already spends.
- **A refusal still stops the call**, since its stdout must stay one JSON envelope.

## Testing

`lyx gate test --wait --upto 1` over the diff scope passes tier0, tier1 and the guard step.
