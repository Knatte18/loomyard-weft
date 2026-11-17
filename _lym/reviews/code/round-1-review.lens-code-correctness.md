```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function the diff adds or changes against a concrete case, including an empty value or a failed call where it had one.

**Function / branch — case run — verdict**
- `machinecli.runStart`, no shortname and `liveHubEntries` fails — refuses naming the record. Correct.
- `machinecli.runStart`, shortname `zz` and `lookupHub` fails — the refusal is hubregistry's text, which opens `creel:` and points at creel mail. Misreported (F2).
- `machinecli.runStart`, `lymBinary` fails — refuses with its way forward. Correct.
- `machinecli.runStart`, `terminal()` or `desktop()` fails — a warning is recorded and the session field stays nil. Correct.
- `startSession.startHub`, pull fails or skips — warning, then the local owners are read. Correct.
- `startSession.startHub`, `readOwners`, `identity`, `hubStart` or `planStart` fails — `hub.Error` is set, the function returns, and the hub fails the verb. Correct.
- `planStart`:
  - prime elsewhere — skipped with the owner named. Correct.
  - prime with no owner — the prime is skipped. Correct.
  - prime owned here — `primeOwned` is true. Correct.
  - beam elsewhere, or beam with no owner — skipped. Correct.
  - beam owned here under an unowned prime — planned as `lym beam start <slug>`. The child then claims the prime (F1).
- `runBeamStarts`:
  - prime owned — a single `--all` child. Correct.
  - child errors — the label is marked failed. Correct.
  - entry carries an error — that worktree is marked failed. Correct.
  - unowned prime with an owned beam — the `beam start pay` child claims the prime. Wrong (F1).
- `openWindows`:
  - nothing started — opens nothing. Correct.
  - workspace open fails — warning. Correct.
  - `NamedWorktree` fails — warning, and no attach command is printed for that worktree. Acceptable.
  - terminal is nil, or opening it fails — the attach command is set. Correct.
  - prime title — `ConsoleWindowTitle` is passed `prime.WorktreeName` as the slug, so it renders the prime title. Correct.
- `keepLauncher`:
  - desktop is nil — does nothing. Correct.
  - something is owned and `Add` fails — warning. Correct.
  - nothing is owned — `Remove`. Correct.
- `holdOf`:
  - no holds — returns nil. Correct.
  - this machine's hold — the re-run is set. Correct.
  - another machine's hold — no re-run. Correct.
  - only `Holds[0]` is printed, the same as beamcli's status. Acceptable.
- `removeGoneLaunchers`:
  - `Shortnames` fails — returns the error, and the caller warns. Correct.
  - a launcher's hub is gone — it is removed. Correct.
  - a `Remove` fails mid-list — returns the partial list and the error. Correct.
- `runBeamStart`:
  - non-zero exit whose envelope has a result — entries are returned. Correct.
  - envelope error and no result — the error is used. Correct.
  - no JSON — falls back to stderr, then `runErr`, then the decode error. Correct.
  - the dobby and the orch watcher redirect stdio to files, so a grandchild cannot hold the pipe open and `cmd.Run` returns. Correct.
- `lastJSONLine`:
  - empty output — returns nil, and decoding nil errors. Correct.
  - the envelope is single-line JSON (`output.Ok` uses `json.Marshal`). Correct.
- `Desktop.Add`:
  - freedesktop path with `%`, `$` and `\` — escaping is correct to the Exec spec, and the test row confirms it. Correct.
  - WSL path holding `"` — refused. Correct.
  - WSL path holding `$` or a backquote — passes the check but breaks the outer double quotes (F3).
  - native Windows — quoted, `%` doubled. Correct.
  - second write fails — the first file stays and the error is returned. Acceptable.
- `Desktop.Remove`:
  - launcher already gone — no error. Correct.
  - last hub — the all-hubs launcher is removed. Correct.
  - another hub's launcher stands — the all-hubs launcher is kept. Correct.
- `Desktop.Shortnames`:
  - unrelated `lym-hub-notes.txt` — ignored. Correct.
  - a directory — skipped. Correct.
  - `lym-hub-.desktop` (empty shortname) — skipped. Correct.
- `locate`:
  - windows, WSL by env, WSL by interop, darwin, override set, missing directory, resolve error — each matches its test row. Correct.
- `parseUserDirs`:
  - `$HOME` alone, an absolute path, a relative path, no entry — correct.
