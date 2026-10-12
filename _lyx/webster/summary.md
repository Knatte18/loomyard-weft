# Alignment agent at parent review and PR-Gate

Both batches recorded `status: done`.

## Batch 01-loom-alignment-config (cards 1–6, head `70027516c`)

The batch added loom.yaml's alignment keys and their config checks, `Fabric.CommitLog`, the `lyx loom peers` verb and `peers show`, the `shedrun` alignment paths, and `internal/alignment` with its records, prompt, spec, snapshot, stencil and `Session`.
It had no declared-file deviations.
One departure from card 3, from the fork's own account: `lyx loom peers` carries no audience annotation, because `clihelp.Invocable` treats a runnable group with no `Use` token or flag as non-invocable, and `TestCLIForm_EveryCommand` rejects an audience on it.
`peers show` carries `AudienceRole`.
The fork also reported that the card-2 and batch gates name a nonexistent `./internal/fabricengine_test`, which fails setup on every run, and left a friction note for it.

## Batch 07-parent-review-gate-alignment (cards 7–12, head `f1ce7696f`)

The batch added the parent-review gate's alignment step, loom's wiring of the gate's `Aligner`, the shared `RecordApproval` and `RecordRejection` write path, the `PR-Align` producer and its recipe row, and the orch stencil and CLAUDE.md changes for escalation-only reviews.
One deviation: `internal/loomcli/parentreview_integration_test.go`, which no card lists.
The wired aligner now spawns a real session in that test, so its fixture sets `env.ParentReview.Aligner = nil`.
The fork did not run the `cmd/lyx` test-budget guard, since it lies outside the batch's packages.
It raised the budget rows for `internal/landingshed` and `internal/loomcli`.

## Verify gate fixes

The gate's first attempt failed on `TestEnforcement_FabricVocabulary/tree-scan`: `internal/loomcli/alignmentwiring_test.go` held a bare weft token outside the owner set.
The fixer reworded one comment from "weft repository" to "fabric record" in commit `c3c39e923` (`fix: drop bare weft token from alignmentwiring test comment`).
It reported `lyx gate test --tags integration ./internal/lyxcwd ./internal/loomcli` passing and no unfixed failures.

## Process notes

Both batch forks reported writing test files through Bash heredocs, which breaks the no-script edit rule.
The content was reported correct, and one such file was later fixed with Edit.
