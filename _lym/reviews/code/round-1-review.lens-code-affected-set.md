```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I looked for everything that reads or asserts it outside the touched lines.

**Per change: the set found, covered or not**

1. **beam.yaml `pull_request` gains `merge`**
   - Validators: the only one is `dobbyengine.LoadConfig`, and the diff updates it.
   - Texts that name the values: the `Config.PullRequest` comment, the `LoadConfig` comment, `template.yaml`, the dobbyengine `doc.go`, `beamModeUnreadableReason` and the `lym config beam` way-forward text. All are updated.
   - The set-path through `lym config beam` goes through the generic `configengine.Spec`, so it needs no change.
   - `README.md:124` names the setting without listing its values, so it stays correct.
   - Nothing under `docs/`, `contracts/` or `plugins/` lists the values.
   - Result: covered.
   - Not checked: `configreg_test.go:86` and `landingdeps_integration_test.go:210`. They only name the key or write a mode, and I did not read them to see whether either lists the old values.
2. **`board.Task` `pull_request` field message**
   - `boardengine/task.go:304` says "pull_request is on, off or empty". That is the entry field, and `merge` is beam.yaml's alone (the `pullrequestmode.go` const comment says so), so it stays correct.
   - Result: covered.
3. **`PullRequestDecision.MergeAtOnce`, consumed by Finalize and PR-Review**
   - Callers: `resolveWithParentRule` has exactly two, `PRReview.decide` and `pullRequestLanding`, and the diff updates both.
   - Docs: the `landingshed` `doc.go`, the `PRReview` doc and the `Finalize` doc all describe the new behaviour.
   - Tests: they cover the resolver, PR-Review and Finalize.
   - Result: covered.
4. **`mergeRefusalReason` signature, from `rec` to `landing`**
   - Its only caller is `mergeOnGitHub`, which the diff updates.
   - Result: covered.
5. **`pushParent` protected-branch way forward**
   - Docs: the `pushParent` comment, `Finalize` doc and `doc.go` are updated.
   - The existing `adopterFirstRepair` tests were adjusted so they expect no repair note when an exact way forward is set.
   - Result: covered.
6. **PATTERN entries**
   - `PATTERN-pull-request-mode` is `PATTERN.md:46`. It says "from the receiving `beam.pull_request` … a yes merges on GitHub", which still holds with `merge` added.
   - `PATTERN-github-auth` allows Finalize to open and merge a pull request, so Finalize opening one stays inside it.
   - `PATTERN-parent-lock`, which keeps merging a run's pull request in Finalize under the parent lock, is respected because the open and the merge both run under the lock.
   - The directive's declared PATTERN changes list was not visible to me, and the diff touches no `PATTERN.md` or `pattern/` file. I therefore cannot confirm that none were declared.
   - Result: covered, with the caveat above.
7. **Commit message**
   - It claims that PR-Review passes at once and that a refused push names the setting. Both match the code.
   - No `Pattern-Departure:` trailer is present, and none is needed.
   - Result: covered.

**Findings**

1. **LOW, `consistency`: `prLanding.merge` comment is stale.**
   - Location: `internal/landingshed/finalize_pr.go:25`.
   - The comment reads "merges the open pull request on GitHub before the parent-side merge". Under `open`, the pull request is not yet open when the landing is decided.
   - The `Finalize` doc at `finalize.go:172-173` still says "a yes needs the recorded pull request and halts without one", and carries the exception only in the added line at `finalize.go:134`.
   - Neither is wrong enough to mislead. This is a near-nit.

**Tool calls**
1. `grep` over `docs`, `contracts`, `pattern`, `PATTERN.md`, `plugins`, `README.md` and `internal` for `pull_request`, `goto PR-Review`, `squash=<` and `protected branch`. My context had no search of the docs and stencils.
2. `grep` of `pull_request` in `configreg_test.go`, `landingdeps_integration_test.go` and `help_test.go`, plus a listing of `docs/`. My context had not read those test files or `docs/`.
