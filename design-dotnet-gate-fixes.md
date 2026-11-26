# Gate test fixes for dotnet hubs: hung testhost, loggers, filters, tiers

`dotnet-gate-fixes` · Task · bug, gate, dotnet, darn, burler · running · Darn-Write

Depends on: [`config-keys-nested`](design-config-keys-nested.md)

Bundled from the notes below; each part keeps its note's text.

# Gate tier timeout_min does not stop a hung dotnet testhost; the slot stayed held 21 hours

Imported from [issue #216](https://github.com/Knatte18/loomyard/issues/216).

## What happened

In hub `dop`, run `models-torque-drag-adapter`, a Webster batch fork ran `lym gate test --tier tier3` at 2026-11-23 15:36.
Tier3 has `timeout_min: 35` in the gate config.
The tier's `dotnet test` testhost hung (about 4.5 min CPU in 21 h), and the gate slot stayed held for nearly 21 hours: `lym gate status` showed the slot held since 15:36 the next day.
The 35-minute bound never fired, so the fork, Webster and the run waited without any stop or notice to the parent orch.
Another hub's orch noticed the idle process; the parent orch killed the testhost by hand, and the gate then released its slot.

## Expected

At `timeout_min` the gate kills the tier's whole process tree (`sh -c dotnet test` → `dotnet` → `vstest` → testhost) and reports a timeout failure to its caller.

## Likely area

The timeout may kill only the direct child (`sh`), or wait on output pipes that the orphaned testhost still holds open, so the wait never returns.


## From issue #215

Imported from [issue #215](https://github.com/Knatte18/loomyard/issues/215).

On a Models hub, `lym gate test --wait --gate finalize` (PID 3536339) ran tier1, `dotnet test NORCE.Models.SlowTests -p:RunSlowTests=true` with `timeout_min: 30`, from 2026-11-23 15:36 to the next day, over 20 h. The test host had used about 7 min CPU; it was hung. The gate slot stayed held the whole time and the run sat at Finalize. On the same machine a DrillOpPlan hub's tier3 gate, started the same minute with a 35-minute bound, hung the same way and its bound never fired either. The machine may have been suspended overnight, but the bound did not fire before that either (15:36 + 30 min). Expected: the gate kills the tier's process tree at timeout_min and reports a timeout.

# gate test: reports 'ran no test' when the detailed console logger's summary is used

Imported from [issue #235](https://github.com/Knatte18/loomyard/issues/235).

With `--logger "console;verbosity=detailed"` passed to `lym gate test`, the gate reported that it ran no test, although the test log shows `Total tests: 1, Passed: 1`. The gate's test counter appears to misread the detailed logger's summary format.

This is abnormal because a passing run is reported as a run of nothing, which an agent may read as a failure or retry needlessly. The counter should parse the summary lines of every console logger verbosity, or read the result from a structured logger (trx) instead.

Provenance: task models-stick-slip-floor-tune, note Darn-Write.md.

# gate test: tier3 command already carries --filter, so a second --filter in args makes dotnet test refuse

Imported from [issue #234](https://github.com/Knatte18/loomyard/issues/234).

The tier3 command of `lym gate test` already carries a `--filter`. Passing a test filter in `{args}` adds a second `--filter`, and `dotnet test` refuses the call. An agent found that `-p:VSTestTestCaseFilter=...` works as an override, but nothing in the verb's help says so.

This is abnormal because selecting one test class is the common use of the verb. The gate should merge a caller filter into the tier's filter (or replace it) and document the way to do it.

Provenance: task models-stick-slip-floor-tune, note Darn-Write.md.

# gate test: darn-write gate key runs tier0 only, so a tier3 measurement test runs nothing

Imported from [issue #233](https://github.com/Knatte18/loomyard/issues/233).

A prompt told an agent to run a measurement test through `lym gate test --gate darn-write`. That gate key runs tier0 only, while the test lived in tier3 (a slow-test project, category `ModelsLong`). The first call ran no test, and the orch had to allow `--tier 3` by hand.

This is abnormal because the prompt and the gate key disagree on the tier. Either the prompt generator should pass `--tier` when the named test is not tier0, or the gate should say, when it runs no test, that the filter matches nothing at this tier and name the tiers that would match.

Provenance: task models-stick-slip-floor-tune, note Darn-Write.md.

# lym gate test answers that the hub declares no tier0, leaving a Burler fix round without a gate

Imported from [issue #237](https://github.com/Knatte18/loomyard/issues/237).

A Burler code fix round ran `lym gate test ./<module path>` on a Go module and the verb answered that the hub declares no tier0.
The round therefore had no gate, and the agent ran `go vet` and `go test` on the module directly.

This is abnormal because the fix-round instructions end each round with the gate; a hub without a declared tier0 leaves the round with nothing to run, and the agent improvised a substitute.

What to do: either have `lym gate test` fall back to a default tier when tier0 is undeclared, or make the refusal name the config key to add, and have the Burler code prompt say what to run when no gate exists.

Provenance: task slug timesheet-suggest-tool; notes burler-code-r1.md.

