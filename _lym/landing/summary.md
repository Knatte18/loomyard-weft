# Raddle perch mode: split-perch recipes file without operator approval

A raddle run whose recipe rows hold a split perch, a Bouncer row over the raddle proposal, now records a third raddle mode, `perch`.
In that mode the perch's approval files the proposal in place of the operator's, so `raddle_auto_approve` is never read.
This is the raddle half of the arras recipe, where the perch replaces the manual approval the operator kept while raddle was being tuned.
A `raddle` run is unchanged: it keeps its interactive approval when `raddle_auto_approve` is off.

## What changes

- `loomrecipe.HoldsSplitPerch` and `RowsHoldSplitPerch` decide perch mode from the recipe's rows: a Bouncer row whose `artifact_paths` name the raddle proposal.
  A fixture recipe, `testdata/split-perch-recipe.yaml`, tests it.
- `RaddleMode` moves from `loomshed` to `loomengine` and gains `RaddleModePerch`.
  Raddle-Write records the mode on its first call; it picks perch before it reads `raddle_auto_approve`.
- In perch mode:
  - the raddle gate's pass stores the cold proposal, as in auto mode;
  - `raddle-propose` posts no operator notice and tells the agent to end its turn on the finished proposal;
  - `raddle-approve` refuses, naming the perch as the approval;
  - Raddle-File files with no approval-hash match;
  - every fresh entry into Raddle-Write moves the last proposal aside.
- `raddleparser` takes a `Perch` option and a new `perch-outcome` check.
  Perch mode allows one task (shape `tapestry`, or shape `tasks` with one part) or one project (shape `project` whose slug is the run's own entry).
  Shape `beam`, shape `tasks` with several parts and a project under another slug fail the check with a finding naming the allowed shapes.
  `slug-sketch` and `slug-taken` exempt the perch-mode project slug.
- `lym loom decision add` refuses on a perch-mode run from the Raddle-File row on, with a way forward.
  Before the filing it is `lym loom goto --to Raddle-Write`, which re-splits with the decision recorded first.
  After the filing the refusal lists the filed slugs, read from the run's filing, and points to `lym loom decision add` on the part once the operator has released it and its run has passed its record row.
  A `raddle` run behaves as before.
- The raddle spec, the raddle stencil's mode text and `docs/overview.md` describe perch mode.

## Design decisions

- Perch mode is decided by the recipe's rows, never by the recipe's name.
- `freshRaddleEntry` in `internal/loomcli/raddlewiring.go` is corrected to match a goto entry as `shedengine.Goto` and `shedengine.Relaunch` write it: producer `Raddle-Write`, outcome `goto`, no `RoutedTo`.
  Before, a goto into the row was never fresh, so the seat's new session started over the old proposal, in a `raddle` run as well as a perch run.
  The goto rows of its table test are rebuilt the same way.
- Beam is refused in perch mode, since a beam would start the family's runs and take the release from the operator.
- Filing a perch-mode project proposal as the converted entry is not part of this change; it comes with arras-family-filing.
  Here such a proposal only passes the checks.

## Bounds on the removed approval

Perch mode removes the operator's approval only in a run whose recipe rows hold the split perch.
Raddle-File still runs every proposal check and the board re-check, and every filed task stays draft until the operator releases it.
Three routes file a proposal that was not cleanly approved:

- With the split perch's auto-accept key on, a spent budget accepts a round that still had open findings.
  The key defaults to false and its guard category marks it as one that hollows an approval.
- With the key off, `softCapRule` in `internal/shedadapters/bouncer.go` accepts a spent-budget round with no open BLOCKING or MEDIUM entry tagged other than `mechanical`.
  Such a round may still hold open LOW and NIT entries and BLOCKING or MEDIUM entries the judge tagged `mechanical`, and the proposal is filed with them.
  The judge's tag is the only thing between a BLOCKING `mechanical` entry and the filing.
- A goto from a perch that has not settled can file a proposal no round reviewed.

The open findings of an automatically accepted round are not carried into the record; they stay only in the round's ledger in the perch's run directory.

## To check by hand

- Run a perch-mode raddle run end to end against a real split-perch recipe; the tests use a fixture recipe, and nothing here ran the full run.
- Read the `perch-outcome` finding text and the `lym loom decision add` refusals in a terminal for clarity.
