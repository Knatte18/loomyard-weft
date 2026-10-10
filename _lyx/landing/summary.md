# Add the darn recipe and fix shared Publish, Finalize and teardown

Small changes are too big for a full loom run and today go to a hand-briefed agent in the prime, which skips the pair, the verify gate and PR-Gate, and leaves the prime dirty.
This adds `darn`, a second task recipe: one LLM producer reads the board entry, edits, tests and commits, then the usual Publish, PR-Gate and Finalize run.
A failed verify re-prompts the same producer up to a configured budget; a spent budget halts the run for the parent.

It also lands four fixes in shared code that the darn run depends on.

Closes #502
Closes #501

## What changes

- **Recipe selection.**
  The board entry's existing `recipe` field picks the recipe.
  Batten seeds a `darn` child for `recipe: darn`, and `Run-Shed` starts it through the shared bootstrap.
  An unknown recipe keeps the unsupported-recipe refusal, which now names `loom` and `darn`.
  `lyx loom start`, `lyx shed step`, `lyx loom resume` and `lyx loom goto` build the darn shed over a `darn` seed and loom's otherwise.
- **Darn recipe and producer.**
  `contracts/recipes/darn-recipe.yaml` is built by `loomrecipe.NewDarn` from the same registry.
  The `Darn` row is a `SingleLLM`-backed producer with a `verify` gate and the existing `description` gate, re-spawned with the operator's findings after a PR-Gate rejection.
  `lyx loom reject` accepts a darn run at `PR-Gate` and a darn run blocked at `Darn` with a pending rejection.
- **Config.**
  A hub-wide `darn.yaml` holds the writer model, writer timeout, the verify command and `verify_attempts`.
  An empty `verify` or `verify_attempts` below 1 refuses `lyx loom start` with the way forward.
- **Verify command source.**
  The verify gate and the landing producers take the command as told, so loom passes the plan's `## verify:` and darn passes `darn.yaml`'s.
  The gate also takes an optional merge base, read through a new go-git merge-base in `gitrepo` and `fabricengine`.
- **#502, verify logs.**
  `internal/verifytree` writes a numbered `verify-<n>.log` per run instead of overwriting one file; the failure record names its own run's log.
  History is bounded, and a log a failure record names is never pruned.
- **#501, Finalize.**
  Finalize commits a dirty `_lyx/shed/` subtree on the parent weft before its parent-side merge.
  A dirty path elsewhere on the parent still refuses.
- **Teardown.**
  Pair teardown settles the run's board claim: a landed run's entry is marked done, an abandoned run's claim is cleared, and a run still `running` keeps it with a warning.
  A board or status-read failure does not change the teardown verdict.
- **Halt note.**
  When a halt names a dirty worktree, the Go halt note lists the parent pair's dirty paths, and the reflection stencil reads it.
- **Docs.**
  The orch stencil, the `ly:board` skill, board and `lyx loom reject` help, `docs/overview.md`, `refusal-spec.md`, `shed-recipe-spec.md`, `PATTERN-verified-tree` and `PATTERN-pair-teardown` name the darn recipe or carry the new clauses.

## Design decisions

- Publish's failed-verify way forward comes from the recipe through a `Deps` field: loom names the `Webster-Burler` goto, darn names `lyx loom goto --to Darn` then `lyx loom resume`.
- The verify attempt budget is counted afresh for each `Darn` spawn.
- The `tmux` tier still runs at Publish for darn exactly as for loom.
- The `Darn` row sits in segment `PR-Review` from the first step, so a status reader that labels rows by segment shows a writer under a review segment name.
- A hand-started darn is not supported: `lyx loom start` with no seed seeds `loom`, so a darn run starts only through batten.
- Finalize committing the parent weft's records can attribute a sibling run's status commit to the landing run's message; the content is batten's and unchanged.
- Clearing the claim also fires for a pair removed by hand while its batten run is halted; the entry then loses the trace of where the run stood, which batten's `_lyx/shed/<slug>/` records in the prime keep.
  A pair removed by hand while its run is `running` keeps the claim.
- A hub with no `darn.yaml` yet refuses the first darn start until the operator sets `verify`.

## To check by hand

Nothing here was run against a live hub: no real darn run, board claim or Finalize has been exercised end to end.

- Start a darn run through batten from a `recipe: darn` entry, let a verify fail, and confirm the same session is re-prompted and a spent budget halts at `Darn`.
- Reject at `PR-Gate` and confirm the `Darn` spawn renders the findings.
- Tear down a landed, a halted and an abandoned run and check the board entry ends done, cleared and unchanged respectively.
- Gone-worktree claim: a pair whose worktree is already gone at teardown may leave its claim unsettled.
- Claim substrates: the real board and batten status stores behind the claim settlement are covered by fakes only.
- Seed bound: the darn seed bound and the loom resume path were reconciled late, so check that `lyx loom resume` over a darn seed builds the darn shed.
