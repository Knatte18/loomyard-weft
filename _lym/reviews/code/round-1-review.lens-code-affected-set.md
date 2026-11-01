```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. What the rename forces outside the touched lines.**

I found no findings.

Per change, the set I found and whether it is covered:

1. **`lymcwd.BoltDirName` changed to `_bolt`** (`internal/lymcwd/anchor.go:25`).
   - Readers: `fabricengine.BoltDirName` aliases it (`junctionnames.go:99`), as do `BoardDirName` (`:104`) and the reserved-name list (`:222`).
   - Every other consumer I grepped reads the constant: `configcli`, `fabriccli`, `fabricengine` (`clone.go`, `cleanup.go`, `pull.go`, `boardweft.go`, `codebinding.go`), `ideengine` tests and `websterengine` tests.
   - No second spelling of the directory name remains in lym code.
   - Covered.
2. **Geometry enforcement tables and near-miss fixtures.**
   - `enforcement_test.go` swaps the policed token to `_bolt`, its owner row to `internal/lymcwd`, and the `_boardroom` fixture to `_boltroom`.
   - `junctionnames_test.go` also gets `_boltroom`.
   - Covered.
3. **Residual `_board` spellings.**
   - A tree-wide grep, excluding `.git`, `.lym` and `_lym`, finds `_board` only in `tools/boltrename`.
   - Tokens with a letter before `_board` (`custom_board`) are intentionally kept.
   - Covered.
4. **Stored absolute or relative paths into the checkout.**
   - The board path comes from the `--board-path` flag and never from a config file (`internal/boardengine/config.go:25-29`, `yaml:"-"`).
   - So no config file in the hub holds `_board`, and `tools/boltrename` not touching config is correct.
   - The files that do hold the name (reed commands, `.vscode`, workspace files) are covered by the glob list in `renameBolt`.
   - Covered.
5. **Docs, SVG, sandbox suites and research records.**
   - The scripted swap touched all of them; the residual grep finds none left.
   - The `calls.tsv` and `transcript.jsonl` edits are a lens-adjacent concern for the handler's non-Go-change focus. I did not judge them.
6. **PATTERN.md and `pattern/`.**
   - Neither mentions `_board` or `BoltDirName`.
   - The directive declares no PATTERN changes, so nothing is owed.
   - Covered.
7. **Commit messages and `_lym/landing/summary.md`.**
   - Both match the diff on what I checked. That covers the `lymcwd.BoltDirName` value, the `_boltroom` rename, the tool's refusals and stops, and the `git grep` claim.
   - No `Pattern-Departure:` trailer is present or needed.

Tool calls (2 calls, both in a single message that ran two commands each; each was for something the manifest lacked):
1. `grep` of `internal/boardengine`, `ideengine` and `lymcwd` for relative `../` paths and `BoltDirName`, plus the `board-path` users. I needed to know whether a config file stores a path into the checkout.
2. `cat _lym/landing/summary.md` plus a `grep` of `PATTERN.md`, `pattern/` and `CLAUDE.md` for `_board` and `BoltDirName`. My context had neither.
3. `grep` for `path: ../` literals and `Path` in `internal/boardengine/config.go`. This confirmed the board path is flag-only.
