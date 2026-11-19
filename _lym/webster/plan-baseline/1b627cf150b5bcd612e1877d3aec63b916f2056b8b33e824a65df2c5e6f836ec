# Card 1 — gate-copies

**Create:**
- `plan:internal/gateslot#BilledTier` -> `const BilledTier = 3`
- `plan:internal/gateslot#FormatStartedLine` -> `func FormatStartedLine(run string, started int) string`
- `plan:internal/gateslot#ParseStartedLines` -> `func ParseStartedLines(output string) []StartedRun`
- `plan:internal/gateslot#StartedRun` -> `type StartedRun struct`
- `plan:internal/gateslot#TestStartedLines` -> `func TestStartedLines(t *testing.T)`
- `plan:internal/gatecli#TestRunTierCopies` -> `func TestRunTierCopies(t *testing.T)`

**Edit:**
- `internal/gatecli#newCommand`
- `internal/gatecli#testRequest`
- `internal/gatecli#flagShapeRefusal`
- `internal/gatecli#gateKeyRefusal`
- `internal/gatecli#execution.runWithSlots`
- `internal/gatecli#execution.report`
- `internal/gatecli#runInvocations`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`

**Extends:**
- `internal/gatecli/doc.go`
- `internal/gateslot/doc.go`

**Uses:**
- `internal/gateslot#Pool.AcquireSlots`
- `internal/gatecli#acquireSlot`

**Intent:**
Add `--copies <n>` to `lym gate test` (D4), and make every tier run report how many test binaries it started.

`newCommand` gains the int flag `--copies` (default 0, meaning off), carried on `testRequest`.
Refusals, each an envelope error naming its way forward, reached before any slot is taken:
- `flagShapeRefusal`: `--copies` below 0; `--copies` without `--gate` ("way forward: name the gate key with --gate"); `--copies` with no explicit package target ("way forward: name the packages to run as targets").
- `gateKeyRefusal`, which holds the loaded config: `--copies` with a key naming more than one tier, or naming `gateslot.BilledTier` (the tier that drives a billed LLM session), each naming the key and "way forward: set the key to one tier below tier3 with lym config gate --set from the prime".
- `execution.runWithSlots`: `--copies` in a call whose slot is inherited from an enclosing process (`exe.inherited`, set when `LYM_GATE_SLOT` names a held slot), naming "way forward: run the call outside a held gate slot", since the copies need a lease of their own.

The rows of `TestGateTest_RefusesBeforeResolvingTheHub` cover the three flag-shape refusals.

A `--copies` call takes one lease of n times the tier's own slot count through `gateslot.Pool.AcquireSlots` in `execution.runWithSlots`, in place of the tier run's own acquire, and releases it when the call ends, so it holds nothing past its call.
A tier whose slot count is 0 (`slots: all`) needs every slot per copy: its lease asks every slot and the call runs one copy.
`AcquireSlots` clamps a count above the pool to every slot; the call then runs one copy per whole tier share of the held slots (held slots divided by the tier's slot count, the pool's slot count for `slots: all`, at least 1) and prints one line naming the asked and the run copy count, the clamp line.
`runInvocations` starts the copies concurrently, each the tier's own rendered command over the same targets, each under the tier's counter, timeout, CPU watchdog and memory guard, and waits on every copy.
The call's result is red when any copy is red; each copy is labelled `copy <i>` in the failure lines.
Bound: a copy is never a raw compiled test binary, and a call with `--copies 0` or no flag runs exactly as before.

`execution.report` prints, for every tier run and for each copy, one line built by `gateslot.FormatStartedLine` naming the run label and `len(record.Launches)`, the packages whose test binary started; a tier run that started none prints the line with 0.
`gateslot.ParseStartedLines` is that line's sole parser, returning one `StartedRun{Run, Started}` per line; the live gate of card 6 reads it.

`TestRunTierCopies` is tier0, driven with a POSIX shell tier command and the stand-in cap and guard factories, as `TestRunTierProcessCap` drives one, over a hand-built `gateslot.Config` and a pool in a temporary directory:
- with a pool of 2 slots and `--copies 2` both copies run inside one lease and two started lines print, and the pool holds no slot once the call returns;
- with `--copies 3` over a pool of 2 the clamp line prints and two copies run;
- a tier with `slots: all` runs one copy and prints the clamp line;
- a red copy makes the call red;
- an execution with an inherited slot refuses `--copies` naming its way forward;
- `gateKeyRefusal` over a config whose key names `[0, 1]`, and over one whose key names `[3]`, refuses each naming its way forward.
`TestStartedLines` is tier0: a formatted line parses back to its run and count, a 0 count parses as 0, and lines of other text are skipped.
The package docs gain the flag, its bound and the started line.

**ImpactSummary:** Adds one flag and one output line to `lym gate test`; a call without the flag keeps its behaviour, and the line has one parser in gateslot.
