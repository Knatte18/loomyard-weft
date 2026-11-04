```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness, run over the diff from my inherited context plus one grep.

**Per-function list (case run → verdict)**

- `ResolvePullRequest`, parent `merge`, field "", origin ok, task has commits: Open and MergeAtOnce true. Correct.
- `ResolvePullRequest`, parent `merge`, no upstream: demanded is true, so Halt. Correct.
- `ResolvePullRequest`, parent `merge`, empty task branch: the no path, MergeAtOnce false. Correct.
- `ResolvePullRequest`, parent `on` with `parentRuleRequired`: MergeAtOnce false. Correct.
- `resolveWithParentRule`, parent `merge`: returns on the first call because the decision is Open, and makes no rules read. Correct.
- `PRReview.decide`, MergeAtOnce yes: Done with no push or GitHub call. This runs after `answerReopen`, so a standing reopen request is still answered first. Correct.
- `pullRequestLanding`, MergeAtOnce with no record:
  - It builds `prLanding{merge, open, atOnce}`.
  - It makes no Status call, and the gate and owner checks still run first. Correct.
- `pullRequestLanding`, record open or closed with `ReviewEnded`: `atOnce` is carried. Correct.
- `pullRequestLanding`, recorded PR merged: `prLanding{}`, which lands through the already-landed path as before. Correct.
- `pullRequestLanding`, recorded PR closed with `ReviewEnded` false, under `merge`: see finding 2.
- `mergeOnGitHub` / `openAtOnce` re-entry cases:
  - Crash after `Create`, before `WriteRecord`: the re-entry finds no record, `FindOpen` finds the PR, `Edit` runs, the record is written and the merge happens once. Correct.
  - Crash after `WriteRecord`, before `Merge`: the re-entry finds a record and an open PR, and merges. Correct.
  - Crash after `Merge`: the re-entry finds the PR merged, so there is no second merge. Correct.
  - A failed `WriteRecord` goes to `errorStop` with no merge. Correct.
  - A `FindOpen` or `Create` failure is judged by `githubFailure`, and nothing is merged. Correct.
- `mergeRefusalReason`, `RefusalMethod` under `atOnce`: names the repository settings and not `squash`. Correct.
- `pushParent` protected branch:
  - `transientFailure` is checked first, so a transient failure is not overridden.
  - `PushFailedError` is unwrapped for the cause.
  - It returns through `stuckOrCancelled` with the same `"error", err` fields as the other push failures. Correct.
- `protectedBranchRefusal`: matches by substring on the lower-cased error text. A non-protection rejection quoting "protected branch" would be misread, which is an unlikely corner. Not a finding.
- `Finalize` parent-side merge after the GitHub merge, config `squash=false` under `merge`: wrong. See finding 1.

**Findings**

1. **MEDIUM, design.** `internal/landingshed/finalize.go:336`, with `finalize_pr.go:273`.
   - `mergeOnGitHub` squashes on GitHub whatever `landing.yaml` says (`squash := Config.Squash || landing.atOnce`).
   - The parent-side merge still uses `MergeOptions{Squash: fz.deps.Config.Squash}`.
   - With `squash=false`, the task head is not an ancestor of the parent after GitHub's squash and fast-forward. The fabric already-landed check is parentage-based (`internal/fabricengine/mergeguards.go:388`, `IsAncestor`).
   - The non-squash parent-side merge then runs a real merge and creates a local merge commit, and `pushParent` pushes it to the protected parent, where it is refused.
   - Fix: pass `Squash: fz.deps.Config.Squash || atOnce` to the parent-side merge as well, or carry the effective method out of `mergeOnGitHub`.
   - No test covers this combination. The `squashOff` case at `finalize_pr_test.go` uses a fake parent that always lands.
   - I did not run this against real git, so the merge-commit step is inferred from the code.

2. **LOW, design.** `internal/landingshed/finalize_pr.go:205-208`.
   - Under `merge`, PR-Review passes at once and writes no record mark.
   - A recorded PR closed on GitHub with `ReviewEnded` false therefore halts with "reopen it on GitHub, then `lym loom goto PR-Review`".
   - That PR-Review step does nothing under `merge`, so the way forward is misleading.
   - A closed PR could be reopened and merged as with `ReviewEnded`.

3. **LOW, design.** `internal/landingshed/finalize.go:633-641`.
   - A refusal that matches the protected-branch markers always names `pull_request=merge` as the way forward.
   - It does so even when the parent is already `merge`. The push there is a no-op after the fast-forward, but any other refusal would be told to set a setting that is already set.
   - This is a corner case with a plausible but unlikely trigger.

**Tool calls**

1. `grep -rn` for already-landed and `AlreadyUpToDate` in `internal/fabricengine/*.go` and `finalize.go`: my context lacked how the parent-side merge decides "already landed", which finding 1 depends on.
