```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set I found, covered or not.

**Per-change list**
1. **`ReadAgentActivity` reads waiterless running runs** (`internal/shuttleengine/activity.go`).
   - Callers:
     - `battencli/wire.go:605` (idle notice).
     - `loomcli/looparm.go:168` (`len(readings) > 0` as "an agent is live").
     - `loomcli/decision.go:855` (`liveRunStrands`).
   - Tests covered: `activity_test.go` and `battencli/wire_seams_test.go`, both updated.
   - Docs covered: `shuttleengine/doc.go` and the `ReadAgentActivity` doc comment.
   - Not covered, F1 and F2 below: the `liveRunStrands` doc comment, `looparm.go`, and `battenshed/doc.go`.
2. **Hook set in the settings** (`PostToolUse` always present, new `PreCompact` and `PostCompact`).
   - Golden: covered; all 64 lines changed (one per case) and the `PreCompact` count matches.
   - Tests: `settings_test.go` covered.
   - `claudeengine/doc.go`, the `settingsHooks` comment and the file header: covered.
   - `PostToolUse` mentions outside `claudeengine`: `docs/overview.md:477` and `creelcli/cli_test.go`. Both still accurate; creel's mail hook still rides `PostToolUse`.
   - Set covered, nothing left wrong.
3. **`SessionSignal.Instruction` and `typedInstruction`.**
   - Other `SessionSignal` producers and fakes: the field is optional, so nothing breaks.
   - Hook names and the `<task-notification>` marker stay in `claudeengine`, as `PATTERN-shuttle-provider-seam` requires.
   - `claudeengine/doc.go` and `signals_test.go` covered.
   - Set covered.
4. **`RunState.RoleKey` and `CarriedInstructions`.** Both are `omitempty`, so an older record still reads (`PATTERN-tolerant-records`). `rundir.go` doc comments are present. Covered.
5. **`sent-texts.jsonl` and `recordSentText` in `typeVerified`.**
   - `shuttleengine/doc.go` and the `typeVerified` doc cover it.
   - I found no other send entry point to check: `typeVerified` is the single spot, ahead of the paste branch.
   - Covered.
6. **Start carry-over, notifier and Warn log.**
   - `SetNotifier` doc in `doc.go:89` says "holds silently"; the carry-over line is a new use of the notifier.
   - Not wrong, but see F3.
7. **PATTERN and `pattern/`.**
   - The diff declares no PATTERN entry changes.
   - No `Pattern-Departure:` trailer exists, and none is needed.
   - The invariants I checked hold: no new wait or poll, and tests keep their tier.
   - Covered.

**Findings**

- **F1, LOW, consistency.** `internal/loomcli/decision.go:844`.
  - The doc comment on `liveRunStrands` says "live shuttle runs". After the change it returns every running-record run, waiterless ones included.
  - `notifyLiveRuns` (`decision.go:334`) then posts decision notices to the strand of a waiterless run, which no process waits on. The notice goes to a session that may still be working, which is arguably right, but the comment no longer states it.
  - Fix: reword the comment to "running shuttle runs, a run whose waiter is gone included", or filter on liveness if the notice must not reach them.
- **F2, LOW, consistency.** `internal/battenshed/doc.go:20-28` and `internal/loomcli/looparm.go:168-181`.
  - `battenshed/doc.go` still describes `AgentActivity` readings as "live run" and "any live run", and says "no live run found" gives the "no agent activity readable" fallback.
  - A waiterless running run is now read and has `LastActivity` only. That is the intended D3 change, but the battenshed doc was not updated: a waiterless run no longer gives the fallback, and its API-error and usage-limit parts read zero.
  - `looparm.go` reads `len(readings) > 0` as an agent being active, so a waiterless run now counts as active there too, with no comment on it.
  - Fix: reword the `battenshed/doc.go` lines to "running run" and state that a waiterless run carries activity only. Add a one-line note at `looparm.go:177`.
- **F3, NIT, consistency.** `internal/shuttleengine/doc.go:89`.
  - The `Runner.SetNotifier` paragraph lists what the notifier is used for. If it enumerates the dialog notice only, the carry-over notice is missing from it.
  - I did not re-read that paragraph beyond the grep line, so check it.
  - Fix: add the carry-over line to that paragraph if it enumerates uses; otherwise drop this finding.

**Tool calls**
1. Grepped for `liveRunStrands` callers, `PostToolUse` outside `claudeengine`, the "no agent activity readable" and waiter message strings, and `ReadAgentActivity` or waiter mentions in `PATTERN.md`, `pattern/` and `docs/`. My context had none of these hits.
2. Read `decision.go` 810-830, `battenshed/doc.go` 20-32, `looparm.go` 140-168 and the `docs/overview.md` and `creelcli/cli_test.go` `PostToolUse` hits. My context lacked the `liveRunStrands` call site, the `battenshed` doc text and the `looparm.go` context.
3. Read `decision.go` `liveStrands` usages (`notifyLiveRuns`). My context lacked what the strands list is used for.
