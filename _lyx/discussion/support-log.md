# Support log — reed-launch-script

## Interview

Autonomous session: no operator was present, so every turn below is a question the agent posed to itself after reading the code, with its self-picked answer.

1. **Is this the right thing to build?**
   The board body names the problem (long typed line is noisy) and a direction (generated script, short source line).
   Reading `panebin.go`, `spawn.go`, and `claudeengine/command.go` confirmed the line is composed from two independent layers and typed via one `send-keys -l` at `launchStrandLocked`.
   A launch script at that one chokepoint fixes every strand-realizing path at once, with no change to shuttle; building it as described.
2. **Who owns the script — reed or shuttle?**
   reed: only reed holds the joined prelude+command, reed replays `ResumeCmd` itself, and non-shuttle strands exist.
3. **Where does the file live?**
   Under reed's `.lyx` state dir, `.lyx/reed/launch/<guid>.sh|.ps1`, one file per strand.
   The board text said "the strand's scratch dir"; reed has no per-strand dir today, and one file does not justify creating one.
4. **How is it invoked?**
   Dot-source on the absolute quoted path in both dialects, so env exports land in the pane shell as before.
   Revised in review round 1: pwsh dot-sources a script block built from the file's text instead of the file itself, so no ExecutionPolicy can refuse the launch.
   Both the statement and the file extension go into `internal/shell` (Shell Mechanics Seam).
5. **What goes in the file?**
   The exact composed line plus a newline — no header, no reformatting — so "behaviour unchanged" holds by construction.
6. **Failure and edge cases.**
   Write failure falls back to the full typed line with a `logger.Warn`, mirroring the executable-error degrade.
   Empty command and unresolvable executable both still go through the script.
   Atomic temp+rename write; rewritten on every launch.
7. **Lifecycle.**
   Delete on `RemoveStrand` (subtree-wide, best-effort); keep across `Down` because `Resume` rewrites anyway.
   Revised in review round 1: `Down` deletes `reed.json`, so nothing survives it for `Resume`;
   the rule became "a script exists only while its GUID is in the persisted strand table", covering `ReplaceStrand`, `Down`, and a failed add as well.
8. **Testing.**
   `internal/shell` unit tests for both dialects; reedengine fake-tmux tests for payload, content, relaunch, degrade paths, and removal; one live POSIX tmux integration test proving env exports survive sourcing.

## Rejected alternatives

- **Shuttle writes the script into its run dir (next to `prompt.md`).**
  The prelude is added by reed afterwards, so the shuttle-side script could only hold half the line and the typed line would still carry `export PATH=…; export LYX_BIN=…`.
  It would also leave non-shuttle strands and reed's own `ResumeCmd` replay uncovered.
- **Execute the script (`sh launch.sh` / `pwsh -File`) instead of sourcing it.**
  A child shell drops the `PATH`/`LYX_BIN` exports when it exits, changing what the pane shell sees after the agent returns — a behaviour change.
- **pwsh dot-source of the `.ps1` file path (`. '<path>'`), relying on pwsh 7's default `RemoteSigned`.**
  A machine whose effective policy is `Restricted` or `AllSigned` (common under Group Policy, and the client default for a Windows PowerShell 5.1 pane shell set via `LYX_REED_SHELL`) refuses it, and the strand never starts, undetected by reed.
  That contradicts the never-fail-a-launch rule the write-failure degrade follows.
  The script-block form costs a few dozen typed characters against a full line carrying several paths and flags, so it keeps the point.
- **pwsh `Invoke-Expression (Get-Content -Raw …)`.**
  Equally policy-proof, but dot-sourcing a script block mirrors the POSIX `.` and states the current-scope intent directly.
- **Reformat the script one statement per line.**
  reed can only split its own prelude — the launch command is opaque and may already chain statements — so the result would be half-reformatted and no longer byte-equal to what was typed before.
- **Per-launch filenames (`<guid>-<n>.sh`) to avoid overwrite races.**
  No overwrite race exists: every relaunch path reaches `launchStrandLocked` only for a strand with no live pane (Resume and `--if-absent` select strands whose pane is not alive, and `UpdateStrand` launches a strand that has never held a pane), so no earlier pane can source a rewritten script.
  Unique names would also need their own cleanup; atomic rename is enough.
- **Per-strand directory `.lyx/reed/strands/<guid>/launch.sh`.**
  An extra directory level for a single file; the GUID-named file is equally identifiable.
- **Relative script path.**
  `PaneCwd` and `AnchorPath` are separate geometry fields and need not coincide.
- **Config key to opt back into the full typed line.**
  Nobody asked for it; the file keeps the line inspectable.
- **Hard-fail the launch when the script cannot be written.**
  A cosmetic feature must not stop agents from starting.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Right thing to build? — **resolved, auto-pick:** yes, at `launchStrandLocked`; the board direction matches the code's single chokepoint.
- Q2 Script owner? — **resolved, auto-pick:** reed.
- Q3 Script location? — **resolved, auto-pick:** `<AnchorPath>/.lyx/reed/launch/<guid><ext>`.
- Q4 Invocation? — **resolved, auto-pick:** dot-source the absolute quoted path; `Source` and extension accessor added to `internal/shell`.
- Q5 Script content? — **resolved, auto-pick:** composed line verbatim plus newline.
- Q6 Write failure? — **resolved, auto-pick:** warn and send the full line.
- Q7 Cleanup? — **resolved, review round 1:** a script exists only while its GUID is in the persisted strand table; removal, replace, `Down` and a failed add all delete.
- Q8 Windows ExecutionPolicy? — **resolved, review round 1:** pwsh dot-sources a script block built from the file's text, which no ExecutionPolicy governs.
- Q9 Opt-out config key? — **resolved, auto-pick:** none (YAGNI).
- Q10 Scrollback readers of the launch line? — **resolved, review round 1:** shuttle's production pane readers (`requireReadyAgentPane`, `sendVerified`) classify the provider TUI's ready marker and the sent text, never the typed launch line, so only tests could read it;
  an Acceptance criterion now requires any such test to be retargeted.
