# reed: send a short launch-script line instead of the full command

## Goal

Every strand pane reed realizes currently shows the full composed launch line as typed input: the pane-binary prelude (`export PATH=…; export LYX_BIN=…`) chained onto the caller's opaque launch command (for shuttle, the `CLAUDE_CODE_FORK_SUBAGENT=1 'claude' "$(cat …/prompt.md)" --session-id … --settings … --model … --effort … --dangerously-skip-permissions` line).
The operator finds this noisy.
reed instead writes the composed line to a per-strand launch script and types only a short source statement for it.
The pane shell runs the same statements in its own scope as before, and the full line stays inspectable on disk.

## Scope

In:

- `internal/reedengine`'s strand-launch chokepoint (`launchStrandLocked`, reaching `composePaneLaunchLine` in `panebin.go`): write the composed line to a launch script, send the source statement instead.
  Every strand-realizing path goes through this chokepoint — `AddStrand`, `UpdateStrand`'s relaunch, `Resume`'s replay of `ResumeCmd`, and the `--if-absent` relaunch — so all of them are covered with no per-caller change.
- `internal/shell`: two new dialect mechanics — the statement that runs a script file's contents in the current shell, and the dialect's script file extension.
- Deleting launch scripts whenever their strand records leave reed's state: strand removal (`RemoveStrand`, including a recursive subtree, and `ReplaceStrand`'s removal of the replaced strand), `Down`, and a launch whose strand record is never persisted.
- Docs and invariants: `CONSTRAINTS.md`'s Pane Binary Resolution clause, `internal/reedengine/doc.go`'s pane-binary paragraph, `panebin.go`'s file doc, `docs/sandbox-howto.md` where it describes how the prelude reaches the pane.

Out:

- Any change to what the launch command contains — `claudeengine`'s `buildLaunchCmd`/`buildResumeCmd` are untouched, and so is shuttle.
- Selvage's pane and the `new-session` first pane (already exempt from the pane-binary seam).
- `io.go`'s text-send path (`Send`/keys to a live strand) — it types agent input, not a launch line.
- An opt-out config key to restore the full typed line.

## Decisions

