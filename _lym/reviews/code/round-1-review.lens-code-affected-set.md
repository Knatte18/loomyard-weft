```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the diff forces outside the lines it touches.

**Per change, the set found**
- **PATTERN.md entries.**
  - The set is `internal/pattern/check.go`, which caps the overview at `MaxOverviewBytes` (16384) and rule text at `MaxEntryTextChars` (280). `PATTERN.md` is now 9622 bytes, and the new entry lines are well under 280 characters. Covered.
  - Group placement matches D1 and D3, and the `(test)` marks and background links match the declared text verbatim. Covered.
  - Both background files exist and name their scan, and `CLAUDE.md` is unchanged. Covered.
- **`PATTERN-interactive-spawn` scan.**
  - The set is the neighbouring scans that read test-file text. `tierpurity_test.go` bans `exec.Command` tokens in untagged test files, and the new `allowedSpawners` entry is correct and its reason is accurate. The file carries `tier0` and the entries are justified like `cmd/testtiming/spawnscan_test.go`. Covered.
  - `spawnobservability_test.go` scans production files only, so the new test file is out of its scope. Covered.
  - `grep` shows no stale or other reference to the new entries or files outside the six diff files. Covered.
- **`PATTERN-fs-links` scan.**
  - The set is `osImportName`, which exists at `cmd/lym/baretemp_test.go:32` with the same package scope. It returns `""` for a blank or dot import, so the scan skips those files; that is the gap the focus question names (finding 1 below).
  - The only production `os.Symlink` outside `internal/fslink` is `tools/deploy/main.go:295`, matching the D4 allowlist key. Covered.
  - `fslinkonly_test.go` holds the raw `os.Symlink` text only as parsed fixture strings. Whether `tierpurity` flags it depends on whether it contains an `exec.Command` token, and it does not. Covered.
- **Background file claims.**
  - `PATTERN-fs-links.md` says `fslink`'s Windows path "has never been executed" and that `internal/fabricengine`'s package documentation "records what settling it takes". `internal/fabricengine/doc.go:1629-1638` does say this, so the claim is true. Covered.
  - D3 and the acceptance criteria say the entry "states no Windows claim". The background file does make a Windows statement, an accurate one. See finding 2.

**Findings**

1. **MEDIUM, design.** `cmd/lym/fslinkonly_test.go:46` (`osImportName`, `cmd/lym/baretemp_test.go:40`). A file that imports os as `_` or `.` yields `osName == ""`, and the scan returns early. A dot import (`import . "os"; Symlink(...)`) therefore passes. D4 requires "any name the file binds `os` to". This is a missed way to bind `os`, and no fixture row covers it. The code is confined to `cmd/lym` and `fslink` can be bypassed there, though a dot import of `os` is unlikely in practice. A fixer could flag a dot import of `os` as a failure, or add a fixture row and a bound note to the background file.

2. **LOW, consistency.** `pattern/PATTERN-fs-links.md:9` against the declared D3 acceptance criterion ("states no Windows claim"). The last bullet asserts that `fslink`'s Windows path has never been executed and points at fabricengine's docs. The claim is accurate, since `fabricengine/doc.go:1629-1638` says the same. The strict reading of D3 is "no claim", and the entry and the background file do differ. A fixer could keep it as a pointer ("Windows behaviour is unsettled; see `internal/fabricengine/doc.go`") or accept it as is.

3. **LOW, consistency.** `pattern/PATTERN-fs-links.md:3`, "which owns the platform difference between link kinds". This restates `fslink`'s Windows junction versus symlink claim as the entry's rationale, which `fabricengine/doc.go:1639` says is currently unsettled. It is a mild Windows implication, tied to finding 2.

No other member of the affected set is wrong. The commit message has no `Pattern-Departure:` trailer and none is needed, since every declared entry lands with the declared text.

**Tool calls**
1. Grepped Windows and junction text in `fabricengine/doc.go` and `fslink`, to check the "no Windows claim" boundary.
2. Grepped for references to the new entries and files, and the `pattern/check.go` limits. The manifest lacked the caps and any other consumers.
3. Grepped neighbouring test scans and the `tierpurity` allowlist header, to check the `allowedSpawners` neighbours.
4. Grepped fabricengine and `fslink` for the unsettled note, to verify the background file's pointer.
5. Grepped Windows in `fabricengine/doc.go` and read lines 1625-1645, to confirm that the cited doc records the gap.
