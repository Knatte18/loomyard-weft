---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/websterengine/gitwrap.go:4
    summary: New gitwrap.go comments (header lines, refuseMidMerge and reconcileReportHead doc) are fixed-column wrapped, splitting phrases mid-line and leaving a semicolon clause on the same line
  - id: F2
    severity: LOW
    location: internal/websterengine/doc.go:80
    summary: The new doc.go merge-only paragraph is fixed-column wrapped, and its opening sentence says both verbs twice and hangs the acceptance rule off the recover-batch clause via a colon
  - id: F3
    severity: LOW
    location: internal/websterengine/recordbatch.go:79
    summary: Edited comments on RecordResult, postBatchInputs, the postBatchChecks delta note, RecoverResult and PersistRecoveryTerminal keep fixed-column wraps that break phrases mid-line
  - id: F4
    severity: LOW
    location: internal/websterengine/inflight.go:12
    summary: RunInFlight doc breaks before a non-clause phrase (per the Cwd Resolution...) and leaves the semicolon clause (every path is built...) on the same line
  - id: F5
    severity: LOW
    location: internal/fabriccli/merge_verbs.go:34
    summary: websterInFlightWarnings doc leaves a semicolon clause on its line, and the addMergeVerbs loc paragraph is wrapped mid-sentence before a relative clause
  - id: F6
    severity: LOW
    location: internal/fabriccli/merge_verbs.go:30
    summary: The merge-in warning asserts record-batch tolerates this parent merge, which is false for a fast-forwarded merge-in (AlreadyUpToDate false, Committed false) past a fork's reported head
  - id: F7
    severity: LOW
    location: internal/websterengine/recordbatch_test.go:938
    summary: New test-file comments (gitwrap_test, recordbatch_test, recoverbatch_test, parentmerge_test, mergein_webster_integration_test) are fixed-column wrapped mid-phrase
  - id: F8
    severity: NIT
    location: internal/fabriccli/mergein_webster_integration_test.go:66
    summary: assertOneWebsterWarning checks only for one non-empty string, so card 6's exactly-the-one-warning and same-warnings claims are not pinned to the warning's content
  - id: F9
    severity: NIT
    location: internal/websterengine/gitwrap_test.go:223
    summary: The non-merge-between-merges refusal case does not assert the error names both SHAs and the rule as card 1 requires, and the report-head-is-merge case's negative check misses the last list position
  - id: F10
    severity: NIT
    location: internal/websterengine/gitwrap.go:41
    summary: The mid-merge refusal is prefixed websterengine while the sibling operator-facing head refusal in the same verbs is prefixed webster
  - id: F11
    severity: NIT
    location: internal/websterengine/recordbatch_test.go:985
    summary: TestRecordBatch_ParentMergeAfterForkCommit aliases newHead := merge, so the names-the-new-HEAD check repeats the merge check under a second name
---

Review range: `_lyx/shed/webster-parent-merge-midrun/status.json` names `product.parent` = `main`;
`git merge-base main HEAD` is `0e5074bbe`, and the range is its six commits (cards 1–6).
`go build ./...`, `go vet` on both packages, and the plan's `verify:` test commands (untagged and `-tags integration`) all pass.

Per-card mechanical checks.
Edit groups: every caller of the three signature-changed symbols is updated — `setMergeExit` (merge-in plus the three `merge` branches passing nil), `errConflictsWithRecord` (`setMergeExit` plus both `envelope_test.go` calls passing nil), `addMergeVerbs` (its one caller in `weft_verbs.go`).
Create groups: no equivalent already existed.
The only other `MergeHeadPresent`/`CommitParents` users are fabricengine's own merge-state internals, which websterengine cannot and should not reach, and no webster in-flight or resume predicate existed outside `websterengine`.

Logic checks, all settled as correct.
`reconcileReportHead` terminates correctly for each case card 1 lists:
equal HEAD takes the fast path;
merges on top are collected HEAD-first (walk order);
a report head that is itself a merge is matched before its parent count is read;
a fast-forward or a non-merge commit between merges ends at a single-parent commit and refuses;
a side-branch-only or all-zero report head walks the first-parent chain to its first non-merge commit or the root and refuses;
an empty `head_sha` does the same in record-batch, and recover-batch skips the check as before.
`refuseMidMerge` sits after the report parses and before the dirty check in `RecordBatch`, and after the `DigestStatusRunning` return and before the head check in `awaitTerminal`.
The moved-HEAD warning reaches `RecordResult.Warnings` on the success path, and it reaches `RecoverResult.Warnings`, which the CLI extends with `PersistRecoveryTerminal`'s warnings.
The merge-in condition `err == nil && (!res.AlreadyUpToDate || len(res.Conflicts) > 0)` matches card 6: it covers a committed merge, a fast-forward and the conflict envelope, and excludes hard failures and already-up-to-date.
The weft is not a merge participant, so "only the warp side moved" is the normal case, not an edge.
The ly-drive rule is recipe-blind and uses a semicolon clause break.

### [LOW] gitwrap.go comments are fixed-column wrapped

**Location:** internal/websterengine/gitwrap.go:4
**Issue:** CONSTRAINTS.md `## Comment Line-Break Convention` requires one sentence per line with breaks only at independent-clause boundaries, for new and changed comments.
The new header lines break `record-batch and recover-batch` / `share:` mid-phrase.
The `refuseMidMerge` doc splits `a linked worktree` / `keeps MERGE_HEAD`.
The `reconcileReportHead` doc splits `tolerating` / `merge commits only` and `BEFORE looking` / `at its parent count`, and it leaves `a non-merge commit or the root ends...` after a semicolon on the same line as the preceding clause, wrapped mid-phrase.
**Fix:** Rewrite each new comment to one sentence per line, breaking only after the semicolon.

