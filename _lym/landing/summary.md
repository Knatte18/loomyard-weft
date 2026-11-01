# Hold writer and fixer sessions until their worktree is clean

A writing or fixing session could end with an untracked or uncommitted file in its worktree, such as a shell redirect's output or a scratch list of paths.
The run then went on and halted later, at Finalize's merge-in or at Parent-Align, until the operator removed the file by hand.
Now the session that made the file is told to commit or delete it before its turn ends.

## What changes

- New `clean` gate, `loomshed.NewCleanGate`.
  It fails while git reports an uncommitted or untracked path in the worktree, ignored files excluded.
  Its finding names every path and asks the session to commit what belongs to the change and delete the rest.
  A failure to read git is a returned error, not a finding.
- `shedrecipe` adds `clean` to the gate vocabulary and resolves it over `Env.WorktreeRoot`, which must be absolute.
  The unknown-name error lists the new value.
- The darn, sampler and tapestry recipes list `clean` before `verify` on the producers that run a verify gate: Darn-Write and Code-Burler-Fix on darn, Code-Burler-Fix on tapestry and on sampler. Each has `attempts: 3`.
- `verifytree` gains `UncommittedFindings`, the one finding text shared by the gate and the landing fixer.
- The `landingshed` merge fixer used to halt as soon as a turn left the tree dirty.
  It now re-prompts with the paths, on a charged turn, and halts as dirty only when no turn is left.
  `verifyGate.clean` is split so the fixer can read the paths through `dirtyPaths`.
- `shed-recipe-spec.md` lists `clean` and `parts` in the vocabulary and states the `clean` gate's contract.

## Design decisions

- `clean` is a separate gate rather than part of `verify`, so a stray file is caught before the verify command runs and costs no verify pass.
- It goes first in the list, so a writer fixes the tree before the verify runs against it.
- The fixer path keeps the existing dirty halt as the last resort, so a fixer that ignores the re-prompt still ends the run with the same stop as before.
- The gate's `attempts` is 3 in every recipe row, unlike the verify budget, which the builder stamps from `darn.yaml` at every build.

## To check by hand

- The Webster fork is not given a `clean` gate by this diff; only the recipe rows above are.
  Check whether the Webster row needs one too.
- No run through a real session was made: the gate is covered by an integration test against a scratch repo, and the fixer re-prompt by `TestFixerRun_Outcomes`.
