```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I walked each function and branch the diff adds or changes through a concrete case.

## Per function and branch

- `secretpath.New`:
  - `nil` gives the zero-like Matcher: correct.
  - `["*.key","[unclosed"]` is refused, naming the pattern: correct.
  - An empty string never arrives, because `strings.Fields` drops it: correct.
- `secretpath.Matcher.Match`, all correct:
  - `config/.env` → `.env`, and `id_rsa.pub` → no match.
  - `""` → base `.` → no match.
  - Hub pattern `config/*` against `config/a/b`: no match, because `*` does not cross `/`. The doc says so.
  - `./.env` → base `.env` → match.
- `Matcher.Hits` and `Describe`:
  - Empty input gives nil hits and `""`, and callers never render nil: correct.
- `Config.SecretPathMatcher`:
  - `"[x"` gives `ErrInvalid` naming the key: correct.
  - `""` gives the shipped patterns only: correct.
- `LoadSecretPathMatcher`:
  - Its load error is passed through: correct.
- `gitrepo.Repo.NewPathsIn`:
  - An empty pathspec returns nil: correct.
  - On an unborn branch, `git diff --cached --diff-filter=A` lists the staged file. I probed this and it is correct.
  - An absent path gives no error and no entry: correct.
  - A git failure returns a wrapped error and the commit is refused, so it fails closed: correct.
  - An untracked directory is listed recursively: correct.
- `gitrepo.Repo.AddedFilesBetween`:
  - Invalid SHAs give `ErrInvalidSHA`: correct.
  - Base equal to head gives none: correct.
  - A multi-commit range that adds `.env` in commit 1 and deletes it in commit 2 gives `[]`. **Wrong for the guard's purpose:** see finding 2.
- `Fabric.Commit` and `refuseSecretPaths`:
  - Tags only, `warpFiles` nil, `weftFiles` nil: both reads return nil and the commit proceeds. Correct.
  - With `SkipGit` the weft side is not checked, and nothing commits there anyway. Correct.
  - A warp `NewPathsIn` failure returns before any lock and changes nothing. Correct.
  - A hit returns before `commitBothSides`, so neither index nor HEAD moves. Correct.
  - The weft check uses the unfiltered `weftFiles`. The `ls-files` and `diff` reads accept absent paths, so this is correct.
- `websterengine.failOnSecretPaths`:
  - A shipped-pattern hit gives a `BatchFailedError` with the reset, delete and recover-batch steps, and the state goes terminal-failed. The record itself is correct.
  - The way forward cannot be followed by its receiver. **Wrong:** see finding 1.
  - The doc comment's claim about the files left on disk is false. **Wrong:** see finding 4.
  - An `AddedPaths` error is returned with nothing mutated: correct.
- `RecordBatch`, where the secret check is placed:
  - A FAILED or NEEDS_PARENT report is still checked: correct.
  - A batch with a correctness finding returns at `recordbatch.go:434-446` before the secret check. **Wrong:** see finding 3.
- `webstercli` record-batch matcher load:
  - A nil loader in standalone mode falls back to the zero Matcher: correct.
  - A load error is refused before the mutation lock, through `configLoadError`: correct.
- `realGit.AddedPaths` and `addedPaths`:
  - Errors are wrapped with the worktree and range: correct.

## Findings

**1. BLOCKING, design: `internal/websterengine/recordbatch.go:619-660` (`failOnSecretPaths`).**
Summary: a batch failed for a secret file is cleared by the Master's normal ladder through `recover-batch`, which builds on the secret-bearing commits and never re-checks. The named reset cannot run inside the run.

- The way forward names `lym webster reset --to batch-start --batch NN`.
- `PlanReset` refuses every target but report-head while `run.lock` is held (`internal/websterengine/reset.go:146-148`). `record-batch` runs from the Master strand inside `lym webster run`, which holds `run.lock` (`internal/webstercli/doc.go:116`).
- The Master stencil's `batch_failed` rung says to run `lym webster recover-batch &lt;NN&gt;` (`contracts/stencils/webster/webster-template-master.md:177`). Its only exception is a plan edit.
- `recover-batch` proceeds from a failed batch, and the diff adds no secret check to it.
- In the standard loop, the secret commit therefore stays in the branch's history and is recorded once recovery finishes. The guard is bypassed.

**2. BLOCKING, design: `internal/gitrepo/gitrepo.go:367-406` (`AddedFilesBetween`), used by `failOnSecretPaths`.**
Summary: the check compares the start tree with the head tree, so a secret added by one batch commit and deleted by a later one passes.

- Webster judges the batch's commits.
- A file committed and later removed still sits in history and is pushed with the branch.
- The board entry asks to refuse "a commit that adds a path matching". Each commit's additions need checking, for example per commit over the start..head walk, not the net tree delta.

**3. MEDIUM, design: `internal/websterengine/recordbatch.go:434-446`.**
Summary: a batch with any correctness finding returns through `failOnCorrectness` before the secret check runs, so its secret file is never named.

- That failure's way forward is plain `recover-batch`, which does not check secrets.
- The secret survives in the branch, independent of finding 1.

**4. LOW, design: `internal/websterengine/recordbatch.go:622-623`, the doc comment, and the step `"delete … from the worktree"` at line 643.**
Summary: the comment says the reset "leaves each file on disk untracked". That is false.

- The reset moves tracked files back to the target (`internal/webstercli/reset.go:32-36`).
- A file that a discarded commit added is tracked in HEAD, so `reset --keep` and `--hard` both remove it from disk.
- Step 2 is therefore a no-op that names files which no longer exist.

**5. LOW, design: `internal/secretpath/secretpath.go:656-668` (`Match`).**
Summary: matching is case-sensitive, so `.ENV`, `ID_RSA`, `Secrets.yaml` and `Credentials.json` pass.

- On case-insensitive filesystems such as macOS and Windows these are the same secret files.
- The entry's patterns are literal, so this is a look-alike gap, not an answer-key violation.

Tool calls:
1. Grepped `StartSHA` writers, `WayForward` users, `refuseBatchStart` and `reconcileReportHead`. I needed to know whether `StartSHA` can be empty and whether parent merges fall inside the start..report-head range. Neither is a finding: `StartSHA` is always set, and merges are tolerated only above report-head.
2. Grepped `RunActive`, `run.lock` holders and the Master stencil's `batch_failed` rung. I needed to know whether the reset way forward can run inside a run and what the Master does with a `batch_failed`.
