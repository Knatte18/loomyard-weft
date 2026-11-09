# Raddle's perch mode, decided by the recipe's rows, with its checks

A raddle run gets a third mode, `perch`, for a recipe whose rows hold a split perch: a Bouncer row that reviews the raddle proposal.
In that mode the perch's approval files the proposal in place of the operator's,
and the proposal checks allow only the two outcomes an arras run may end in.
No embedded recipe holds a split perch yet, so today's `raddle` runs behave as before,
apart from the corrected goto into Raddle-Write.

## The mode and the rows test

- `loomrecipe.HoldsSplitPerch(recipe)` and `RowsHoldSplitPerch(rows)` are the rows test.
  A row counts as a split perch when its engine is `Bouncer` and its `artifact_paths` names `loomengine.RaddleProposalRel()`.
  `internal/loomrecipe/testdata/split-perch-recipe.yaml` is the fixture recipe that pins it.
  The raddle, motif and tapestry recipes answer false.
- `RaddleMode` and its constants move from `loomshed` to `loomengine`, beside the raddle paths, and gain `RaddleModePerch`.
  `RaddleTable` takes the mode instead of an `interactive` bool, so it can render perch mode's own text.
- Raddle-Write's first call records `perch` when the new `RaddleWriteDeps.SplitPerch` seam reports a split perch, and then never reads `raddle_auto_approve`.
  Otherwise the key decides auto or interactive as before.
- Perch mode behaves as auto mode wherever the two differ from interactive mode:
  - the session ends on the finished proposal;
  - the gate's pass stores the cold proposal;
  - a fresh entry moves the proposal to the returned-proposal file;
  - Raddle-File files with no approval-hash match.

## Verbs

- `lym loom raddle-approve` refuses in perch mode, naming the split perch as the approval.
- `lym loom raddle-propose` in perch mode checks the proposal and stores the cold proposal, but posts no waiting notice, since no operator reviews it.

## Outcome checks

`raddleparser.BoardContext` gains `Perch`, set by `raddleBoardContext` from the run's raddle state.
It reaches the gate, both verbs and Raddle-File's checks.
With it set:

- One task passes: shape `tapestry`, or shape `tasks` with one part.
- A family passes: shape `project` whose `project.slug` is the run's own entry.
- Shape `beam`, shape `tasks` with more than one part and a project under another slug fail the new `perch-outcome` check.
  Its finding names the allowed outcomes.
- `slug-sketch` exempts that project slug.
  `slug-taken` already exempted the run's own entry for every run.

The proposal spec gains a Perch mode section and the new check, and the raddle stencil's opening line no longer says only the operator approves.

## `freshRaddleEntry`

The goto branch now matches the entry that `shedengine.Goto` and `Relaunch` write: Raddle-Write as producer, outcome `goto`, no `RoutedTo`.
The Raddle-File Stuck branch keeps its `RoutedTo` test.
Before, a goto into Raddle-Write was never fresh, so the new session started over the old proposal in every raddle run.
The table test's goto rows are rebuilt to that shape.

## `lym loom decision add`

On a perch-mode run whose current row is its RaddleFile row, done or not, the verb refuses after its caller checks:

- Before the filing it names the way forward: `lym loom goto --to Raddle-Write`, re-run the verb, then `lym loom resume`.
  The run then splits again with the decision recorded first.
- After the filing it lists the filed slugs, read from the raddle store entry whose created stamp matches the run's board entry.
  Its way forward is `lym loom decision add <part-slug>` once that part is released and its run has passed its record row.

A raddle run outside perch mode at Raddle-File is not refused.
The row comes from the recipe's rows (`EngineRow(recipe, "RaddleFile")`) and the mode from the run's state, never from the recipe's name.

## Design decisions

- `RaddleMode` moved to `loomengine` instead of a second enum there.
  `loomshed` imports `loomengine`, so the table factory could not name the `loomshed` type, and one type keeps one set of values.
- The perch option is a `BoardContext` field beside `Filing`, so every caller of the checks gets it from the one context builder.
- Raddle-File's board-write re-check (`RecheckBoard`) needs no perch option:
  its only slug rule, `slug-taken`, already exempts the run's own entry.
- Filing a perch-mode project as the converted run entry is out of scope, as the record's Delivers line says; arras-family-filing brings it.
  Until then no embedded recipe can reach that path, because none holds a split perch.

## Testing

- `raddleparser`: rows for each allowed and refused perch outcome, the slug exemption, and the same project refused outside perch mode.
- `loomshed`: Raddle-Write records perch without reading the key, and sets the proposal aside on a return or a goto in perch mode.
  Raddle-File files a perch-mode proposal with no approval.
- `loomengine`: `RaddleTable` across all three modes, first session and return, checks each mode's text and that no other mode's text appears.
- `loomcli`: the approve refusal and the propose pass without notice in perch mode.
  The two `decision add` refusals are covered, and a raddle run outside perch mode at Raddle-File still appends.
  `freshRaddleEntry`'s goto and return rows are covered too.
- `loomrecipe`: the rows test over the fixture recipe and the embedded recipes.

`go vet` passes over every touched package. The gate tests were not run in this session; the verify gate runs them on the committed tree.
