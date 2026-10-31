```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every way forward the fix added: the Darn-Write missing-record route, the row-dependent `decisionWayNoRecord` text, the differs-from-committed-copy cases, and the halt route for D5's guard.

**Ways forward checked**
1. D4, Darn-Write missing-record refusal: `goto --to Record-Write` then `resume`. Record-Write is in no review segment, so `settledSegmentGuard` admits it. `goto` moves a run back from Darn-Write, Drift-Baseline or Drift-Await. On a branch with commits, the Drift rows pass because of `awaited_at` or scope off. This route makes progress.
2. D4, `decisionWayNoRecord` darn text by current row: `resume` at or before Record-Write; `goto --to Record-Write` then `resume` on a later row. Both can be carried out.
3. D4, differs from committed copy, appended-entries case: `lym fabric commit`, then re-run the verb. See finding 1.
4. D4, differs from committed copy, other cases: restore the committed copy, then re-run the verb. This makes progress.
5. D5, guard failure: restore each failing file to its latest version that `validate-discussion` accepts, `lym fabric commit`, then `resume`. `resume` re-runs Darn-Write, so the route makes progress. In the uncommitted case the restored file equals HEAD, and `fabricengine/commit.go:93` handles a commit with nothing else to commit. Reachable.

**Findings**

- MEDIUM. D4, the "Differs from its committed copy" bullet: "the way forward is `lym fabric commit` in the `_lym` weft repository, then re-running the verb".
  This case arises when an earlier `decision add` appended its entry and then failed to commit. After `lym fabric commit`, that entry is already committed. If the caller re-runs the same call, the entry is appended a second time, because nothing refuses a duplicate `Added after Discussion` entry. The only de-duplication in `internal/discussionparser/append.go:41` replaces a `## PATTERN changes` bullet by its entry ID; entries are appended whatever they hold. The same gap is in the D4 acceptance clause "naming `lym fabric commit`".
  Fix: re-run the verb only for an entry the committed record does not already hold.

Tool calls beyond context:
- Grepped loom's goto guards and read `internal/loomcli/settledguard.go`, to confirm `goto --to Record-Write` is admitted. My context held only the guard in `shedengine/goto.go`.
- Grepped `fabricengine` for its nothing-to-commit handling and `discussionparser/append.go` for duplicate handling. My context had neither, and I needed them to check the `fabric commit` and re-run steps.
