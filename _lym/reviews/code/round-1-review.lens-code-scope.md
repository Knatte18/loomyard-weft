```yaml
lens: code-scope
outcome: reported
```

Lens: code-scope. I traced each change in the diff to the board entry's goal points, and each goal point to the changes that build it.

**Decisions** (the decision record alone is the answer key; its `## Goal` holds the board entry, and the record has no `Added after Discussion` entries):
- **G1:** Shuttle counts edits per file per agent session from its `PostToolUse` events.
- **G2:** Past a set count in a set window, it raises a friction note to the run's parent, without killing the session.

**Changes → decision**
- **C1, edit recording hook.** `claudeengine/editrecord.go`, the `settings.go` PostToolUse entry and its doc, `engine.go` (`EditHookVerb`, `EditRecorder`), `shuttlecli/hookedit.go` and the `cli.go` wiring, `shuttleengines.EditRecorder`, and the `claudeengine/doc.go` and `shuttleengines/doc.go` paragraphs.
  - Traces to G1. The entry assumes `events.jsonl` already carries PostToolUse, but it does not, so a recording is needed to count "from its PostToolUse events". The summary says so.
- **C2, the file-edit signal.** `SessionSignalFileEdit`, the `FilePath` and `AgentID` fields in `sessionsignal.go`, and `fileEditSignal` in `signals.go`. Traces to G1.
- **C3, the counting watch.** `editloop.go` `fold`, the `editLoops` field in `run.go`, and the cursor init and tick call in `wait.go`. Traces to G1 and G2.
- **C4, the parent notice and its offset.** `editLoopNotice`, `watchEditLoops`, `persistEditLoopNotifiedOffset` and `RunState.EditLoopNotifiedOffset` in `rundir.go`.
  - The notice traces to G2. Reusing the notifier keeps the session alive, as G2 asks.
  - The persisted offset is forced by C4, so that a reattach does not repeat the notice.
- **C5, config.** `edit_loop_edits` and `edit_loop_window_s` in `config.go`, `template.yaml` and `ConfigSpec`, plus the load-time validation. Traces to G2's "set count in a set window".
- **C6, the session fold skip** in `sessionstate.go`. Forced by C2: a new signal kind would otherwise become the fold's newest signal and hide the contradiction an idle notice signals.
- **C7, tests.** `hookedit_test.go`, `editrecord_test.go`, the `config_test.go` row, `editloop_test.go`, the `EDIT:` case in `fakes_test.go` and the `sessionstate_test.go` row. Forced by C1 through C6.
- **C8, the `shuttleengine/doc.go` notice paragraph.** Forced by C3 and C4.

**Decisions → changes**
- **G1:** built by C1, C2 and C3, for the claude engine only (see Finding 2).
- **G2:** built by C3, C4 and C5, but as a notifier line rather than a friction note (see Finding 1).

**No unrequested scope.** Every change traces to G1 or G2 or is forced by a traced change. Two narrowings follow from the notice channel, not from extra scope:
- The interactive-run exclusion. The existing dialog notice skips interactive runs the same way.
- Counting subagents and forks apart through `AgentID`. G1's "per agent session" covers it.

**Findings**

1. **MEDIUM · decision.** `internal/shuttleengine/editloop.go:90-120` (`watchEditLoops`), and `_lym/landing/summary.md` "Design decisions", third bullet.
   - G2 asks for "a friction note to the run's parent". The diff sends a `Shuttle notice:` line through the runner's notifier and writes no friction note.
   - In this repository "friction note" is a defined record type. `pattern/PATTERN-run-records.md` says its one writer is `internal/friction`.
   - The writer reinterpreted the term on their own judgment. The summary states the reinterpretation, but the decision record has no `Added after Discussion` entry approving it.
   - Fix: leave the code in place and defer to the run's parent. The parent either records the reinterpretation with `lym loom decision add`, or asks for an `internal/friction` note beside the notice.
2. **LOW · scope.** `internal/shuttleengine/engine.go:314-324` (`EditRecorder`): only the claude engine implements it (`claudeengine/editrecord.go:17`).
   - G1 is engine-neutral: "Shuttle counts edits per file per agent session".
   - A Codex run records no edits, so it never gets an edit-loop notice. The summary's last design bullet says the codex engine "has no edit hook yet".
   - Codex already installs PostToolUse hooks (`codexengine/command.go:251`, and the activity and creel hooks per `codexengine/doc.go:62`), so the same seam was there to use. No record entry licenses the narrowing.
   - Fix: add a codex `RecordEdit` and its hook, or have the parent record the claude-only narrowing with `lym loom decision add`.

Tool calls:
- `grep PostToolUse internal/shuttleengine/codexengine/*.go`, to see whether Codex already has the PostToolUse seam an edit hook would need. My context held only the header of the codex `ParseSessionSignals`.
