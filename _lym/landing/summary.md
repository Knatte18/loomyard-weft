# Gate test fixes for dotnet hubs: hung tiers, loggers, filters, tier0

Fixes to `lym gate test` from the dotnet hubs' reports.

Closes #216
Closes #215
Closes #235
Closes #234
Closes #233
Closes #237

## timeout_min counts across a host sleep, and the kill reaches what left the process group

Dotnet tiers whose test hosts hung held their gate slots for about 21 hours, and their `timeout_min` bounds never fired.

- `withTimeoutMin` (`internal/gatecli/tierrun.go`) replaces `context.WithTimeout` for the tier, the build step and the `--copies` shared prebuild.
  It counts the monotonic and the wall clock, whichever has run further, and reads them at least every 10 seconds.
  A bound the host slept through therefore ends the run within seconds of waking.
  A timed-out tier or build is told apart by the context's cause, `errTimeoutMinPassed`.
- `runTiedChild` (`internal/gatecli/child.go`) gives every tied child its own environment entry, `LYM_GATE_CHILD_<pid>_<n>=1`.
  Once the shell ends, timed out or not, it kills the shell's process group and every process that carries the entry.
  That reaches a test host in a session of its own, which the group kill misses.
  A process that cleared its environment still escapes.
  On Windows the kill-on-close job ends the tree as before.

## The vstest counter reads the normal and detailed loggers' summary

With `--logger "console;verbosity=detailed"`, a passing run was reported as having run no test.
The counter now reads the `Total tests:` block as well as the minimal logger's one-line summary.
The block's `Passed:`, `Failed:` and `Skipped:` lines count up to `Total time:`, a one-line summary or the end of the invocation.
Other lines in between, such as another project's output, are passed over.

## A caller's --filter is ANDed into the tier's {filter}

`dotnet test` refuses a second `--filter`.
When a tier's command holds `{filter}`, a caller's `--filter X`, `--filter=X` or `--filter:X` now leaves `{args}`.
It is ANDed with the tier's filter and the scope's class terms.
A command without `{filter}` keeps the words in `{args}`.
Planning refuses a caller `--filter` for a command that holds its own `--filter` and no `{filter}`,
naming `gate.tiers.<tier>.filter` as the place for that expression.
It also refuses words after `--` that end in a bare `--filter`.
The verb's help states the merge.
To stay under the help text's byte cap, it drops the inert-only note on scope `all` and the slot-lease clause of `--copies`.

## The no-test notice names the tiers a filtered call did not run

A call through a gate key that runs tier0 only, filtered to a tier3 test, got the `IsTestProject` notice.
When the words after `--` carry a test filter (`--filter` or `VSTestTestCaseFilter`),
the notice now says no test of the project matched it.
It names the declared tiers the call does not run and gives `lym gate test --tier <N>` as the way forward.

## A hub without tier0 gets a refusal that names the key

- The refusals for a missing tier0, from planning and from a gate key, now name `gate.tiers.tier0`.
  `gateslot.Config.Tier` names the missing tier's key too.
- The Burler fixer stencil says what to do when `lym gate test` refuses to run any tier,
  no change to the fixer's own call clears it, and the way forward is a gate config change.
  The fixer runs no raw test command, checks that each changed package or project compiles,
  and defers the refusal to its parent.

## Design decisions

- A hub with no tier0 is still refused, even for `--tier N`: the fix names the missing key and adds no fallback tier.
- `timeout_min` counts wall-clock time, so a host sleep counts against it.
  Holding slots through a sleep blocks other gates on the machine.
- The marker kill runs after every tied child, not only after a timeout,
  so any process a run leaves outside its group dies with the run.

## To check by hand

- No real hung `dotnet test` tree and no real host sleep was exercised: the bound is tested on fake clocks,
  and the kill on a `setsid` child.
- #215 reports that the bound had not fired 30 minutes in, possibly before the machine was suspended.
  Check that the sleep explanation is enough for that report.
- No run against a real `dotnet test` is part of the tests for the detailed logger's summary or the merged `--filter`.
- Fixer stencil no-gate step covering every refusal, in `contracts/stencils/burler/burler-template-fixer.md`:
  check that the step applies only to a refusal the fixer cannot clear by changing its own call.
- Planning's `otherTiers` being unasserted, in `withOtherTiers` (`internal/gatecli/tiers.go`):
  check that the `wantOtherTiers` rows of `TestPlanTierRuns` cover both return paths of `planTierRuns`.
- The wall-clock term of the `timeout_min` bound being unpinned, in `withTimeoutMin` (`internal/gatecli/tierrun.go`):
  check that the `TestWithTimeoutMin` row where only the wall clock passes the bound pins it.

## Merge danger

**Door:** two-way
**Blast radius:** gate
