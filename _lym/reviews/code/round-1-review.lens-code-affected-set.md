```yaml
lens: code-affected-set
outcome: reported
```

My task was the code-affected-set lens: everything outside the diff's own lines that its changes force.

**What each change reaches, and whether the diff covers it**

1. **New hidden verb `hook-edit` (`shuttleengine.EditHookVerb`, `shuttlecli/hookedit.go`).**
   - Covered: the `cli.go` resolution skip (line 73), the `AddCommand` wiring, the file header in `cli.go`, and the `Command` doc comment.
   - Covered: `shuttleengines.EditRecorder` and its paragraph in `shuttleengines/doc.go:34`.
   - Covered: the `cmd/lym` clitree walk checks Use, Short and audience. `Short` is lower-case with no period, the audience is internal, and the gate passed.
   - Not covered: `docs/overview.md:703` documents the sibling hidden hook verb `hook-batching` and its PostToolUse hook, but has no line for `hook-edit` or the edit hook. This is finding F1.
2. **New signal kind `SessionSignalFileEdit` and the new `FilePath`/`AgentID` fields.** Every reader of `ParseSessionSignals` was checked; all covered.
   - `SessionFold.applySignal` gets its new case.
   - These ignore unknown kinds: `waitshadow.go:146`, `loadturn.go:214`, `send.go:456,994,1014`, `apierrorretry.go:62`, `gatefixer/strand.go:77` and `positionedTurnStarts` (`wait.go:1078`).
   - These read only Event-marked signals: `activity.go:248` and `session.go` `stampEventTimes`. A file edit is not Event, and Claude's `ParseEvents` skips PostToolUse.
   - `newestSessionID` (`sessionread.go:104`) and the liveness read (`wait.go:1609`) take the session id; an edit line carries the session's id.
   - The fake engine in `fakes_test.go` has its `EDIT:` case. No test enumerates every kind; `codexengine/hookrun_test.go:58` lists only codex's.
3. **Claude `ParseSessionSignals` now reads PostToolUse lines.** Covered. Only the edit hook writes PostToolUse payload lines; the activity-stamp hook only touches a file. The Claude engine `doc.go:304-310` paragraph is added.
4. **`buildSettings` gains a PostToolUse entry.** Covered: the `settings.go` doc comment and the `settingsHooks` comment are updated, and `TestBuildSettings_EditHook` pins the entry.
5. **New config keys `edit_loop_edits` and `edit_loop_window_s`.**
   - Covered: `Config` fields, `template.yaml`, the `ConfigSpec` categories, `LoadConfig` validation, and the `shuttleengine/doc.go` paragraph.
   - CategoryNone fits PATTERN-config-layers. The watch emits a notice; it switches off no guard, review or approval gate.
   - No reinterpretation is needed, since these are new keys and no meaning changes.
   - `docs/overview.md` never mentions them; that is part of F1.
6. **`RunState.EditLoopNotifiedOffset`.** Covered: it is `omitempty`, per PATTERN-tolerant-records. It is written only through `saveState`, under `recordMu`. Its only reader is the Wait cursor init (`wait.go:307`).
7. **New parent notice text (`editLoopNotice`).** Covered within the package doc (`doc.go:618-632`).
   - Not covered: `docs/overview.md:662-663` lists the notices an autonomous run sends its parent, held turn end and dialog only. The edit-loop notice is missing. Folded into F1.
8. **New interface `EditRecorder`.** Covered: the engine.go doc, the Claude assertion `var _`, and `shuttleengines/doc.go`.
   - The capability list at `shuttleengine/doc.go:37-40` already omits `BatchingNudger`. It was incomplete before this diff, so no finding.
9. **Repository rules.**
   - PATTERN-code-conventions: the engine name goes through the `engineName` constant inside its own engine directory, and the matcher is built from tool constants. Holds.
   - PATTERN-shuttle-provider-seam: `shuttlecli` reaches the engine only through `shuttleengines`, and `shuttleengine` stays neutral. Holds.
   - PATTERN-run-records: no file is written to the friction directory, so its writer rule holds. The summary states this departure from the board's "friction note" wording.
   - PATTERN-refusal-way-forward: it binds webster, shed and loom only; the new config refusals still name a way forward.
   - PATTERN-child-processes (`selfexec.LymBinary` for a hook): the hook command names bare `lym` from PATH. So do the existing batching hook (`batchingnudge.go:96`) and creel hook (`settings.go:105`). A bare PATH name never re-execs the running binary, which is the rule's target. I judge this the established hook convention, not a violation.
   - PATTERN-cli-cobra: hidden, internal audience, `ShouldAbort` checked first. Holds. The stderr plain text matches `hook-batching`.
10. **Commit messages and departures.** None of the three commits carries a `Pattern-Departure:` trailer. The first commit's claims (a slim hook, the session keeps running, one notice) match the diff. The directive declares no PATTERN changes.

**Findings**

- **F1** — severity LOW, class consistency, origin lens:code-affected-set. `docs/overview.md:662-663` and `docs/overview.md:703`.
  - The overview lists an autonomous run's parent notices (held turn end, dialog) and the PostToolUse hooks with their hidden verbs (`hook-batching`).
  - The diff adds a third notice, a new PostToolUse edit hook running `lym shuttle hook-edit`, and two `shuttle.yaml` keys. It leaves the overview without any of them.
  - Evidence: `docs/overview.md:662` reads "An autonomous run notifies its parent once per held turn end … and once per permission or input dialog". `docs/overview.md:703` documents `hook-batching` alone.
  - Fix: add the edit-loop notice to the notice sentence at line 662, and add a sentence on the edit hook and its keys beside line 703.

Tool calls:
1. Grep for other readers of shuttle.yaml keys, of `BatchingNudger` and of `DialogNotifiedOffset`. My context lacked any doc or test outside the diff that lists config keys, capabilities or notice offsets.
2. Grep of `shuttleengine/doc.go`, `codexengine.go`, `spec.go` and the `cmd/lym` tests for key and capability lists and hook-verb lists. I needed to know whether a capability list or a hidden-verb list must change.
3. Read `shuttleengine/doc.go:35-80`. My context held only the diff hunk, not the package doc's capability list and `shuttle.yaml` paragraph.
4. Grep for exhaustive lists of `SessionSignalKind`. I needed to find any test or table that enumerates every kind and would need the new one.
