```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 and F2 test and stand-in changes through the fix-code-correctness lens.

**Findings**

1. LOW. The doc comment of `landInLinked` in `internal/gatecli/testmap_integration_test.go`.
   - The second line is `It removes the file testMapRemoveFileEnv names, else changes the file testMapLandFileEnv names, else the file `<pkg>.go`.`
   - It is 121 characters of text, and the prose rule breaks at 120.
   - Evidence: I counted it as 48 + 48 + 25 characters.
   - Fix: split it at the second comma, before "else the file".

**Traced and correct**

- **F1, `ASkipAfterTheTestsCountsAgainstTheCap`** (cap 1, `dirtier` then `follower`):
  - `dirtier` is named, so it is rebuilt, its tests dirty `scratch.txt`, and `mapPackage` skips it with `return false`, so it counts.
  - `follower` is never reached, which gives `Unbuilt: 1` and a single skip.
  - If that branch returned true, `built` would be decremented and `follower` would be tried, so `follower` would show a before-build skip and `Unbuilt` would be 0. The test would then fail.
  - The `lym gate test` run passed.
- **F2, `Added` row:**
  - It runs `appendAndCommit(linked, "landed.txt")`, which creates the file and commits it.
  - `before.hashes` lacks `landed.txt` and `after.hashes` has it, so only the added-file loop in `changedFiles` flags it.
  - The cause reads `linked/landed.txt`, because the linked package's `dir` is `linked` and its name differs from the own import path.
  - Dropping that loop leaves `changed` empty, so `dirtier` would build and the test would fail.
- **F2, `Removed` row:**
  - `git rm -q -- landed.txt` followed by `commit -- landed.txt` works, and the file was tracked from the `Added` row.
  - The first loop in `changedFiles` flags it, since `after.hashes[rel]` is `""` and differs from the hash. Dropping that comparison would fail the test.
  - The `Removed` row depends on the `Added` row's state; a comment says so.
- **Callers of `appendAndCommit`:** the `Moves`, `Lands` and `LandsLinked` stand-in cases still behave as before, now through `gitSteps`. The default `LandsLinked` path (no `testMapLandFileEnv`) is unchanged, and the existing `ALinkedFileChangedUnderTheTestsWritesNoMap` passes.
- **Failure of the stand-in's calls:** the `git add`, `git rm` and `git commit` failures all return an error, and `runTestMapStandIn` turns that into exit 1.
- **Full run:** 31 `TestGateTestMap` subtests passed in the full tier-1 run.

Tool calls made: none; the context already held the changed files and the gate output.
