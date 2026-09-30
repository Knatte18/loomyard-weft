# Card 4 — reedengine: type the launch script's source statement instead of the full line

**Edit:**
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.addStrandLocked`
- `internal/reedengine#Engine.AddStrand`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedengine/panebin.go`
- `internal/reedengine/spawn_test.go`
- `internal/reedengine/emptycmd_integration_test.go`

**Prosa:**
- `internal/reedengine/doc.go`
- `CONSTRAINTS.md`
- `docs/sandbox-howto.md`

**Uses:**
- `internal/reedengine#stageLaunchScript`
- `plan:internal/reedengine#removeLaunchScripts`
- `internal/reedengine#composePaneLaunchLine`
- `internal/reedengine#executablePath`
- `internal/shell#Shell`

**Intent:**
Make every strand pane show a short source statement as its typed line, with the full composed line written to the strand's launch script.
The chokepoint covers `AddStrand`, `UpdateStrand`'s surface, `Resume`'s `ResumeCmd` replay, and the `--if-absent` relaunch with no per-caller change.

`launchStrandLocked`:
bind `sh := shell.ForGOOS()` once, compose with `composePaneLaunchLine(sh, launchCmd, s.GUID)` exactly as today (so `panebin_enforcement_test.go`'s composes-the-prelude check keeps passing), then pass the composed line through `stageLaunchScript(sh, e.stateDir(), s.GUID, composedLine)` and send its return through `sendKeysLiteralArg` as the `send-keys -l` payload, followed by the separate `Enter` as today.
The payload stays a single line.
Rewrite the comment above the send and the doc comment's pane-binary paragraph: the prelude and the launch command now ride the launch script the send-keys line sources, and a write failure degrades to typing the composed line.
Since the script is written on every launch, a relaunch with a different command (`Cmd` versus `ResumeCmd`) or a different spawning executable always regenerates it.

Failed add — a script whose strand record is never persisted:

- `addStrandLocked` calls `removeLaunchScripts(shell.ForGOOS(), e.stateDir(), []string{guid})` when `launchStrandLocked` fails, before returning the error.
  Both of its callers then drop the unpersisted record (`AddStrand` returns without saving, `ReplaceStrand` cuts it off the table).
- `AddStrand`'s ordinary path does the same for `strand.GUID` when the `SaveState` right after `addStrandLocked` fails.
- `ReplaceStrand` records a failure of its own `SaveState` right after a successful `addStrandLocked` apart from a launch failure, since that path leaves the new strand in `st.Strands`.
  The tail's `reconcileApplyPersistLocked` ends in `SaveState`, so when it returns `nil` the new strand's record was persisted after all and its script stays;
  when that post-add save failed and `reconcileApplyPersistLocked` also returns an error, the record was never persisted, and `ReplaceStrand` calls `removeLaunchScripts` for the new strand's GUID before returning.

Update `panebin.go`'s file doc and `composePaneLaunchLine`'s doc comment: its output is now the launch script's content rather than the send-keys payload itself.
Reword `emptycmd_integration_test.go`'s file doc the same way: an empty-Cmd strand's launch script holds the composed prelude alone, and the send-keys literal is the source statement for it.

Docs, in this same commit per the Documentation Lifecycle:

- `CONSTRAINTS.md`'s Pane Binary Resolution clause gains a bullet: the prelude and launch command ride a per-strand launch script at `<AnchorPath>/.lyx/reed/launch/<guid><ext>` that the send-keys line sources through `internal/shell`'s `Source`;
  the script exists only while its GUID is in reed's strand table;
  a script write failure degrades to typing the composed line plus a named `logger.Warn`, never a failed launch.
- `internal/reedengine/doc.go`'s pane-binary paragraph replaces "riding the same send-keys line the launch command already rode" with the script route: the composed line is written to the strand's launch script, and the typed line sources it in the pane shell's own scope, so the prelude still runs after the pane shell's profile.
- `docs/sandbox-howto.md`'s paragraph on panes the agent spawns says reed writes those statements to each strand pane's launch script, which the pane shell sources.

Semantic line breaks in every changed comment and markdown line.

Tests in `spawn_test.go`, with `executablePath` injected via `withInjectedExecutablePath` and the `e.tmux.execHook` fake:

- Retarget `TestLaunchStrandLocked_SendsThePreludeAheadOfTheStrandCommand`: the first `send-keys` payload is `sendKeysLiteralArg(sh.Source(path))` for `path` under `<e.stateDir()>/reed/launch`, and the script file's content equals `composePaneLaunchLine(sh, launchCmd, s.GUID) + "\n"`.
- Launching the same strand twice, first with a `Cmd` string and then with a different `ResumeCmd` string, leaves the file holding the second composed line.
- With `executablePath` returning an error, the script holds `launchCmd + "\n"` alone.
- An empty launch command writes the prelude alone plus `"\n"`, with no trailing separator.
- A regular file at `<e.stateDir()>/reed` makes the payload the full composed line, and `captureLogOutput` names the strand GUID.
- `addStrandLocked` with a fake whose literal `send-keys` fails returns an error and leaves no file in the launch directory.
- `AddStrand` with a fake whose `split-window` returns a fresh pane id and whose `Enter` send-keys replaces `<e.stateDir()>/reed.json` with a non-empty directory, so `SaveState`'s rename fails after the script is written: `AddStrand` returns the persist error and the launch directory holds no file.
- `ReplaceStrand` with the same failing-save fake: every later `SaveState` fails, so `reconcileApplyPersistLocked` errors too; `ReplaceStrand` returns an error and the launch directory holds no script for the new strand's GUID.

**ImpactSummary:** Changes the typed line of every strand pane on every realizing path through the one chokepoint; `addStrandLocked`'s callers `AddStrand` and `ReplaceStrand` gain a best-effort delete on a failed launch or an unpersisted record only, never changing either op's result or error; selvage, `new-session`, and `io.go`'s `Send` are untouched.

**Commit:** 4: reedengine: type the launch script's source statement instead of the full line
