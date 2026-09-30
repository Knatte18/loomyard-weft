# reed: send a short launch-script line instead of the full command

Every strand pane used to show the full composed launch line as typed input: the pane-binary prelude (`export PATH=…; export LYX_BIN=…`) chained onto the caller's launch command.
The operator found this noisy.
reed now writes the composed line to a per-strand launch script and types only a short source statement, for example `. '<AnchorPath>/.lyx/reed/launch/<guid>.sh'`.
The pane shell runs the same statements in its own scope as before, and the full line stays inspectable on disk.

The change sits at the single chokepoint every strand-realizing path goes through (`launchStrandLocked`), so `AddStrand`, `UpdateStrand`'s relaunch, `Resume` and the `--if-absent` relaunch are all covered with no per-caller change.
The launch command itself is untouched, so `claudeengine` and shuttle need no change.

This task resolves no tracked issue, so there is no `Closes` line.

## Design decisions

- **reed owns the script, not shuttle.**
  Only reed holds the full composed line, and strands need not be shuttle runs.
  reed writes an opaque string it never interprets, so the Shuttle Provider-Seam Invariant holds.
- **Two new `internal/shell` mechanics**, `Source(path)` and `ScriptExt()`, for both dialects, so reed emits no raw dialect syntax.
  POSIX sources with `. '<path>'` and uses `.sh`.
  pwsh dot-sources a script block built from the file's text, `. ([scriptblock]::Create(<ReadFile(path)>))`, and uses `.ps1`.
  The pwsh form evaluates text instead of running a script file, so no ExecutionPolicy can refuse the launch.
- **Script content** is `composePaneLaunchLine`'s output plus a newline, with no header or shebang.
  It is rewritten atomically (temp file, then rename) on every launch, mode `0o644`, because a relaunch may carry a different command or spawning executable.
  The directory name comes from `lyxdirs`, anchored beside `reed.json`.
- **Write failure degrades** to the old behaviour: a named `logger.Warn` (strand GUID, path, error), then the full composed line is sent.
  A cosmetic feature never fails a launch.
  An unresolvable executable still goes through the script, holding the prelude-less line; an empty launch command yields a script holding the prelude alone.
- **A script lives only while its strand is in reed's persisted table.**
  `removeStrandLocked` (behind `RemoveStrand`, including `--recursive`, and `ReplaceStrand`) deletes the scripts of every dropped GUID.
  `Down` deletes the whole launch directory with `reed.json`.
  A launch whose record is never persisted, including `ReplaceStrand`'s persist-failure path in `replace.go`, deletes its script.
  Deletion is best-effort: a missing file is silent, any other error is a `logger.Warn`.
- **Behavioural difference:** interactive-only processing of typed input (bash history expansion of `!`, the history entry) no longer applies to the launch line.
- `CONSTRAINTS.md`'s Pane Binary Resolution clause, `internal/reedengine/doc.go`, and `docs/sandbox-howto.md` now describe the script route.
  `panebin.go` remains the seam owner and `launchStrandLocked` its only call site.

## To check by hand

- Launch a real strand (for example `lyx reed add`) on POSIX and confirm the pane shows only the short `. '…/launch/<guid>.sh'` line and the agent starts.
- Nothing was run on Windows: the pwsh `Source` form is covered by unit tests only, not by a live pwsh pane.
- The tests that cover `ReplaceStrand`'s persist-failure deletion were not confirmed to fail without the new delete, so review that path in `replace.go`.
