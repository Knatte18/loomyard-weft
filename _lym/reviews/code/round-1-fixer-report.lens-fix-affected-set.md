```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the round-1 fix (sets behind F1, F2, F3).

Unchecked findings: none.

Coverage:
- **F2** (hand-over wording sites): `pushResetRecords` has two callers in `reset.go`, and both now name a hand-over. `internal/webstercli/doc.go:37` ("A push that finds the run taken over … fails the verb") agrees with both. No missed member.
- **F3** (failure-branch outcomes): not-committed and hand-over now have subtests, and push-failed was already covered by a row. Committed with a successful push (or a skipped one) still has no row; the review rejected the reachable-origin case, so I do not raise it.
- **F1** (statements of the outward change): one finding below.

Findings:

- **MEDIUM**: `_lym/landing/summary.md`, `## Merge danger`: "The detached push already sent that commit there, so the synchronous push puts nothing new on origin." The code does not hold this.
  - The detached push is asynchronous and fire-and-forget, with its error dropped (`internal/fabricengine/commit.go:169-171`: `_ = spawnDetachedPushFn(pushRequest)`). When the verb runs its synchronous push, that child may not have sent anything yet, and it may fail.
  - The synchronous push is also not records-only. `pushPairRecords` calls `pairteardown.PushRun`, which "pushes the pair at l synchronously" (`internal/pairteardown/handover.go`), code side included. So "pushes the records side to origin" in the sentence before understates it.
  - The sentence is the stated ground for keeping `**Door:** two-way`. A defensible ground is narrower: the same commits are already queued for origin by the detached push every landed commit spawns, so the change adds no new kind of outward content.

Tool calls:
1. grep for `pushRecords`, `pushResetRecords`, "handed over" and `push_error` in `internal/webstercli` and `pattern`: the parent's grep did not cover `pushRecords` callers or `push_error` statements.
2. Read `internal/webstercli/doc.go` lines 28-40 and `pattern/PATTERN-run-fsm.md` lines 43-50: the context held only single matching lines from them.
