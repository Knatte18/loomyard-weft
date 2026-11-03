```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checked whether the F1 and F2 changes still agree with the docs, comments, tests and call sites around them.

**Findings**

- **LOW** — `internal/shuttleengine/codexengine/doc.go:11` and the `Prepare` doc at `codexengine.go:76`. The new code resolves the records checkout's symlinks before using it as the writable root.
  - Both comments still say the engine "adds the records checkout it is told" and that "the launch line's writable root is the records checkout the engine is told".
  - Neither mentions the resolution or the fallback to the told path with a Warn.
  - The package doc states the trust roots' symlink resolution as a contract (`home.go:34`, `home.go:40`), so a reader would expect the writable root's to be stated there too.
  - The new inline comment at `codexengine.go:118` carries the "why", so nothing is wrong. The two doc comments are just incomplete.

- **LOW** — `ConfigDirs` doc comment in `internal/fabricengine/junctionnames.go`. The new text says a stat failure is "logged for either checkout … and a launch that needs the weft checkout goes without it".
  - That names a Codex launch consumer from inside fabricengine.
  - It reads as the enumerated-consumer pattern the prose rules discourage, and the `Dirs.Records` doc in `configengine/layers.go:22` already says who consumes it.
  - It is still true as written.

**Checked and consistent**

- `TestConfigDirs` in `junctionnames_test.go` pins `Records` and `Board` presence and absence, and the refactored `ConfigDirs` keeps that behavior.
- The Bolt warning text is unchanged, and nothing in the grep results asserts on it.
- The `Dirs.Records` doc comment is untouched and still correct. It says "absolute path"; the resolved path is still absolute.
- The `docs/overview.md` sentence about the writable root ("the records checkout it is told through its config dirs") remains accurate.
- The new test row `RecordsToldThroughLink` sits alongside the existing `RecordsTold` row without disturbing the warnings-empty assertion. The fixture resolves its temp dir (per the review), so `want = records` really is the resolved path.

**Tool calls**

1. `grep` for `Bolt checkout not statable`, `Dirs.Records` and `.Records` across `internal` and `docs`. My context held only the changed files, not their other consumers or docs.
2. `grep` of `configengine/layers.go` for the `Records` field doc, plus `grep` for "weft checkout" and "records checkout" in the codexengine files and `docs`. I needed to know whether the siblings of the changed text were updated.
3. `sed` of `codexengine/doc.go` lines 5–20, plus `grep` for `symlink` and `EvalSymlinks` in codexengine. Both were to compare the package doc against the new resolution behavior.
