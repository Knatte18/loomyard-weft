# Card 2 — reedengine: stage a strand's launch line as a script file

**Edit:**
- `internal/reedengine/panebin.go`
- `internal/reedengine/panebin_test.go`

**Create:**
- `plan:internal/reedengine#launchScriptDir` -> `func launchScriptDir(stateDir string) string`
- `plan:internal/reedengine#launchScriptPath` -> `func launchScriptPath(sh shell.Shell, stateDir, strandGUID string) string`
- `plan:internal/reedengine#stageLaunchScript` -> `func stageLaunchScript(sh shell.Shell, stateDir, strandGUID, composedLine string) string`

**Uses:**
- `internal/shell#Shell`
- `internal/reedengine#composePaneLaunchLine`
- `internal/reedengine#Engine.stateDir`

**Intent:**
Add the launch-script file mechanics to `panebin.go`, the pane-binary seam owner, without wiring them into the chokepoint yet (card 4 does that).
Keeping them in `panebin.go` follows the decision that the script write is part of producing the send-keys payload, so it belongs to the same seam.

- `launchScriptDir(stateDir)` returns `<stateDir>/reed/launch`.
  `stateDir` is the value `Engine.stateDir()` returns, `<AnchorPath>/.lyx`, whose `.lyx` segment already comes from `lyxdirs` (Lyxdirs Single-Declarer Invariant);
  the `reed` and `launch` segments are reed's own relative subpath, declared as named package constants rather than inline literals.
- `launchScriptPath(sh, stateDir, strandGUID)` returns `launchScriptDir(stateDir)` joined with `strandGUID + sh.ScriptExt()`.
- `stageLaunchScript(sh, stateDir, strandGUID, composedLine)` writes `composedLine + "\n"` to `launchScriptPath(...)` and returns `sh.Source(path)` as the send-keys payload.
  The content is the composed line byte-for-byte plus one trailing newline: no header comment, no shebang, no reformatting.
  An empty `composedLine` still writes a file (holding the newline alone) so every launch takes one uniform path.
  The write is atomic and regenerated on every call: `os.MkdirAll` the directory (`0o755`), `os.CreateTemp` in that same directory, write, close, `os.Chmod` to `0o644`, then `os.Rename` over the final path, removing the temp file on any failure.
  A private helper in this file does the write; `fsx.AtomicWriteBytes` is not reused because its `os.CreateTemp` file keeps mode `0o600`.
  The file is sourced, never executed, so it carries no exec bit.
  On any write error it logs `logger.Warn` naming `"strand"`, `"path"` and `"err"` and returns `composedLine` unchanged, so a cosmetic feature never fails a strand launch — the same degrade rule as `composePaneLaunchLine`'s executable-error branch.

Update `panebin.go`'s file doc to say the file also owns the per-strand launch script the composed line is written to.

Tests in `panebin_test.go`, each rooted at a `t.TempDir()` state dir:

- For both `shell.Posix()` and `shell.Pwsh()`: the returned payload equals `sh.Source(<stateDir>/reed/launch/<guid><sh.ScriptExt()>)`, and the file's content equals the composed line plus `"\n"`.
- A second call for the same GUID with a different line replaces the content, and the launch directory holds exactly one entry afterward (no leftover temp file).
- On non-Windows hosts, the written file's permission bits are `0o644`.
- An empty `composedLine` writes a file holding `"\n"` and returns the source statement.
- Unwritable directory: a regular file created at `<stateDir>/reed` makes `MkdirAll` fail; the return is `composedLine` byte-for-byte, and `captureLogOutput`'s buffer names the strand GUID and the script path.

**ImpactSummary:** Additive to `panebin.go`; no caller until card 4 wires `stageLaunchScript` into `launchStrandLocked`, so no existing behavior changes.

**Commit:** 2: reedengine: stage a strand's launch line as a script file
