# gate test: run later tiers and the guard step past a failing tier

`lyx gate test` used to stop at the first failing tier.
A failure the change under test did not cause, such as one already present at the base, then hid every later tier and the guard step, and the output gave no sign they were skipped.

Now a failed run stops nothing: every listed tier runs, then every guard run of the plan, whatever the earlier runs' outcome.
A run that started its command and exited non-zero counts as failed, a timed-out tier included.

Closes #864

## Design decisions

- The call still ends at once on a refusal (stdout stays one JSON envelope), a slot wait that ends without the slots held, and a cancelled context from a terminating signal.
  None of these is a test outcome.
  When a halt follows a failed run, the closing line says later runs did not run.
- The exit code is the first failing run's own, in plan order, so a call where one tier fails keeps the documented contract.
  A busy hub after a failed run exits with that failure's code, since `SlotBusyExit` reads as no failure.
  A halt after only passing runs never exits 0.
- When any run failed, the verb prints one closing stderr line naming each failed run, a guard run as `guard tierN`, with its exit code, and which run's code the call exits with.
  A call where every run passes prints none.
- `gateslot.FailedTier` now returns the first failing tier summary line instead of the last, so it matches the exit code.
  Its caller in `landingshed` (the verify gate's failure record) therefore names the tier whose code the call exited with.
- No new refusal is added, each run still holds its own tier's slots and releases them before the next acquires, and the green record still records only passes, so a failed tier never carries.
- Docs follow in `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `internal/landingshed/doc.go`, `docs/overview.md`, and the `test` verb's Long text and `--guards` usage in `cli.go`.

## Cost

A call with a failing run now takes the later tiers' own time, which the passing case already spends.
A caller that read the gate's output for the first failing tier's lines now also meets later tiers' output; the summary lines and run records are per tier, so a reader keyed on them is unaffected.

## To check by hand

Nothing outside the integration tests in `internal/gatecli`, which pin the first tier failing with a later tier still run, the guard step run after a failed listed tier, the first failing code winning, and the closing line.
A live `lyx gate test --upto 2` against a hub with a real pre-existing failure was not run.
