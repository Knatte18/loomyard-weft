```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness of the diff. Each line below is a function or branch, the case I ran, and whether it was correct.

**List**

- `ResolvePullRequest`, `merge-in` branch, parent `required` with GitHub rule true: returns `TrueMerge` with no `Open`, `Halt` or `MergeAtOnce`. Correct.
- `ResolvePullRequest`, beam with `merge-in`: `isBeam` clears the field first, so the field is ignored. Correct.
- `ResolvePullRequest`, `merge-in` with no commits beyond the parent: still `TrueMerge`. Correct, the merge commits nothing.
- `resolveWithParentRule`, `TrueMerge`: returns at once, so no GitHub rule read. Correct.
- `PRReview.decide`, `merge-in` entry: `!decision.Open` goes to `closeRecordedPullRequest`, which closes any recorded pull request and returns Done. Correct.
- `pullRequestLanding`, non-beam, no record: `landingForDecision` returns an empty landing and `trueMerge` is set afterwards. Correct.
- `pullRequestLanding`, non-beam, stale record: `dropRecordedPullRequest` closes and removes the record, and the landing is zero (`trueMerge` set afterwards). Correct. On a stop return, the extra `trueMerge` write is harmless.
- `pullRequestLanding`, `recordedBeamLanding` and its re-resolve at `finalize_pr.go:312`: the beam path passes `""` as the field and `trueMerge` is never set. Correct, because a beam's landing never reads the entry field, so `merge-in` cannot reach it.
- `landUnderLock`, `Squash: (Config.Squash || atOnce) && !trueMerge`: with `trueMerge` true and no PR, `atOnce` is false and `squash` is false. Correct. I did not confirm that `Merge` with `Squash` false takes the non-squash path in `fabricengine/merge.go`. The grep shows `Squash` is passed through, and the doc says non-squash is `git merge --ff --no-commit`.
- `landUnderLock`, `pushParent(..., atOnce || trueMerge)`: the flag only changes the protected-branch way forward. Correct. The advice text still points at a by-hand pull request merge, which is acceptable.
- `mergeTaskVerb`: `merge-in` and `merge-stage` pass, and `merge --continue` and `--abort` pass. `merge <branch>` returns false, so it stays refused. Correct.
- `mergeTaskVerb`, `merge` with both a branch argument and `--continue`: it returns true, so the guard lets it through. This is bounded because the fabric verb itself should reject mixed forms. Not a defect I could confirm.
- `sandboxedRoleRefusal`, ordering: the bypass-flag checks run before `readerVerb || mergeTaskVerb`, so the exception cannot admit a bypass flag. Correct.
- `IsMergeTaskWriter`, `darn-2` role and `beam-merge-in-2` slug: true. The slug `beam-merge-in-notes` is false (the regex is anchored at the end). Correct.
- `mergeParent`: sets `pull_request` to `merge-in`. `normalizeTask` accepts it, so the filing succeeds. Correct.
- `sketchOf` clears `PullRequest`: a merge task is released directly, so it never becomes a Sketch. No issue.
- `EntryField` in `loomcli/landingdeps.go:361`: reads `task.PullRequest` only, with no project inheritance. A `merge-in` on a task is read as written. Correct.
- `normalizeTask` error text: names `merge-in`, and the test matches it. Correct.
- `mergeParentBody`: defect, see F1.

**Findings**

**F1 — LOW, consistency.** `internal/beamcli/mergeparent.go:126`. The diff deleted the space before the backtick, so the body reads ``Filed by`lym beam merge-parent X`.``. It is a regression from the original ``Filed by `lym ...` ``. `mergeparent_test.go` asserts only `lym fabric merge-in main` and `reviews the merge alone`, so nothing pins the text. Fix: restore the space, giving ``Filed by `+"`lym beam merge-parent %[1]s`"+`.``.

**F2 — LOW, consistency.** `internal/boardengine/task.go:~202`. The `normalizeTask` doc comment still says "A PullRequest other than empty, on and off is refused", but `merge-in` is now accepted. Fix: add `merge-in` to that sentence.

**F3 — LOW, design.** `internal/fabriccli/roleguard.go` `mergeTaskVerb`. A `merge <branch> --continue` or `merge <branch> --abort` invocation is classed as allowed because only the flags are read, and the branch argument is not checked. Fix: also require `len(args) == 0` for the `merge` form. This needs the args passed into `mergeTaskVerb`. If the verb already rejects a branch with those flags, downgrade this to a NIT.

**Tool calls**
1. Read `internal/loomcli/landingdeps.go:340-370`, `internal/boardengine/sketch.go:45-70` and a grep of `fabricengine` for `Squash`. My context lacked how `EntryField` is wired, whether `sketchOf` affects a merge task, and how a non-squash `Merge` behaves.
