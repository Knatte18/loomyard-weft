```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, which traces each change in the diff to the goal point that asks for it and each goal point to the changes that build it.

**Goal points** (the decision record alone is the answer key; it has no `Added after Discussion` entries):
- **G1** (#216, #215): at `timeout_min` the gate kills the tier's whole process tree and reports a timeout.
- **G2** (#235): the counter reads the summary of every console logger verbosity.
- **G3** (#234): a caller's filter is merged into the tier's filter or replaces it, and the way to do it is documented.
- **G4** (#233): either the prompt passes `--tier`, or the gate's no-test notice says the filter matched nothing and names the tiers that would match.
- **G5** (#237): either tier0 falls back to a default, or the refusal names the key to add; and the Burler code prompt says what to run when there is no gate.

**Per change:**
- **C1** `withTimeoutMin`, its use in `runTierCommands` and `runBuild`, the cause-based `timedOut` check, the doc.go "Timeouts" lines and `TestWithTimeoutMin`/`TestRunBuildTimeout`: traces to G1.
- **C2** child marker and `killMarkedLeftovers` in `child.go`, the doc.go "Child lifetime" lines and `TestRunTierKillsWhatLeftTheProcessGroup`: traces to G1.
- **C3** `vstestTotalTests`, `vstestOutcomeCount`, `observeSummary`, `inTotals` resets, the doc.go counter lines, the `TestCounters` row and `TestVSTestCounterInvocationClosesAnOpenSummary`: traces to G2.
- **C4** `takeCallerFilters`, `andFilters`, the `filterArgument` signature change, the `renderTierCommand` merge, the planning refusal for a command's own `--filter`, `tierFilterLeaf`, the help line and doc.go lines, and the `TestRenderTierCommand`/`TestPlanTierRuns`/`TestFilterArgument` edits: traces to G3.
- **C5** `holdsCallerFilter`, `filteredNoTestNotice`, `tierRun.otherTiers`, `withOtherTiers` on both return paths of `planTierRuns`, the doc.go notice lines and the `TestRunTierNoTestNotice` rows: traces to G4.
- **C6** the planning refusal naming `gate.tiers.tier0`, the `gateslot.Config.Tier` and `Config.Gate` refusal text, the stencil paragraph in `burler-template-fixer.md`, the doc.go planning line and the `TestPlanTierRuns` exact row: traces to G5.
- **C7** the cli.go help shortening in `a98e830e0`, which drops the "no tier when every changed file is inert" clause and the "one lease of N times its slots" clause: forced by C4's added help line pushing the text over its byte cap. Both facts are still stated in `internal/gatecli/doc.go:49` and `:172-178`. It is no scope finding; whether the help may lose them is the handler's judgement.
- **C8** the gofmt and comment rewrap commit `60090d827`: forced by C1–C6.

**Per goal point:**
- **G1**: C1 and C2. One part is not built (finding 1).
- **G2**: C3.
- **G3**: C4.
- **G4**: C5, through the notice branch the goal offers.
- **G5**: C6, through the key-naming branch plus the fixer-stencil step.

No change traces to no goal point, and no commit carries a `Pattern-Departure:` trailer.

**Findings:**

1. **LOW · scope** — `internal/gatecli/copies.go:66`
   - Summary: G1 is only partly built. A `--copies` tier's prebuild is still bounded by `context.WithTimeout(ctx, TimeoutMin*timeoutUnit)` on the monotonic clock. A host sleep therefore still stretches that part of the tier's `timeout_min`.
   - Conflict: `internal/gatecli/doc.go:316-320` (diff lines 380-383) says the tier's `timeout_min` "bounds the prebuild and every invocation" and "counts wall-clock time as well as awake time". The summary says `withTimeoutMin` "replaces `context.WithTimeout` for the tier and for the build step".
   - Evidence: `copies.go:66` as read in the manifest's copies.go output.
   - Fix: bound the copies prebuild through `withTimeoutMin`, matching `runTierCommands`.

Tool calls: none. Everything came from the inherited context.
