# Gate test fixes for dotnet hubs: hung tiers, loggers, filters, tier0

Five `lym gate test` fixes from the dotnet hubs' reports, each in its own commit.

## A tier's timeout_min now ends a hung tier across a host sleep and kills the whole tree

Two dotnet tiers whose test hosts hung held their gate slots for about 21 hours on a machine that also slept.
Their 30- and 35-minute bounds never fired.

- `withTimeoutMin` (`internal/gatecli/tierrun.go`) replaces `context.WithTimeout` for the tier and for the build step.
  It counts both the monotonic clock and the wall clock, whichever has run further, and reads the clock at least every 10 seconds.
  Go's timers run on the monotonic clock, which stands still while the host sleeps,
  so a bound the host slept through now ends the run within seconds of the host waking.
  A timed-out run is told apart by the context's cause, `errTimeoutMinPassed`, rather than by `context.DeadlineExceeded`.
- `runTiedChild` (`internal/gatecli/child.go`) gives every tied child its own environment entry, `LYM_GATE_CHILD_<pid>_<n>=1`.
  Once the shell ends, after a timeout or not, it kills every process that carries the entry, as well as the shell's process group.
  The group kill misses a process in a session of its own, such as a test host, and the marker kill reaches it.
  A nested gate's children carry both gates' entries, since each name is unique.
  On Windows nothing changes: the kill-on-close job already ends the tree.

The package doc's "Timeouts" and "Child lifetime" sections drop the old limit that a process outside the group is not killed.

## The vstest counter reads the normal and detailed loggers' summary

With `--logger "console;verbosity=detailed"`, a passing run printed `Total tests: 1` / `Passed: 1`.
The counter read only the minimal logger's one-line `Passed!` / `Failed!` summary, so the gate reported that the run ran no test.
`vstestCounter.observeSummary` now also reads the multi-line form.
A `Total tests: N` line opens a summary whose `Passed:`, `Failed:` and `Skipped:` lines each add their count, up to the `Total time:` line.
A `Total tests:` line also closes a failed test's block.

## A caller's --filter is ANDed into the tier's {filter}

A tier command that renders `{filter}` also passed a caller's `--filter` through `{args}`, and `dotnet test` refuses two `--filter` options.
`renderTierCommand` now takes `--filter X`, `--filter=X` and `--filter:X` out of the args when the command holds `{filter}`.
It ANDs each expression with the tier's filter and the scoper's class terms, as `(tier)&(caller)&(terms)`.
A command without `{filter}` keeps the words in `{args}` as before.
Planning refuses a caller `--filter` for a command that spells its own `--filter` outside `{filter}`.
The refusal names `gate.tiers.<tier>.filter` as the place for that expression.
The verb's help and the package doc describe the merge.

## The no-test notice names the tiers a filtered call did not run

A darn session measured a tier3 test through `--gate darn-write`, which runs tier0 only.
The empty tier0 run printed the IsTestProject notice, which sent the session looking for an MSBuild property.
When the words after `--` carry a test filter (`--filter` or `VSTestTestCaseFilter`),
`filteredNoTestNotice` says instead that no test of the tier matched the filter.
It names the declared tiers the call does not run and points at `lym gate test --tier <N> <its package or test project> -- <args>`.
`planTierRuns` fills a new `tierRun.otherTiers` field from `notRunTiers` for that line.

## A hub without tier0 gets a refusal that names the key, and a Burler fixer gets a step for it

- When the hub declares no tier0, planning's refusal now names `gate.tiers.tier0` in full and says no call runs a tier until it exists.
  `gateslot.Config.Tier` and `Config.Gate` name the key as well.
- `contracts/stencils/burler/burler-template-fixer.md` tells a fixer what to do when `lym gate test` runs no tier.
  It runs no raw test command and checks that each changed package or project still compiles.
  It names the refusal in the fixer-report's deferred section, so the parent takes the step.

## Design decisions

- Planning still refuses a `--tier N` call on a hub with no tier0.
  The fix names the missing key instead of letting other tiers run, which keeps the rule that tier0 is the default every gate key needs.
- The wall-clock bound departs from the awake-time bounds elsewhere in lym, such as the loop lock's.
  A test command that outlived its minutes in wall time has failed, and holding its slots through a sleep blocks every other gate on the machine.
- The marker kill runs after every tied child, not only after a timeout.
  Processes that left the group, such as tmux servers a test forgot, then die with the run instead of holding the slots.

## Tests

Every new test or row fails on the merge base:

- `TestWithTimeoutMin` (tier0) checks the bound on a fake clock.
  A reading past the bound ends the context, an open bound keeps polling every `timeoutMinPoll`, and the last poll waits only for what is left.
- `TestRunTierKillsWhatLeftTheProcessGroup` (tier1) starts a `setsid` child from a tier command.
  It checks that the child is dead once `runTier` returns, both for a command that exits and for one that outlives its timeout.
- `TestRunBuildTimeout` (tier0) checks that a build outliving its `timeout_min` fails as timed out and blocks its tiers.
- `TestVSTestCounterInvocationClosesAnOpenSummary` (tier0) checks that an invocation boundary closes an open `Total tests:` summary.
- `TestCounters` gains a row for the detailed logger's summary.
  `TestRenderTierCommand` gains the filter-merge rows, and `TestRunTierNoTestNotice` the two filtered-notice rows.
  `TestPlanTierRuns` gains the exact refusals for a missing tier0 and for a doubled `--filter`.

## Merge danger

**Door:** two-way
**Blast radius:** gate
