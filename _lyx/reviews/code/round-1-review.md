---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/gitexec/gitexec.go:194
    summary: runCoreWithin doc says every child gets the environment less GIT_DIR and GIT_WORK_TREE, omitting the GIT_OPTIONAL_LOCKS handling this diff adds
    fix: In the runCoreWithin doc comment, reword the line "Every child gets the process environment less GIT_DIR and GIT_WORK_TREE." to say every child gets the process environment less GIT_DIR, GIT_WORK_TREE and GIT_OPTIONAL_LOCKS, with GIT_OPTIONAL_LOCKS=0 set. Keep one sentence per comment line.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: scope
    location: internal/gitexec/gitexec_test.go:412
    summary: the remote spawn path's GIT_OPTIONAL_LOCKS=0 is pinned by no test, though the decision record says local and remote
    fix: In TestRun_RemoteBounds, row "a remote command gets the low-speed environment", add "GIT_OPTIONAL_LOCKS=0" to the list of wanted variables checked against gitErr.Stderr. Add no new test function.
    origin: lens:code-tests
---

### [LOW] runCoreWithin doc comment misstates the child environment

**Location:** internal/gitexec/gitexec.go:194
**Issue:** The diff changes `childEnv` to drop any inherited `GIT_OPTIONAL_LOCKS` and force it to `0`, and updates `childEnv`'s own comment and the package doc.
The `runCoreWithin` doc comment still says every child gets the process environment less `GIT_DIR` and `GIT_WORK_TREE`, which is now incomplete.

### [LOW] Remote spawn path's forced GIT_OPTIONAL_LOCKS=0 has no test

**Location:** internal/gitexec/gitexec_test.go:412
**Issue:** The new `TestRun` row runs through the local branch (`childEnv(nil)`) only.
The decision record's decision says "local and remote", and the change description claims both.
The existing remote env-print row asserts the low-speed variables only, so appending the entry only on the local branch would pass every test.

## Rejected

- Whether `extra` can override the forced `0` (focus item 1): not a defect. The only non-nil `extra` is `lowSpeedEnv`, a fixed slice with no such key.
- Production reliance on a refreshed index (focus item 2): not a defect. A search of non-test code found no `update-index`, `diff-index` or `--refresh` use, and no caller depends on a stat-cache write-back.
- Portability of `alias.lyxenv=!env` and the `t.Setenv` without `t.Parallel` (focus item 3): not a defect. The file already uses the `!sleep 2 && env` alias form, `\r` is trimmed for Windows, and the subtest is non-parallel with the reason stated in the `TestRun` doc comment.
- Scope (focus item 5): not a defect. The diff touches only `internal/gitexec`, and the smoke test and weft write lock are untouched.
- Commit trailer names a different model than the session's attribution reminder: not a finding, no rule in the rubric or repo constraints governs it.
