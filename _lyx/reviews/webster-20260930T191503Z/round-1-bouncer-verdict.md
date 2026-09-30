---
verdict: APPROVED
rationale: "All 14 round-1 findings are fixed in commits 0e1f112dd..224d017f2 and re-verified against the code; go build, go vet, go test, go test -tags integration and go vet -tags smoke all pass on HEAD 224d017f2, and a re-scan of the diff finds no mid-sentence Go comment wraps left (105 before the F11 commits, 0 after)."
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (merge base `2cb4ac68c`), with `main` read from `_lyx/shed/refusal-audit/status.json`'s `product.parent`.

The round-1 report had no `## Focus departures` section, so there are no departures to ratify or reject.
Its verdict was APPROVED with 14 findings, and the fixer's report says it fixed all 14.
I checked each fix against the code rather than taking the report's word for it:

- F1/F2: `refusal-spec.md` now has the missing rows, and I spot-checked the cells against the code: `fabricSyncWayForward`, the goto target, `missingStatusWayForward`, the loom seed/status text, status `--watch --json`, and Webster paused out of band.
  Where a message named its way forward before this audit (`no run in progress`, the `remedy:` unqualified-merge text, the `--plan-dir` override), it keeps its own wording, and the spec's new intro sentence says so.
  I confirmed all three against the merge base.
- F3: both recover-batch clauses now format `%02d` of `batchNumber`, and the test asserts the exact runnable command.
- F4: rebaseline's sync refusal appends `fabricSyncWayForward`, and a new test reaches it and checks the restamped fingerprint was saved.
- F5: the Master stencil has a terminal recover-batch `batch_failed` rung that forbids a second recover-batch, and the template test pins it.
- F6: all three goto refusals now end in a `way forward:` clause, and a new test takes the missing-status way forward.
- F7: a shared `fingerprintMismatchWayForward` const is used by begin-batch and run, and both are asserted through `requireWayForward`.
- F8/F9: the docs are corrected.
- F10: the empty-SHA slot is gone, and a new test pins it.
- F11: the heuristic scan is clean.
- F12: `TestFabricSyncWayForward_NextSyncCommitsSavedState` fails a sync against a real `hubforge` hub, then shows that the next sync commits `webster/state.json` into the weft.
- F13/F14: fixed as stated.

I found one residual NIT, which does not block.
In `contracts/specs/refusal-spec.md`'s loom intro (added by `7f01c4d35`), a line breaks before ", except that loom's own seed…".
That break is not at a coordinating-conjunction or semicolon boundary, so it departs slightly from CLAUDE.md's semantic-line-break rule.
It is recorded in the ledger and does not warrant another round.
