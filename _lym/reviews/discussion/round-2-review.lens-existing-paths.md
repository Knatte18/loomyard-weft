```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: I traced each decision's new state against the code paths that already write, reset or read it.

**Paths traced, per decision:**
- **D1, code-side dirty checks** (`verifyGate.dirtyPaths`/`clean`, `prFindingsVerb`'s `verifytree.DirtyPaths`). Survives. `_lym` in the code worktree is a symlink into the weft worktree, so `git status` there never lists a file in the kept directory.
- **D1, status commit** (`statusCommitPathspec`). Survives once the directory joins the pathspec. `git add <dir>` honours the directory's own `.gitignore`.
- **D1, teardown** (`commitPendingRecords`). Survives: it stages the configured `_lym` names whole.
- **D1, hand-over** (`PathsOutsideRunRecords`). Survives once the directory joins the records set.
- **D2, copy commit** (`pullRequestRecordCommitter`). Breaks as worded: it stages only `PullRequestRecordRel`. The PR-Review row's `CommitRecord` shares the committer, so widening it is harmless.
- **D2, a stopped push or commit.** Survives: the copy rides the next status commit, because the kept directory is in the status pathspec.
- **D3, ask-record writers** (`RecordParentAsk`, `MarkParentAskPosted`, `EndParentAsk`, `ClearParentAsk`, `AnswerParentAskOrUnreached`, `ExpireParentAsk`). Survive: each is one `state.UpdateJSON`.
- **D3, readers of `ask.Report` after a seed** (`ReadPRReviewStanding`, `parentCoverageGate`, `SettleParentAsk`). Break; see finding 1.
- **D3, `reportengine` reads of `parent-review/round-*`.** Survive: the asks subdirectory is outside the reviews root.
- **D4, fixer gate check** (`fixerSession.judge`, for both Finalize and Parent-Align through `newFixerRun`). Survives: the copy is per turn and named by row.
- **D4, `archiveStaleReport`'s rename of an earlier session's report.** Survives: it is independent of the copy.
- **D4, `runFixReview`'s start-failure return.** Survives: it writes no report, which the Bound covers.
- **D4, attempt-record reset or removal** (`RemoveFixerAttempts`, `readRowAttempts` way forward, `ReadFixerAttempts`' entry mismatch). Survives: copies take their own sequence.
- **D5, `landUnderLock` → `recordLanding` → `pushParent` → `pushRecords` → `markTaskDone` → `dropBeamRecord` ordering.** Survives: the row is written before `pushRecords`, and the URL is read before the drop.
- **D5, already-landed path** (`syncParent` fast-forward to upstream, then `Merge` reports already up to date with no record) **and `archiveHeldParentMerge`'s re-read.** Break; see finding 2.
- **D5, `DriftVerdict`.** Survives: it reads the committed drift records.
- **D6, `readLoomRecords` / `selectRuns` partial classification.** Survives: D6 reads Done separately.
- **D7, `PlainArchiveTagsOnOrigin` way forward.** Breaks the absolute wording; see finding 4.

**Findings:**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md#D3`: the seed restores a record whose done asks name reports the new owner does not have, so the PR-Review standing read errors on every call.
   - D3 seeds the live record from the tracked copy but never restores the live report files.
   - Each done ask's `Report` field is the old worktree's `.lym/parent-review/ask-N.report.yaml` path (`internal/burlerengine/parentreview.go:390-394`), and on the new owner it is absent.
   - `ReadPRReviewStanding` returns the `ReadParentReport` error for the latest done PR-Review ask at head (`internal/landingshed/prreviewask.go:118-121`).
   - `parentCoverageGate` turns that read error into a gate error (`internal/burlerengine/parentseat.go:83-86`).
   - `SettleParentAsk` fails with "read the parent report" (`internal/burlerengine/parentreview.go:314-316`).
   - Before D3, a new owner had no record and read ask-again. With the seed, the run errors on every PR-Review call and every round settle after an adopt.
   - Fix: the seed also restores each report copy to its live path and rewrites `Report` to that path, or readers resolve the report through the tracked copy. Add the adopt-then-read case to the D3 criterion.

2. **MEDIUM, design.** `_lym/discussion/decision-record.md#D5`: on the already-landed path with no record, and after `archiveHeldParentMerge`, the parent's head is read as the landing commit even when the parent has moved past it.
   - `syncParent` fast-forwards the parent pair to its upstream before the merge (`internal/landingshed/finalize.go:519`). That upstream may hold the operator's hand merge or an adopter's landing and later landings on top of it.
   - `Merge` then reports already up to date, and `recordLanding` stores `HeadSHA` (`internal/landingshed/finalize.go:742`).
   - `archiveHeldParentMerge` resets to origin's parent and reads `HeadSHA` again (`internal/landingshed/finalize.go:853-854`).
   - In both cases the row names a later commit, its time and its message. That contradicts D5's claim that "a merge the operator made by hand … the row's landing commit names it".
   - Fix: in D5, take the landing commit on those paths as the parent commit that first holds the task's work, not the head. Otherwise state the limitation and add a criterion for it.

3. **LOW, consistency.** `_lym/discussion/decision-record.md#D2`: "The commit the verb already makes stages the PR findings subdirectory whole" describes behaviour the code does not have.
   - `pullRequestRecordCommitter` stages only `PullRequestRecordRel` and returns early when that file has no content (`internal/loomcli/landingdeps.go:528-531`, `:547-549`).
   - Fix: word it as the change: the committer's pathspec gains the PR findings subdirectory.

4. **LOW, consistency.** `_lym/discussion/decision-record.md#D7` and the `PATTERN-pair-remote` change: "No code deletes an archive tag" and "archive tags are never deleted" sit beside a way forward lym prints that deletes one.
   - The way forward `PlainArchiveTagsOnOrigin` prints pushes `:refs/tags/archive/...`, deleting the plain-namespace archive tag on origin after copying it (`internal/fabricengine/cleanupremotewarp.go:374-376`).
   - Fix: bound the rule, so that a plain-namespace tag moved under the archive prefix by that way forward is no deletion. Otherwise state the exception in D7.

Tool calls:
- `verifytree.DirtyPaths` and `verifyGate.dirtyPaths`/`clean`: needed to check whether kept-directory files make the task worktree dirty for the fixer, Finalize and pr-findings dirty checks.
- The code repo's `.gitignore` and the `_lym` entry, plus readers of `ask.Report`: needed to see whether `_lym` is visible to code-side `git status`, and who reads a seeded ask's report path.
- `ReadPRReviewStanding` and `parentCoverageGate` bodies: needed to confirm what a missing report file does to those readers.
