# Name the run in a darn verify-budget notice

A darn run's verify gate sends its parent orch one notice once the gate has run past `darn.yaml`'s `verify_budget_min`.
The line names the gate site, the budget, the attempt and the failed runs, but not the run.
Neither the line nor its sender, `<shortname>:LOOM`, says which run it is about,
so an orch with several runs in Darn-Write could only guess.

The budget's `Notify` seam, built in `treeScopeSettings` (`internal/loomcli/darnwiring.go`), now opens the line with `loom <slug>: `.
Raddle's notices already use that form.
A notice now reads:

```text
loom verify-budget-notice-no-slug: Darn-Write gate: the verify gate has run past its 30m0s budget and is at attempt 3; ...
```

## Design decisions

- The slug goes in at the wiring seam rather than in `loomshed`'s `verifyBudgetNotice`.
  `loomshed` renders the gate's own facts.
  The loom CLI wiring knows the pair and is where raddle names its run, so both notices name the run the same way.
- The slug is the pair's worktree name, the value `orchcli.NotifyParent` already puts on the mail's run tag.

## Tests

`TestVerifyBudgetNotice_NamesTheRun` in `internal/loomcli/notifystamp_integration_test.go` (tier 1) builds a hub with one pair and runs creel in process.
It takes the budget `treeScopeSettings` builds for a darn run, calls its `Notify`,
and checks that the posted subject is the line prefixed with `loom budget-task: `.
The test fails on the merge base, which posts the line unprefixed.

## Merge danger

**Door:** two-way
**Blast radius:** loom
