```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff adds or changes, I ran one concrete case, including an empty, failing or boundary input, and checked the result.

**Cases run**

- **`Finalize.landedDiffEmpty`, no merged head held** (`parentHeadMerged == ""`): returns false. Correct.
- **`Finalize.landedDiffEmpty`, `ChangedPaths` nil**: returns false. Correct.
- **`Finalize.landedDiffEmpty`, `ChangedPaths` fails**: logs at Warn and returns false. Correct, and it fails safe.
- **`Finalize.landedDiffEmpty`, the parent-side merge changed nothing** (parent head before and after are the same, or the merge commit carries no tree change): the path list is empty, so it returns true. Correct.
- **`Finalize.landedDiffEmpty`, a sibling run's uncommitted status record sits in the parent pair of a single-repo hub, and the task landed no work**: returns false. **Wrong.** Step 2b commits the parent's records after `parentHeadMerged` is read, so those `_lym` paths count as changed. See finding 1.
- **`Finalize.landedDiffEmpty`, merge-in-required retry**: `mergeParent` calls `keepParentHead` again, so the base is the fresh parent head. Correct.
- **`Finalize.landedDiffEmpty`, archive path** (finalize.go:847): after `archiveHeldParentMerge`, `recordLanding` re-reads origin's tip, which already holds the task's work, so the diff is not empty. Correct.
- **`Finalize.recordLanding`, head read fails**: returns before `RecordLanding`, so no flag is written. Correct.
- **`Finalize.recordLanding`, `RecordLanding` nil**: `landedDiffEmpty` is never called. Correct.
- **`WriteLandedCommit({abc, true})`**: writes `"abc\nempty-diff\n"`. Correct.
- **`WriteLandedCommit({abc, false})`**: writes `"abc\n"`. Correct.
- **`WriteLandedCommit({"", true})`**: writes `"\nempty-diff\n"`, which reads back as `{Commit: "empty-diff"}`. Wrong in isolation, but unreachable: the only caller passes a commit only after a successful `HeadSHA`. No finding.
- **`ReadLandedCommit`, file missing**: returns the zero value and nil. Correct.
- **`ReadLandedCommit`, old one-line file `"abc\n"`**: returns `{abc, false}`. Correct.
- **`ReadLandedCommit`, CRLF file `"abc\r\nempty-diff\r\n"`**: returns `{abc, true}`. Correct.
- **`ReadLandedCommit`, file is a directory**: returns the error, which battencli logs while leaving the commit unknown. Correct.
- **`battencli.DobbyRuns` landed read**: on an error, `LandingCommit` stays empty and `EmptyLanding` stays false. On success it carries both fields. Correct.
- **`landNoticeLine`, `EmptyLanding` true with commit `0123456789abcdef`**: prints the empty-diff line with the commit cut to 12 characters. Correct.
- **`landNoticeLine`, `EmptyLanding` true with an empty commit**: falls to "commit unknown". Correct.
- **`landNoticeLine`, `EmptyLanding` with a beam member's follow-up**: the follow-up is appended. Correct.
- **loomcli `RecordLanding` seam**: passes commit and flag through to `WriteLandedCommit`. Correct.
- **`LoadConfig` `report_dir`**:
  - `"docs/research/"` loads as `"docs/research"`. Correct.
  - `"../x"` and `"/abs"` are refused. Correct.
  - `""` is skipped. Correct.
  - `"."` loads as `"."`, so the report goes at `<slug>.md` in the root. Acceptable.
  - `"_lym/research"` and `".lym/x"` are accepted. **Wrong** for the key's purpose. See finding 2.
- **`reportPassage`, empty `reportDir`**: returns `""`. Correct.
- **`reportPassage`, `ReadForEngine` fails**: the error is wrapped by `DarnTable`. Correct.
- **`reportPassage`, `"docs/research"` with slug `s`**: `report_path` is `docs/research/s.md`, and the text is led by `\n\n`. Correct.

**Findings**

1. **MEDIUM, design**, `internal/landingshed/finalize.go:751` (`landedDiffEmpty`).
   In a single-repo hub, a landing that carries no work is still recorded as not empty whenever Step 2b committed the parent pair's own run records. That hides the empty-landing flag in the case the board entry targets.
   - **Evidence:**
     - `keepParentHead` sets `parentHeadMerged` inside `mergeAndGate` (finalize.go:1215).
     - Step 2b's `commitParentRecords` then commits sibling runs' uncommitted status records on the parent branch before the parent-side merge (finalize.go:378-383). Its contract is in deps.go:201-209.
     - `ChangedPaths(parentHeadMerged, landing)` therefore lists those records paths, such as `_lym/...`, which a single-repo hub tracks on the code branch. This worktree's tracked `_lym` shows this hub is one.
   - **The summary's claim does not hold:** "Merges in a one-repository hub restore the receiving side's records paths, so a run's records alone never make a landing count as non-empty". It covers only the task's own records, not the parent's records commit made in between.
   - **Fix:** take the emptiness base from the parent head read after `commitParentRecords`, or exclude the records paths from the path list. Add a test row where the parent head moves by a records-only commit between the merge-in and the landing.
2. **LOW, design**, `internal/darnengine/config.go:139-144`.
   `LoadConfig` accepts a `report_dir` inside the records trees (`_lym/...`, `.lym/...`). A report written there is stripped at the merge, or is never tracked, so it never reaches the parent branch, which defeats the key's stated purpose.
   - **Evidence:** the `filepath.IsLocal` check at config.go:140 is the only check. PATTERN-single-repo-hub says merges strip the records paths.
   - **Fix:** refuse a directory under a records root (via `lymdirs` names), with a way forward.

**Tool calls**

1. `grep` and `sed` of `internal/landingshed/finalize.go` lines 330-395 and the `commitParentRecords` call sites. The manifest lacked where Step 2b runs relative to `keepParentHead` and the parent-side merge.
2. `grep` of `CommitParentRecords` in `internal/loomcli/landingdeps.go` and `internal/landingshed/deps.go`. The manifest lacked what that step commits, and where.
