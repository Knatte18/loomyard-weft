# Gate test: refuse untakeable targets, name unreached dotnet projects

Fixes [issue #105](https://github.com/Knatte18/loomyard/issues/105).

A tier command that holds neither `{targets}` nor `{test_targets}` runs whole whatever scope it is given.
Under explicit targets and under a narrowed `dotnet` scope that meant a gate could report a pass for tests other than the ones asked for.
In the reported case a card's tests lived in a second test project, tier0's command named only the fast project, and every gate route passed without running them.
The only signal was a stderr line saying the tier "runs whole", which reads as a superset rather than a miss.

## What changes

**Explicit targets that no listed tier can take are refused.**
`planTierRuns` (`internal/gatecli/tiers.go`) now refuses the call before any slot when every kept tier's command lacks both placeholders.
The refusal names the targets and the tiers, and gives the way forward: drop the targets to run the tier whole, or name a tier that takes targets with `--tier`.
It also says only the prime adds `{targets}` to a tier's command, with `lym config gate`.
A call where at least one listed tier takes the targets still runs, and each tier that ignores them is named on stderr as before.

**The "runs whole" notice moved from run time to planning.**
`planTierRuns` prints it once per kept tier whose command ignores the scope's targets, after the refusal check.
The `tierRun.ignoresTargets` field, which existed only to carry that notice to `runListed`, is gone.

**A dotnet call names each selected test project no run reaches.**
`unrunProjectLines` (`internal/gatecli/dotnetscope.go`) reads the projects a narrowed `dotnet` share selected for a tier that runs whole.
A project counts as reached when a run's targets, or for a command without a target placeholder its command's words, name the project's file name, name a solution file, or name no project file at all.
The last two count as reached because lym reads neither a solution nor the directory a bare `dotnet test` runs in.
For every selected project no run of the call reaches, `runTest` closes with a line saying the pass is not green for that project.
Its way forward runs the lowest declared tier, not manual, whose command reaches the project, or, with none, names adding `{targets}` through `lym config gate`.
The line is output only: it changes no exit code or record, like the existing not-run-tiers line of a card scope.
It applies to every narrowed dotnet scope form, card, computed and `--base`, since each can select a project the whole run misses.

## Design decisions

- The issue offered refuse or warn for the target case.
  Explicit targets are refused only when no listed tier takes them, since then the call runs none of what was named.
  A mixed call such as `--tier 0 --tier 2 ./pkg`, with tier2 a whole-only tier, keeps running; the scenario hub's integration tests rely on that shape.
- For the card and diff routes the gate warns rather than refuses.
  A refusal there would halt every card gate of a C# hub whose tier0 names one project until the prime edits the config.
- The reach check is a file-name heuristic over the command's words, scoped to the `dotnet` scoper, where test projects are named files.
  Under `go`, a whole run over a module is a superset of its packages, so the existing notice suffices.

## Effects elsewhere

- `gateslot.FailedTestsCall` renders explicit-target calls per tier.
  On a tier whose command takes no targets such a call is now refused instead of running the tier whole with its `-run` pattern.
- darn's `tree_scope` passes explicit targets beside `--gate`.
  On a hub whose key lists only tiers without a target placeholder that gate now refuses instead of passing whole.
  This hub's tiers all take targets, so neither path changes here.

## Tests

- `TestPlanTierRuns` gains a refusal row for explicit targets no tier takes, and a row where a whole-only tier runs beside a tier that takes the targets.
  The two rows that expected the dropped `ignores` marker now assert the stderr notice.
- `TestUnrunProjectLines` covers the reach rules: an unnamed project with a targets tier or a naming tier as the way forward, a manual-only tier, a project another run names, a solution file and a command naming no project.
- The missing-build-dir scenario subtest decodes the last stdout line as the refusal envelope.
  Its tier2 run-whole notice now prints at plan time, ahead of the envelope.
- The integration subtest that pinned `--tier 2 ./pkg` running whole is removed, since that call is now the refusal the unit row pins.
