# reed: send a short launch-script line instead of the full command

## Goal

Every strand pane reed realizes currently shows the full composed launch line as typed input: the pane-binary prelude (`export PATH=…; export LYX_BIN=…`) chained onto the caller's opaque launch command (for shuttle, the `CLAUDE_CODE_FORK_SUBAGENT=1 'claude' "$(cat …/prompt.md)" --session-id … --settings … --model … --effort … --dangerously-skip-permissions` line).
The operator finds this noisy.
reed instead writes the composed line to a per-strand launch script and types only a short source statement for it.
Pane behaviour is unchanged, and the full line stays inspectable on disk.

## Scope

In:

- `internal/reedengine`'s strand-launch chokepoint (`launchStrandLocked`, reaching `composePaneLaunchLine` in `panebin.go`): write the composed line to a launch script, send the source statement instead.
  Every strand-realizing path goes through this chokepoint — `AddStrand`, `UpdateStrand`'s relaunch, `Resume`'s replay of `ResumeCmd`, and the `--if-absent` relaunch — so all of them are covered with no per-caller change.
- `internal/shell`: two new dialect mechanics — the statement that sources a script file into the current shell, and the dialect's script file extension.
- Deleting a strand's launch script when the strand is removed (`RemoveStrand`, including a recursive subtree).
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
  one file per strand keyed by GUID makes each file identifiable, keeps concurrent strands apart, and needs no per-strand directory for a single file.
  The directory name `.lyx` is taken from `lyxdirs`, never spelled literally (Lyxdirs Single-Declarer Invariant).
- **The typed line is the dialect's source statement on the script's absolute, quoted path** — POSIX `. '<path>'`, pwsh `. '<path>'` (dot-source).
  Rationale: sourcing runs the script in the pane's own shell, so `export PATH`/`LYX_BIN` land in the pane session exactly as when typed, and the prelude still runs after the pane shell's profile.
  The absolute path is robust against `PaneCwd` and `AnchorPath` differing.
- **Both new mechanics live in `internal/shell`:** a `Source(path)` statement and a script-extension accessor, implemented for both dialects.
  Rationale: Shell Mechanics Seam — reed emits no raw dialect syntax, and the extension is dialect knowledge like the rest.
  The dialect stays `shell.ForGOOS()`, the same selector already shared by the prelude and the launch command.
- **Script content is the composed line exactly as today (`composePaneLaunchLine`'s output) plus a trailing newline.**
  Rationale: "behaviour unchanged" is then true by construction — the pane executes the same bytes it used to receive by send-keys.
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
- **Removing a strand deletes its launch script**, for every GUID in the removed subtree, best-effort: a missing file is silent, any other delete error is a `logger.Warn` and never fails the remove.
  `Down` keeps the scripts, since the strands stay persisted for `Resume`, which rewrites them anyway.
  Rationale: GUIDs are unique per strand, so without cleanup a long-lived worktree accumulates one file per strand ever launched; while the strand exists its script stays inspectable.
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

- The Windows pane shell (pwsh 7, default ExecutionPolicy `RemoteSigned`) runs a locally written `.ps1` by dot-source; no `-ExecutionPolicy Bypass` or `Invoke-Expression` workaround is built in.
- The POSIX pane shell is a Bourne-family shell (bash/sh), as the existing `export PATH=…${PATH:+…}` prelude already assumes.
- No operator-facing config key is added; the script route is unconditional.
- `.lyx/reed/launch/` is the path; no per-strand directory.

## Open risks

- A Windows machine whose effective ExecutionPolicy is `Restricted` or `AllSigned` would refuse the dot-source and the strand would never start, where the typed line used to work.
  The failure shows in the pane as a PowerShell policy error; it is not detected by reed.
- Anything that scrapes pane scrollback for the launch line (tests or tooling) would now see only the source statement; the Plan producer must check for such readers.
- A relaunch of the same strand overwrites its script while an earlier pane might not yet have read it; the earlier pane is killed by the relaunch's reconcile, so this is accepted rather than guarded with per-launch filenames.

## Acceptance criteria

- A strand launched on POSIX shows `. '<AnchorPath>/.lyx/reed/launch/<guid>.sh'` as its typed line, and the file holds `composePaneLaunchLine`'s output plus a newline.
- `internal/shell` tests: `Source` and the extension accessor for both dialects, including a path with a single quote and a Windows-style path.
- `reedengine` unit tests (fake tmux): send-keys payload is the source statement; the script content equals the composed line; `Cmd` and `ResumeCmd` launches both rewrite the file; the executable-error degrade writes a prelude-less script; an empty launch command writes the prelude alone.
- Write-failure test: an unwritable script directory yields a send-keys payload equal to the full composed line and a `logger.Warn` naming the strand.
- Remove test: `RemoveStrand` (plain and `--recursive`) deletes the scripts of every removed GUID, leaves survivors' scripts, and tolerates an already-missing file.
- A live tmux integration test (POSIX) proves the sourced script behaves as the typed line did: the strand command runs, and the pane shell has `LYX_BIN` exported and the spawning binary's directory first on `PATH` after the command returns.
- Existing reed integration tests (`contract_integration_test.go`, `emptycmd_integration_test.go`, `replace_integration_test.go`) and `panebin_enforcement_test.go` pass.
- `CONSTRAINTS.md`'s Pane Binary Resolution, `doc.go`, and `docs/sandbox-howto.md` describe the script route.

## Notes for the plan writer

- Existing `panebin_test.go` and spawn tests likely assert on the send-keys payload directly and need retargeting to the script content.
- The executable-path testability seam in `panebin.go` (`executablePath`) is the model for injecting a failing write.
- `lifecycle.go`'s `stateDir()` is the existing `.lyx` anchor reed uses; the launch directory hangs off it.
