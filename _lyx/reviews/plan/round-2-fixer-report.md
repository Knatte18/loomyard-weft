# Fixer report — plan round 2

## Changes

- F1, fixed as proposed, in `_lyx/plan/05-run-bounded-lookup.md`:
  - Intent: the "refused before `wireLanding` … propagated read failure" sentences are replaced. `seedRunStart`'s error covers decode and validation failures as well as read failures. `loomCLI.resolveRecipe` refuses an undecodable seed first only when the addressed run is the self run. `lyx loom run <run-id>` and `lyx loom step <run-id>`, naming another run, reach `wireLanding`'s branch.
    I checked the premise against `internal/loomcli/arm.go:89-94`, `:127-133` and `:412`, and `sharedbootstrap.go:100` and `:309`.
  - `TestWireLanding_TellsLanguageSkills`: after the `RunStart` assertion, the row overwrites the self seed with `{not json`, re-runs `c.wireLanding()` and asserts an error that carries the seed file's path.
  - ImpactSummary: it now names both new failure branches, the verbs' `runStart` read and `wireLanding`'s `seedRunStart` error.

## Self-check

- fix-consistency, LOW: `_lyx/plan/lens-review.md:13` still said the decode-error assertion was dropped because arming refuses that seed first. Fixed as proposed: the line now records round 2's F1 restoring the assertion and why, so the earlier dispute stands on both assertions.
- fix-claims, LOW: the rewritten sentence applied "only when the addressed run is the self run" to `sharedbootstrap.go:100` as well, but that read always reads the self seed. Fixed as proposed: the sentence is split, so the bootstrap read refuses on the start path whichever run is addressed, and only `resolveRecipe` depends on the addressed run.
- fix-proposal, MEDIUM: once the "exempt raw read failure" label is gone, the decode refusal reaching `wireLanding` is bound by `PATTERN-refusal-way-forward`. Card 4's message carries no `way forward:` clause (`contracts/specs/refusal-spec.md:4`, `:38`, `:41`). Fixed as proposed, with the wrap placed in `wireLanding`: it returns the error as `%w; way forward: rm <seed>, then lyx shed seed self --recipe loom --driver go --param parent=<parent>`, following the re-seed remedy in `resolveRunID` (`arm.go:116`) and the `rm` then re-seed precedent at `sharedbootstrap.go:176`. The test row also asserts the way-forward text.
  I did not put the clause in card 4's `seedRunStart` message, because the approve and reject verbs already append their own `transient, re-run` way forward to that error, and the message would then carry two.
- fix-affected-set: no findings; both production callers of `wireLanding`, `loomPreRun` and `buildLoomShed`, go through the changed function.

## Disputed

None.

## Deferred

None.