- `freedesktopDesktopDir`:
  - config file absent — `$HOME/Desktop`. Correct.
  - config file unreadable for another reason — error. Correct.
- `query` and `windowsDesktopDir`:
  - empty output — error. Correct.
  - powershell hangs — bounded at 30s. Correct.
- `dobbyengine.addLauncher`:
  - Owner nil — no add. Correct.
  - Owner errors — warning. Correct.
  - owner `""` or elsewhere — no add. Correct.
  - no desktop — no add. Correct.
  - `Add` fails — warning. Correct.
- `removeLauncherIfOwnsNothing`:
  - empty shortname — does nothing. Correct.
  - `ReadOwners` or `ReadIdentity` fails — warning, launcher kept. Correct.
  - something is owned — kept. Correct.
  - nothing is owned — removed. Correct.
  - the call sits after the early returns for an unreachable origin or a refused registration, so it runs only on a pulled board. Correct.
- `OwnsAnything`:
  - empty prime and no entries — false. Correct.
  - prime recorded under a former name — true. Correct.
  - done beam — false. Correct.
  - consistent with the add rule: an empty effective owner adds nothing. Correct.
- `battencli.HubCatchUp`:
  - no shortname recorded — the zero Launcher keeps no launcher. Correct.
  - `BoltDir` for the shortname and `BoardDir` for the owners match the readers hubregistry uses. Correct.
- `termwindow.resolve`:
  - windows with `wt.exe` on PATH — returns the native arguments. Correct.
  - windows without `wt.exe` — `ErrNoLauncher`. Correct.
  - the Linux and WSL branches are unchanged. Correct.
- `nativeWindowsTerminalArguments`:
  - `;` in the path, title or dir — escaped. Correct.
- `konsoleArguments` and `windowsTerminalArguments`:
  - still build the same attach line from `lymPath` and `dir`. Correct.

**Findings**

F1 — BLOCKING, design. `internal/machinecli/start.go:600-608` and `:618-641` (`planStart` and `runBeamStarts`).
- Summary: when the prime has no recorded owner, the verb starts each owned beam with a `lym beam start <slug>` child. That child's prepare step claims the prime for this machine, so the verb claims a prime. D10 and its acceptance criterion forbid this ("claims no prime", "never claims a prime").
- Evidence, in the child's path:
  - `internal/orchcli/start.go:495` runs `report.Warnings = append(report.Warnings, c.claimPrimeNotes()...)` inside `prepare`.
  - `internal/orchcli/ownerpause.go:104-120`: `claimPrimeFor` writes `Prime: true` when `owners.Prime == ""`.
  - `internal/orchcli/doc.go:233-234`: "The claim runs in the prime and in a beam alike".
  - `internal/beamcli/start.go:24`: the first target is always prepared.
  - `internal/beamcli/cli.go:528-529`: `BeamUp(..., BeamUpOptions{Prepare: prepare, ...})`.
- Why tests miss it: the test fake `beamStart` in `internal/machinecli/start_test.go` never runs the real child. The row `PrimeWithNoOwnerIsNeitherClaimedNorStarted` asserts the opposite of what production does.

F2 — LOW, design. `internal/machinecli/start.go:489-491` (`runStart`, the shortname path).
- Summary: a shortname that does not resolve is reported with hubregistry's creel text, opening `lym machine start: creel: no hub with the shortname ...`. Its way forward is "run lym creel inbox in that hub once". For several matching hubs the text says "mail across hubs needs distinct shortnames". Both misreport the verb's context.
- Evidence: `internal/hubregistry/hubregistry.go:87-91`.

F3 — LOW, design. `internal/desktoplauncher/desktoplauncher.go:148-155` (`content`, the `FormWindowsWSL` case).
- Summary: only `"` is refused, but the POSIX-quoted run sits inside the double quotes of `bash -lc "..."`. wsl.exe's default shell parses those quotes, so a lym path holding `$`, a backquote or `\` is expanded there before bash sees the single quotes. The launcher would then run a different path.
- Evidence: line 155, `run+` -- bash -lc "`+posixRun+`"``.

Tool calls:
1. `sed` of `internal/boardengine/handles.go:190-215` and `internal/orchcli/start.go:470-500`. My context did not show how `ConsoleWindowTitle` treats the prime slug, or confirm that `claimPrimeNotes` runs inside the prepare step of every bring-up.