### [LOW] doc.go merge-only paragraph: wrapping and sentence structure

**Location:** internal/websterengine/doc.go:80
**Issue:** The new paragraph is wrapped at a fixed column (`so a parent` / `merge-in landing`, and more).
Its opening sentence says the two verbs share the rule twice ("record-batch and recover-batch both ..." and "recover-batch follows the same ... as record-batch").
Card 3 appended the second statement to card 2's sentence, so the colon introducing the acceptance rule now hangs off the recover-batch clause and reads as if it described recover-batch alone.
The rule is stated once, but ambiguously attached.
**Fix:** Restructure into semantic lines, with the shared-rule statement once, the acceptance rule as its own sentence, then recording, warning, refusal and the known limit.

### [LOW] Edited recordbatch.go/recoverbatch.go comments keep fixed-column wraps

**Location:** internal/websterengine/recordbatch.go:79
**Issue:** The changed `RecordResult` doc (`after the fork's` / `commit)`), the changed `postBatchInputs` sentence (`Label names the batch` / `in warnings`), and the postBatchChecks delta note (`reconciled report` / `head`) are wrapped at a fixed column.
The same holds in recoverbatch.go for the `RecoverResult` doc (`head_sha and` / `the worktree's HEAD`) and the `PersistRecoveryTerminal` note (`either` / `path`).
All are changed comments, so the convention governs them.
**Fix:** Re-break the changed sentences semantically, leaving untouched sentences alone.

### [LOW] RunInFlight doc comment line breaks

**Location:** internal/websterengine/inflight.go:12
**Issue:** The line break falls before "per the Cwd Resolution and Told-Geometry invariants", which is not an independent clause.
The semicolon-introduced independent clause "every path is built..." sits on that same line instead.
**Fix:** Keep "per the ... invariants;" on the first line and move "every path is built ..." to its own line.

### [LOW] merge_verbs.go new comments' line breaks

**Location:** internal/fabriccli/merge_verbs.go:34
**Issue:** The `websterInFlightWarnings` doc puts "a RunInFlight error degrades..." on the same line after a semicolon.
The new `addMergeVerbs` paragraph breaks mid-sentence before the relative clause "which merge-in reads...", which is not an independent-clause boundary.
**Fix:** Break after the semicolon, and join the `loc` paragraph's sentence onto one line.

### [LOW] merge-in warning over-claims for a fast-forwarded merge-in

**Location:** internal/fabriccli/merge_verbs.go:30
**Issue:** `gitrepo.MergeStart` merges with `--ff --no-commit`, so a merge-in into a task warp branch that is an ancestor of the parent fast-forwards: HEAD moves with no merge commit, `AlreadyUpToDate` and `Committed` are both false, and the warning is emitted.
The warning claims "`lyx webster record-batch` tolerates this parent merge", but record-batch refuses a fast-forward past the fork's reported head (a fork that committed nothing, reporting its `StartSHA`).
In that case the warning tells the operator the opposite of what will happen.
**Fix:** Word the warning as the rule record-batch applies — it tolerates a parent merge commit after a fork's commit but refuses a fast-forward past it — so it is true for every case that emits it.
Do the same in the `Long` help paragraph.

### [LOW] New test comments are fixed-column wrapped

**Location:** internal/websterengine/recordbatch_test.go:938
**Issue:** The convention covers all Go comments.
The new helper and test doc comments are fixed-column wrapped with mid-phrase breaks:
`gitwrapMergeSide`, `gitwrapConflictingMerge` and the changed header line in gitwrap_test.go;
`parentMerge`, `snapshotRecordState`, `TestRecordBatch_ParentMergeAfterForkCommit`, `TestRecordBatch_ParentMergeSymbolsAreNotTheBatchsOwn` and `TestRecordBatch_MergeInProgressRefusedThenSucceeds`;
`recoverAtReportHead` and the three new `TestRecoverBatch_*` docs;
parentmerge_test.go's header and test doc;
mergein_webster_integration_test.go's header and `seedWebsterState`.
**Fix:** Re-break each semantically.

### [NIT] merge-in webster tests do not pin the warning content

**Location:** internal/fabriccli/mergein_webster_integration_test.go:66
**Issue:** Card 6 asks that `warnings` be "exactly the one warning", and that the conflict envelope carry "the same `warnings`".
The helper accepts any single non-empty string.
**Fix:** Assert that the entry names Webster being mid-run and `record-batch`.

### [NIT] Two reconcileReportHead tests assert less than card 1 names

**Location:** internal/websterengine/gitwrap_test.go:223
**Issue:** Card 1: a non-merge commit "with merges on either side of it → refused, error names both SHAs and states the rule".
The between-merges subtest checks only `err != nil`.
`TestReconcileReportHead_ReportHeadIsMergeCommit` checks that the report head is absent at the first and middle list positions but not at the last.
**Fix:** Assert both SHAs and the rule in the between-merges refusal, and assert the exact walked list `(merge2)` in the report-head-is-merge case.

### [NIT] Mid-merge refusal prefix differs from its sibling refusal

**Location:** internal/websterengine/gitwrap.go:41
**Issue:** The operator-facing mid-merge refusal starts `websterengine:`, while the head refusal the same verbs return (and the other record-batch refusals) start `webster:`.
**Fix:** Prefix the mid-merge refusal `webster:`, keeping `websterengine:` for the wrapped probe failure, as `headSHA` does.

### [NIT] Redundant newHead alias in the parent-merge record test

**Location:** internal/websterengine/recordbatch_test.go:985
**Issue:** `newHead := merge` checks the same SHA twice under two names.
**Fix:** Drop the alias and name the merge as the new HEAD in a comment.
