# Card 3 — publish-verify-failed-fix-route

**Edit:**
- `internal/landingshed#verifyGate.recordFailure` -> `func (g verifyGate) recordFailure(kind verifytree.FailureKind, result verifytree.Result) bool`
- `internal/landingshed#verifyGate.verdict` -> `func (g verifyGate) verdict(result verifytree.Result, what, producer, parentBranch string, recorded bool) (string, error)`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed/verifygate_test.go`
- `internal/landingshed/publish_verify_test.go`
- `contracts/stencils/claims_test.go`

**Prosa:**
- `internal/landingshed/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/stencils/orch/orch-template-role.md`

**Intent:** A Publish verify that ran, exited non-zero and wrote its failure record names the fix route in its Stuck reason (D1).
`recordFailure` reports whether it wrote the record: false for a gate with no recorder, a result that is not a failed run, a timeout, a shell that could not start, and a write that failed (the write failure stays logged as today).
`verdict` takes that answer as `recorded`; for `StatusFailed` with an exit code and `recorded` true, it keeps today's reason up to the output path and replaces its trailing `fix forward on the task branch, then resume` clause with the way forward: `way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests`, so the reason names that one route and no hand fix.
The timed-out, could-not-start and dirty reasons are unchanged and name no goto, and so is the failed reason when `recorded` is false, which covers Finalize (no recorder) and a Publish whose record write failed.
`check` and `checkPublishVerify` pass `recordFailure`'s answer into `verdict`.
`TestVerifyGate_CheckMapsVerifyResult` keeps its exact failed reason, since its gate has no recorder, which pins the unchanged text.
`TestPublishVerify_FailureRecord` asserts the Stuck reason per row: the plan-verify and `publish_verify` rows end with the goto way forward and do not contain `fix forward on the task branch`; the timeout row names no goto; and a new row makes the record write fail (the verify directory's record path pre-created as a directory) and asserts a non-zero exit with no written record names no goto and leaves no record.
The refusal-spec's loom section gains the row `Publish: verify failed` (trigger: Publish's plan verify or `publish_verify` ran and exited non-zero and its failure record was written; class: correctness halt; way forward: the goto-then-resume clause above), placed beside the `Publish or Finalize: dirty tree` row.
The orch role's `## Escalations from a child` list gains the item: a Publish blocked on a verify that exited non-zero takes `lyx loom goto --to Webster-Burler`, then `lyx loom resume` in the task worktree, never `Plan-Bouncer`, since the plan does not change; the claims table for `orch-template-role.md` gains a `must` claim for `lyx loom goto --to Webster-Burler` and a `mustNot` claim for `goto --to Plan-Bouncer`.
The landingshed package doc's failure-record paragraph says the Stuck reason names the goto route only when the record was written.

**ImpactSummary:** Two unexported gate methods are re-signed; their only callers are the two gate methods on this card, and the Stuck text changes only for a Publish with a written record.
