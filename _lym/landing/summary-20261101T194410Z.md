# A writer or fixer session cannot end with stray files in its worktree

A darn writer, a Code-Review fixer or a landing merge fixer could end its session with an untracked or uncommitted file at the worktree root, such as a shell redirect's output or a scratch list of changed paths.
The run then went on until Finalize refused the merge-in over the dirty tree, or a Parent-Align fixer halted with "ended without a usable turn (dirty)", and the orch removed the file by hand.
This change makes the session that left the file remove it before it ends.

## What changes

- **A `clean` gate.** `loomshed.NewCleanGate` fails while `git status` reports any path in the worktree, ignored files excluded.
  Its findings name every path and ask the session to commit what belongs to the change and delete the rest.
  `shedrecipe` adds `clean` to the `gates` vocabulary; it needs only an absolute `WorktreeRoot`.
- **Recipes.** `clean` is the first gate of Darn-Write in the darn recipe and of Code-Burler-Fix in the darn, tapestry and sampler recipes, with 3 attempts.
  The verify gate's own dirty check stays, but the verify gate passes on some paths before it reaches that check, for example when the tree-scoped command is empty.
  The clean gate runs at every arrival, before the verify and description gates.
- **Landing merge fixer.** A Parent-Align or Finalize fixer turn that leaves the tree dirty no longer halts the row at once.
  The turn is charged, and the fixer is re-prompted with the paths and the same commit-or-delete instruction.
  Only a dirty tree after the last turn ends the fixer with the `dirty` halt, as before.
- **Shared wording.** `verifytree.UncommittedFindings` is the one finding text both paths send.

## Design decisions

- The check is a recipe gate entry, not a rule wired into every shuttle run, so each row opts in through its recipe like its other gates, per `PATTERN-shed-recipe-registry`.
- Webster needs no new entry: its end gate already fails on a dirty tree and re-prompts the master.
  The overlay fixers of Discussion and Plan write only under the ignored `_lym` tree and run no git, so they get no clean gate.
- "Untracked files outside its outputs" is read from git: a writer's own outputs, such as the change description and the friction note, sit under ignored directories and never trip the gate.

## Tests

- `TestCleanGate` (tier 1, real repo): a clean tree and an ignored file pass, while an untracked file and a modified tracked file fail and are named.
- `TestFixerRun_Outcomes`: a dirty turn with a turn left re-prompts with the paths and a clean next turn is fixed; a dirty tree after the last turn is the dirty end.
- `TestResolveGateSpec_*` cover `clean` as a vocabulary name and in the unknown-name error.
  `TestDarnVerifyBudget` pins the new gate lists of the darn and sampler rows.
