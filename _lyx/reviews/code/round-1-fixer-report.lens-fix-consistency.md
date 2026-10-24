Task: check the fix-consistency lens over the change set (siblings and twins of the changed text).

**LOW** — `internal/websterengine/doc.go:442`, the sentence "dead with no retry, or terminal and not done with its recoveries spent and no amendment a spawn has not rendered".
- Summary: the module doc still states the old dead-end rule. The dead arm now spares a batch that holds an unrendered amendment, because `deadEndBatchError` has `case bs.Status == DigestStatusDead && !rendersNewAmendment(bs)`.
- Evidence: the doc reads "dead with no retry", and the "no amendment" bound is attached only to the second clause. The `deadEndBatchError` doc comment was updated to "...holds no such amendment", so the code comment and `doc.go` now disagree.
- CLAUDE.md requires the module doc to land in the same commit as the behavior change.
- The doc also says nothing about a failed git read returning a plain re-run error instead of a refusal. That is optional.

No other disagreements found:
- The `RecoveryRetry` doc comment ("A git read that fails ... gives false and is logged at Warn") still holds, since the wrapper keeps that behavior.
- The `beginbatch.go:556` caller is unchanged and still gets the same verdict.
- The master stencil line 137 (`recovery_retry: false`) is unaffected.
- No other file or test mentions "begun, not recorded terminal". The new `waitsOnParent` offender text has no sibling to match.

Tool calls:
1. grep of `doc.go`, PATTERN, docs and contracts for the dead-end wording and the `ErrBatchDeadEnd` references, plus `reentry` usages in `runlevel.go`. My context lacked the sibling docs and stencils.
