```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the F1 and F2 changes to `testmap_integration_test.go` agree with their surroundings.

I found one finding. I made no tool calls, because my context already holds the changed file.

**LOW** — the `landInLinked` doc comment in `internal/gatecli/testmap_integration_test.go` runs past the 120-character line limit.
- The line reads `// It removes the file testMapRemoveFileEnv names, else changes the file testMapLandFileEnv names, else the file `<pkg>.go`.`
- Its text after `//` is about 121 characters. The prose rule says 119 passes and 120 breaks.
- `lym loom fmt` does not reflow comments, so nothing else would catch it.
- Splitting it at the second `, else` fixes it.

The rest of the change set agrees with its surroundings:
- The `runTestMapStandIn` doc, the constant-block comment (new `testMapLandFileEnv` and `testMapRemoveFileEnv` lines) and `landInLinked` all describe the same precedence: removal, then the named file, then `<pkg>.go`.
- `appendAndCommit` now goes through `gitSteps`, with the same `add` and `commit` arguments as before.
- `ASkipAfterTheTestsCountsAgainstTheCap` has the same set-up and clean-up as its sibling dirtier/follower subtest and the same cause string.
- The `Added` and `Removed` rows share state on purpose, and a comment says so.
- Later subtests do not depend on `linked/landed.txt`, which the `Removed` row deletes.

Neither review fix touched the own-package added and removed branches of `changedFiles`. The review's fix named the linked-package variant, so that is not a finding.
