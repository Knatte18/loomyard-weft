# Shell guard steers say nothing ran and name the refused part

A shell guard deny refuses the whole compound command for one part.
The old steers named the refusal but never said that the command's other parts had not run either,
so an agent that opened with a probe or scratch step, then did useful read-only work, re-ran the parts it still needed one turn late.
The retro proposal behind this (`docs/research/retro/retro-20261116-115411/report.md`, proposal 3) found three such refusals.

Every steer, plain and per engine, now opens with one shared clause:
"Nothing in this command ran: the guard refuses the whole command for one part, so run the parts you still need again without it. The refused part: ".
The row's own text follows and names the refused part.

## Design decisions

- `internal/shellguard/rows.go` adds the clause once as `wholeCommandRefusal`.
  `Rows` applies it through `openSteersWithWholeCommandRefusal` to every row's `Steer` and to each non-empty `EngineTexts` steer,
  so no row repeats it and a new row gets it without edits.
- Notices are unchanged.
  A notice announces a guard before any command runs, so "nothing ran" would be wrong there.
- The comments on the python, shell-edit and headless claude constants no longer claim to open every steer;
  they now say the text follows the shared clause.
- `internal/shellguard/doc.go` records the rule and why notices are excluded.
- `settings-golden.json` under `claudeengine/testdata` is regenerated for the new steer text.

## To check by hand

- The test for the whole-command clause (`internal/shellguard/rows_test.go`) may lack the `testtiming` keep marker its sibling tests carry.
  Later commits add a marker to the whole-command steer test; confirm the contained test has one too.
- Trigger a refused compound command in a live lym session and read the steer the agent sees.
  No test checks how it renders in either engine.

No regression check ran for this task.