- **reed owns the launch script, not shuttle.**
  Rationale: only reed holds the full composed line (the prelude is joined at reed's chokepoint), reed replays `ResumeCmd` on its own, and strands need not be shuttle runs (manual `lyx reed add`, sandbox).
  The Shuttle Provider-Seam Invariant stays intact: reed writes an opaque string it never interprets.
- **Script location: `<AnchorPath>/.lyx/reed/launch/<strand-guid><ext>`**, with `<ext>` `.sh` on POSIX and `.ps1` on Windows.
  Rationale: never-tracked state belongs under `.lyx` (Durable-vs-Ephemeral State Invariant), anchored beside reed's existing `reed.json` in `stateDir()`;
  one file per strand keyed by GUID makes each file identifiable and keeps concurrent strands apart.
  The directory name `.lyx` is taken from `lyxdirs`, never spelled literally (Lyxdirs Single-Declarer Invariant).
- **The typed line is the dialect's source statement on the script's absolute, quoted path** — POSIX `. '<path>'`;
  pwsh dot-sources a script block built from the file's text, `. ([scriptblock]::Create(<ReadFile(path)>))`, reusing the dialect's existing `ReadFile` mechanic.
  Rationale: both forms run the statements in the pane's own shell scope, so `PATH`/`LYX_BIN` land in the pane session exactly as when typed, and the prelude still runs after the pane shell's profile.
  The pwsh form evaluates text rather than running a script file, so no ExecutionPolicy (`Restricted`, `AllSigned`, a Windows PowerShell 5.1 pane shell's client default) can refuse it — the same never-fail-a-launch rule as the write-failure degrade and Pane Binary Resolution's executable-error degrade.
  The absolute path is robust against `PaneCwd` and `AnchorPath` differing.
- **Both new mechanics live in `internal/shell`:** a `Source(path)` statement and a script-extension accessor, implemented for both dialects.
  Rationale: Shell Mechanics Seam — reed emits no raw dialect syntax, and the extension is dialect knowledge like the rest.
  The dialect stays `shell.ForGOOS()`, the same selector already shared by the prelude and the launch command.
- **Script content is the composed line exactly as today (`composePaneLaunchLine`'s output) plus a trailing newline.**
  Rationale: the pane shell executes the same statements it used to receive by send-keys, in its own scope, so exports, `PATH`, working directory and exit status match the typed line;
  interactive-only processing of typed input (bash history expansion of `!`, the history entry) no longer applies, which only removes ways the typed line could misparse.
  No header comment, no shebang, no reformatting of the opaque launch command.
- **Written on every launch, atomically** (temp file in the same directory, then rename), mode `0o644`.
  Rationale: a relaunch may carry a different command (`Cmd` vs `ResumeCmd`) or a different spawning executable, so the file is always regenerated from the current launch;
  rename keeps a pane from ever sourcing a half-written file.
  The file is sourced, never executed, so it needs no exec bit.
- **Write failure degrades to today's behaviour:** if the directory or file cannot be written, reed logs a named `logger.Warn` (strand GUID, path, error) and sends the full composed line as before.
  Rationale: mirrors the existing executable-error-warns-and-degrades decision in `panebin.go` — a cosmetic feature must never fail a strand launch.
- **An unresolvable executable still goes through the script.**
  The existing degrade (no prelude, `logger.Warn`) happens first, and its prelude-less line is what gets written.
- **An empty launch command still gets a script** holding the prelude alone.
  Rationale: one uniform path through the chokepoint; the empty-cmd behaviour (prelude only, no trailing separator) is preserved inside the file.
- **A launch script exists only while its GUID is in reed's persisted strand table.**
  Every path that drops strand records deletes their scripts:
  the shared removal step behind `RemoveStrand` and `ReplaceStrand` deletes the script of every GUID it drops, subtree-wide;
  `Down`, which deletes `reed.json`, deletes the whole launch directory with it;
  a launch that wrote a script but whose strand record is never persisted (a failed add) deletes that script.
  Deletion is best-effort: a missing file is silent, any other delete error is a `logger.Warn` and never fails the operation.
  Rationale: GUIDs are unique per strand, so any path that forgets its strands without deleting their scripts orphans them permanently;
  `Down` leaves nothing for `Resume` to relaunch, since it deletes the state file.
  While a strand exists its script stays inspectable.
- **The composition stays in `panebin.go`, reached only from `launchStrandLocked`.**
  Rationale: Pane Binary Resolution names `panebin.go` as the seam owner and `launchStrandLocked` as its only call site; the script write is part of producing the send-keys payload, so it belongs to the same seam.

## Constraints

- Pane Binary Resolution: `panebin.go` owns the seam, `launchStrandLocked` is its only call site, `panebin_enforcement_test.go` keeps passing; the clause's text changes to say the prelude rides the launch script the send-keys line sources.
- Shell Mechanics Seam: every shell token, including the source statement and extension, comes from `internal/shell`, which stays stdlib-only.
- Shuttle Provider-Seam Invariant: no Claude specifics enter `reedengine`.
- Durable-vs-Ephemeral State and Lyxdirs Single-Declarer invariants for the script path.
- send-keys payload stays a single line; `sendKeysLiteralArg` still wraps it.
- Semantic line breaks in every changed Go comment and markdown file.
- Documentation Lifecycle: `CONSTRAINTS.md`, `doc.go`, and `docs/sandbox-howto.md` updated in the same commit as the behaviour change.

## Auto-mode assumptions

- The POSIX pane shell is a Bourne-family shell (bash/sh), as the existing `export PATH=…${PATH:+…}` prelude already assumes.

## Open risks

None beyond today's behaviour.

## Acceptance criteria

- A strand launched on POSIX shows `. '<AnchorPath>/.lyx/reed/launch/<guid>.sh'` as its typed line, and the file holds `composePaneLaunchLine`'s output plus a newline.
- `internal/shell` tests: `Source` and the extension accessor for both dialects, including a path with a single quote and a Windows-style path; pwsh's `Source` is the script-block dot-source over `ReadFile`, never a dot-source of the file path.
- `reedengine` unit tests (fake tmux): send-keys payload is the source statement; the script content equals the composed line; `Cmd` and `ResumeCmd` launches both rewrite the file; the executable-error degrade writes a prelude-less script; an empty launch command writes the prelude alone.
- Write-failure test: an unwritable script directory yields a send-keys payload equal to the full composed line and a `logger.Warn` naming the strand.
- Cleanup tests: `RemoveStrand` (plain and `--recursive`) and `ReplaceStrand` delete the scripts of every removed GUID and leave survivors' scripts;
  `Down` removes the launch directory together with `reed.json`;
  a failed add leaves no script for its unpersisted GUID;
  every deletion tolerates an already-missing file.
- A live tmux integration test (POSIX) proves the sourced script behaves as the typed line did: the strand command runs, and the pane shell has `LYX_BIN` exported and the spawning binary's directory first on `PATH` after the command returns.
- No test or tool matches pane text against the launch line's content; any that did is retargeted to the script file or to the command's output.
- Existing reed integration tests (`contract_integration_test.go`, `emptycmd_integration_test.go`, `replace_integration_test.go`) and `panebin_enforcement_test.go` pass.
- `CONSTRAINTS.md`'s Pane Binary Resolution, `doc.go`, and `docs/sandbox-howto.md` describe the script route.

## Notes for the plan writer

- Existing `panebin_test.go` and spawn tests likely assert on the send-keys payload directly and need retargeting to the script content.
- The executable-path testability seam in `panebin.go` (`executablePath`) is the model for injecting a failing write.
- `lifecycle.go`'s `stateDir()` is the existing `.lyx` anchor reed uses; the launch directory hangs off it.
