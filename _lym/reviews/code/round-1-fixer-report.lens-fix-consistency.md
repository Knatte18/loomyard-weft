```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens on the F1/F2/F3 change set.

no findings

Checked:
- Every `shuttleengines.New` caller outside tests builds its `Dirs` through `fabricengine.ConfigDirs`, which now fills `Weft`.
- The one exception is `internal/burlercli/wiring.go:211`, which hand-builds `Dirs{Anchor: res.StateDir}`. That is the standalone, non-hub path, where an empty `Weft` and no writable root is the intended behaviour.
- The `Dirs` doc comment, the `ConfigDirs` doc comment and `TestConfigDirs` agree on `Weft`.
- The `Codex.dirs` field comment, the `Prepare` doc comment, `doc.go` and the `summary.md` Design decisions entry all say `Weft` is told and no link is read.
- No leftover mentions of the old link derivation remain in `codexengine`. The only grep hits are the unrelated "symlinks resolved" comments in `home.go`.
- `writableRoots` is removed from `home.go` together with its now-unused `lymdirs` import. The `fslink` and `slices` imports are still used by other code in the file.

Tool calls: one Bash call with greps over the `shuttleengines.New` callers, `burlercli/wiring.go`, leftover link wording in `codexengine`, `Dirs{...}` literals in tests, and the cwd-resolution pattern. My context did not hold the other callers' `Dirs` construction or the pattern text.
