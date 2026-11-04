```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's changed code through the fix-code-correctness lens.

**Findings**

1. **LOW**, `internal/vscode/config.go`, `applyMarkdownPreview`: a `null` association value defeats the fix.
   - The guard `present && !isMap` returns for `"workbench.editorAssociations": null`. A null leaves the setting unset in VS Code, and no entry is lost by setting it.
   - With render on, the preview entry is therefore never written and markdown does not open rendered.
   - Input run: `{"workbench.editorAssociations": null}` with render on leaves `null`, where the old code produced a map. The array form is the only case the review named, so this is a narrow regression.

2. **LOW**, `internal/ideengine/spawn_clean_integration_test.go`, `checkPrimeNameFailureOpensBareFolder`: the added rows pass with the skip branch removed.
   - The skip branch is `if beamName == "" { ...; continue }` in `filesToOpen`.
   - Without it the entry becomes `".md"`, joined under the anchor, and `os.Stat` on that absent path skips it anyway.
   - `opened` stays empty either way. Only the log assertion on the `taskBeam` warning pins the behaviour, and that warning comes from `taskBeam`, not from `filesToOpen`.
   - A row that creates a file at `<anchor>/.md` would fail with the defect in place.

3. **LOW**, `internal/ideengine/spawn.go` (`seed` and `OpenTask`, F7): no test fails if `ConfigDirs(own)` is reverted to `ConfigDirs(l)`.
   - Every ideengine check passes `l` as the hub location, and `own` resolves to the same Bolt board, so the two stacks give the same result.
   - Only a local override under the opened worktree's anchor would tell them apart, and no row writes one.
   - The review proposed no test for F7, so this is a coverage observation, not a missed instruction.

**Checks that held**
- In `seed` and `OpenTask`, `own` is not nil on the paths that use it. Both functions return early when `NamedWorktree` errors.
- The other callers of `applyMarkdownPreview` and `WriteConfig` are unaffected, since the signatures did not change.
- The F3 loop fails with the defect in place. If the load ran after the write, `settings.json` would exist and the stat assertion would fail.
- The F4 and F2 rows fail if `{beam}` resolves to the wrong name, because the expected paths embed the prime name or the parent slug.
- The beam-check cleanup removes the child pair before the file cleanups, because cleanups run last-in first-out, and the tier-1 run passed.

**Extra tool calls:** none.
