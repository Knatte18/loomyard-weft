---
verdict: APPROVED
---

PR 602 at `b8e5d3242` against `main` (`1c812c1f8`, which the branch has already merged) delivers the board entry's seven bundled defects and keeps to decisions D1 to D7.

## Against the goal and decisions

- D1: `checkLivenessTick` gained `eventsProof`, set only from `awaitStartup`, read only when `RecordsSessionStarts` is true, through `SessionSignalParser` with no hook names; it is checked before the capture, which still runs and dismisses a gate on a false answer, and `markStarted` persists `Started` for both paths.
- D2: `claudeengine.Prepare` puts the skill-load message on the launch line, bounded by `maxLaunchPromptBytes`; `loadSkillsThenPrompt` awaits the first load turn untyped, then types the color, then the retry and the prompt pointer; a spec without skills is unchanged. `PATTERN-role-skills-typed`, `docs/overview.md` and `roleskills_test.go` are reworded as D2 requires.
- D3: `statuspane` redraws with cursor home, erase screen and erase history; `NotifyResize` (SIGWINCH, unix only) ends the wait; the back-off stays; both the checklist and batten's overview get it.
- D4: only `PushWithRetry`'s doc comment changes; no behavior or log level.
- D5: `tmuxkit.BindStaleSocket` replaces the private helper and the closed-listener socket in `tmuxkit_test.go`.
- D6: `lyx loom fmt` checks and formats every file before writing, refuses the whole call with a `refused` list, resolves symlinks for root and path, writes only changed files (an existing file keeps its mode), and reports `written`/`failed` on a partial write; the edit directive gains its one line and the refusal spec its three rows.
- D7: `removeWarpWorktreeDir` finishes an unregistered, still-present target through `removeWorktreeLeftover` in `destroy.go`, retrying only late-writer errors (`ENOTEMPTY`; `ERROR_DIR_NOT_EMPTY` and `ERROR_SHARING_VIOLATION` on Windows) up to three attempts with an injected sleep, and records `KindPathRemoved`; `PATTERN-fabric-destruction-chokepoint` names the exception. The Discussion-Review carry-over on the Windows sharing violation is covered by `latewriter_windows.go`.
- Merge-Review carry-over `probequiet-ignores-go-driver-lock`: fixed in `b8e5d3242`; `probeQuiet` now reads `shedrun.DriverLock`, so a live Go driver refuses teardown, which keeps D7's premise that only short-lived writers remain. This is a merge-time fix that protects a recorded decision, not new scope.
- Plan-Review carry-over `fmt-unreadable-file-refusal-unreached`: an unreadable file refuses through the `os.ReadFile` branch of `planGoFormat` and is named in the refusal spec.

Nothing in the diff falls outside the goal; the Out list (reed layout, `clampToFit`, wait and attach probes, weft push behavior, killing reed processes) is untouched.

## Overlap

- Parent: `lyx loom peers` lists no landed parent commits beyond `main`, which the branch has already merged (`go-driver-assistant`, `1c812c1f8`); the merge's one interaction was the `probeQuiet` finding above, now fixed.
- `logging-cleanup` (Plan-Write) has changed no path yet (`changed_paths` empty, no commits on its branch). Its plan will later touch packages this run changes: `shuttleengine` (card 10 guards the `capture pane during startup probe` Warn in `checkLivenessTick`, card 9 the draft kind in `readiness.go` and `send.go`), `claudeengine` (card 5), `pairteardown` (card 14 adds the log archive) and `battencli` (card 6, a different file). Its decision record already names `run-window-bugs` as an open risk and plans to take this run's changes at Merge-In. Its decisions are compatible with this run's: it keeps `gitrepo: push refused, retrying` at Warn as D4 does, and its D13 discards logger stderr in the logger, not in `tmuxkit`. Neither run has a change that contradicts or duplicates the other's, and no shared code is changed today, so this does not escalate; this run, which is further along, lands first.
- Bookkeeping only: `cmd/lyx/testdata/test-budget.yaml` (`fabricengine` 646, `loomcli` 236, `pairteardown` 7), the module-table line in `docs/overview.md` (`fmt` added to loom's verb list) and a reworded PATTERN index line (`PATTERN-role-skills-typed`, owned by this run's D2).

No board entry is made obsolete: the six folded entries already live as sections of this entry, and `arbiter-fixes` depends on this run but is not replaced by it.
