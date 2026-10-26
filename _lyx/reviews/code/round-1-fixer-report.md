## Changes

All four findings fixed as proposed, one commit per fix group; gate tests through tier 1 for `internal/planparser` and `internal/websterengine` are green.

- F1: `checkRenameLeavesTest` in `internal/planparser/validate.go` compares `goPackageDir(test.Unit)` with `goPackageDir(old.Unit)`, so a test in the external test package meets the renamed symbol; its doc comment says so. A row `test in the external test package is the symbol's` (want 1) joins `TestValidate_RenameLeavesTest`.
- F4: a `none`-language row (want 0) joins `TestValidate_RenameLeavesTest`.
- F3: the mixed rename/delete subtest in `internal/websterengine/recordbatch_git_test.go` no longer sets `LaterTargetsReturned`; its first call must return `*LaterTargetRemovedError` naming `WillGoAway` and not `WillMove`, and a second call after `forkRewroteReport` runs the existing rewrite assertions.
- F2: a subtest `a failed report is never handed back over a removed later target` records a FAILED report over a removed later target and asserts the call returns nil and `LaterTargetsReturned` stays unset.

Commits: `planparser: fix F1, F4`, `websterengine: fix F2, F3`, `websterengine: fix F2` (tightened after the self-check).

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness, LOW: the F2 subtest asserts no outcome of the call, so it could pass if a FAILED report left before the post-batch pass; fixed as proposed, in an own form: the subtest asserts the call returns nil, the outcome a probe run showed.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
