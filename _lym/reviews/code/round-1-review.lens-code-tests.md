```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**
- Replace writes the file as the record, commits it with the trailer, copies it to the board and notifies each live agent: `TestDecisionReplaceVerb_WritesCommitsAndNotifies`, one row per admitted caller (`decisionreplace_test.go:69`).
- Base-hash refusal, naming no hash: `Refusals/BaseDiffersFromHEAD`, with `notWant` on both hashes.
- Working copy differs from HEAD, record not committed, before approval, Discussion-Write running, caller guards, unreadable file, empty base, unknown label, notice too long, lock failure, filed raddle run, perch at the Raddle-File row: one `Refusals` row each. Every row asserts the record, HEAD and the board are unchanged.
- Webster span: `Refusals/AtAWebsterRow` and `BetweenPlanApprovalAndWebster` use a fake span. The real `InImplementerSpan` is pinned by `TestInImplementerSpan` (`implementerspan_test.go`).
- Section check, decision-id check, carry-over dropped, changed or added, added entry rewritten or inserted: `Refusals` rows, plus `TestCheckCarryOversKept` and `TestCheckAddedEntriesSubset`. The latter covers drops, rewrites, inserts and a fenced heading.
- Commit-error ladder:
  - Rows: empty trailer commit, empty commit failing, commit failing without landing (record restored), `commit_error` with the trailer at HEAD, trailer commit landed without the trailer at HEAD, and that last case with the empty commit failing. All in `CommitOutcomes`.
  - Branches with no row: see findings 3 and 4.
- Decision lock serialising `add`, replace and the carry-over write: `TestDecisionLock_SerializesRecordWriters`, using the real `holdDecisionLock`. `store` taking the lock: none, see finding 5.
- `recordHistory` cut: `TestRecordHistoryCut` covers drop, replaced bytes landed before the trailer commit, an entry added after the replace then dropped, an empty trailer commit over a swept replace, another record's trailer, newest own-path trailer, and a sum matching no version. `TestRecordHistory_CutsAtTheNewestReplace` pins it over real commits. A malformed trailer value: none, see finding 6.
- `AnchoredFileFirstParentHistory`: `TestAnchoredFileFirstParentHistory` covers empty history, newest-first order with positions counting an unrelated commit, and a deletion being skipped. Merge and root commits: none, see finding 6.
- `TrailerCommits` and the `FindTrailerCommit` refactor: `TestTrailerCommitsListsKeyCarriers` (with an unborn weft) and the updated `TestTrailerCommits` rows with positions.
- `CommitEmptyWithTrailers`: `TestCommitEmptyWithTrailers` covers an unchanged file with nothing else staged, an unchanged file with another file staged (it stays staged), the tree equalling the parent's, and an invalid key refused. `SkipGit` and the `Warp-SHA` line: none, see finding 6.
- `gitrepo.CommitPathAllowEmpty`: `TestCommitPathAllowEmpty` covers unchanged, another path staged, changed, and a path matching nothing. The pinned-method list in `gitrepoboundary_test.go` is updated.
- D3 copy gating: `TestDecisionCalls_RefreshEveryStoredRecord` covers add, replace and store on a raddle run of a promoted Sketch, a run with no stored record, and a re-entered Discussion segment. The beam-run row is vacuous, see finding 1.
- D3 filed refusal: `TestDecisionCalls_RefusedOnAFiledRaddleRun` covers four filings across add, replace and store. Error paths of the new seams: none, see finding 6.
- Parent directive quoting the replace notice opening: the updated `TestDecisionNoticeOpening_NamedByParentDirective`.

**Findings**

1. MEDIUM, design. `internal/loomcli/decisionreplace_test.go:562`. The "AddOnABeamRun" row sets `boardStored=false`, which is the default, so it duplicates "AddOnARunWithNoStoredRecord". The beam exclusion is the `record.project → false` branch of `boardRecordStored` in `internal/loomcli/decision.go` `preRun`. No test reaches it, so the D3 criterion "`add` on a beam run leaves the project entry's stored record unchanged" is unpinned. Fix: drive the real closure with a project record, or drop the vacuous row.

2. LOW, design. `decision.go` `preRun` real closures `headCarriesReplace`, `raddleBoardFiling`, `raddleEntryTask` and `boardRecordStored`. Only fakes reach them. Unpinned:
   - `headCarriesReplace`'s match on the newest trailer commit being HEAD with an equal value;
   - `raddleBoardFiling`'s absent or unparseable proposal and its part slugs;
   - the not-a-task reading.

3. LOW, design. `decisionReplaceVerb`'s `rowWritten`/`checkDarnRecord` branch and its "no decision record at path" refusal (`decision.go`, replace verb). `Refusals` has no row for either, though D1 says replace takes `add`'s refusals (a Record-Write run whose record is uncommitted or differs).

4. LOW, design. `landReplace` and the write step have no row for:
   - the `writeRecord` failure with restore;
   - `committed()` unreadable after a commit error;
   - the `headCarries` error;
   - restoring the record failing.
   Each is a refusal with its own way forward, which `PATTERN-refusal-way-forward` asks to be reached by a test.

5. LOW, design. `decisionStoreVerb` now takes the decision lock. No row asserts the lock is taken and released, or that a lock failure refuses with `decisionStoreRetry`. `TestDecisionLock_SerializesRecordWriters` covers only add and the carry-over write.

6. LOW, design. Small gaps with no row:
   - uppercase or whitespace-padded `--base-sha256`, which the verb lowercases and trims;
   - `parseRecordReplaced` on a value that is not three fields;
   - `CommitEmptyWithTrailers` with `SkipGit`, and its `Warp-SHA` line;
   - `AnchoredFileFirstParentHistory` over a merge or root commit, which its doc comment defines;
   - `recordHistory`'s fallback to the whole history when the cut is not found. `TestRecordHistoryCut` re-implements the glue inline and the integration test never reaches the fallback.
   - the D2 acceptance that validate-discussion and the discussion gate read the cut history: the callers `wiring.go` 418 and 895, `darnwiring.go` 165 and `decision.go` 1550 are unasserted. They all share `recordHistory`, so the risk is low.

Tool calls: none. My context already held every file the lens needed.
