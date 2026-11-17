```yaml
lens: code-affected-set
outcome: reported
```

Task: review with the code-affected-set lens. For each change, find what it forces outside the lines the diff touches and check whether that is left correct.

**Per change, the affected set and whether it is covered**

1. **`fileContentHash` (claudeengine) moved to `shuttleengine.FileContentHash`.**
   - Callers: `claudeengine/editrecord.go:64` and `codexengine/editrecord.go` (covered).
   - No test or other file refers to the old name (covered).
   - `editloop.go` header updated (covered).
   - `shuttleengine/doc.go:225-235` describes the hash in general terms and is not left wrong (covered).
2. **Codex `EditRecorder` (`RecordEdit`) reached through `lym shuttle hook-edit codex`.**
   - `shuttleengines.EditRecorder` picks up the capability by type assertion with no change needed (covered).
   - `shuttlecli/hookedit.go` covered.
   - `shuttlecli/cli.go:79` covered.
   - `engine.go:314-326` interface doc covered, with one NIT below.
   - `hookedit.go` Long text: NIT below.
3. **Edit hook entry in Codex `hookSettings` and `Prepare`.**
   - `codexengine/doc.go` hook paragraphs updated (covered).
   - `codexengine.go` Prepare doc updated (covered).
   - `TestHookSettings` and `TestPrepare_SwitchedToolHooks` updated (covered).
   - `docs/overview.md:711` and the Codex paragraph at `docs/overview.md:718`: **not covered** (finding 1).
   - `template.yaml:15` `edit_loop_edits` comment is engine-neutral (covered).
4. **Launch manifest / engine parity.**
   - `claudeengine/manifest.go` has no edit-hook item, so the Codex manifest needs none to stay at parity (covered).
5. **Codex `ParseSessionSignals` now reads `PostToolUse` lines as file edits.**
   - `events.go` doc updated (covered).
   - The events file gets no `PostToolUse` payload, because the activity hook only touches a stamp (`command.go:185-188`), so the events-file readers are unaffected (covered).
   - `events.go:3` header ("All Codex payload-shape knowledge the events file needs") is about the events file and is not left wrong (covered).
6. **`content_hash` key and the slim-line shape.**
   - Codex declares its own `contentHashKey` and `editRecord`, copying the claude engine's `claudeengine/editrecord.go:32,39-48`: finding 2.
7. **Rubric checks.**
   - Declared PATTERN changes: none declared (nothing to land).
   - Commits `f48ef5ba` and `e89f2574` carry no `Pattern-Departure:` trailer (nothing to judge).
   - No stencil is changed.

**Findings**

1. **MEDIUM · consistency · `docs/overview.md:711` (and the Codex paragraph at `:718`)**
   The overview still says only an autonomous claude run's `Edit`/`Write` calls run `lym shuttle hook-edit`.
   Its Codex paragraph lists the Codex `PostToolUse` hooks (batching, creel) but not the new `apply_patch` edit hook.
   The diff leaves the user-facing doc stating that Codex runs record no edits.
   Evidence: `docs/overview.md:711` ("an autonomous claude run's `Edit` and `Write` calls run the hidden `lym shuttle hook-edit`") and `docs/overview.md:718` (lists only `hook-batching codex` and creel hooks).
2. **LOW · consistency · `internal/shuttleengine/codexengine/editrecord.go:231-259`**
   The slim edit-record shape and its `content_hash` key now live twice: `contentHashKey` and `editRecord` here, and the same in `claudeengine/editrecord.go:30-48`.
   `PATTERN-code-conventions` says "a name or path that can go stale is one named constant, declared once and referenced everywhere".
   The line is lym's own shared record contract, which the summary calls "the same slim line", not a provider payload.
   It belongs in one place in `shuttleengine`.
   Evidence: `pattern/PATTERN-code-conventions.md` (Named constants); `claudeengine/editrecord.go:32` and `codexengine/editrecord.go:233`.
3. **NIT · consistency · `internal/shuttlecli/hookedit.go:27-28` and `internal/shuttleengine/engine.go:322`**
   Both docs still describe one record per hook input ("appends one record of the edited file", "appends the file edit the hook input payload describes").
   A Codex payload now appends one record per file the patch touches.
   Evidence: `hookedit.go:27`, `engine.go:322`, `codexengine/editrecord.go:273-282`.

Tool calls:
- grep for `FileContentHash`/`fileContentHash` callers and repo-wide `EditRecorder`/edit-loop doc mentions: the manifest did not cover callers and docs outside the read files.
- Read `shuttleengine/doc.go:215-235`, grep `docs/` for edit-loop text, grep the claude manifest for an edit item, grep the payload-shape header: to check the package doc, the overview, the parity item and the events-file claim.
- Read `docs/overview.md:664-672` and `:704-716`: the overview's edit-loop and Codex paragraphs were not in context.
- Read the hook mentions in `docs/overview.md:717-730`: to check whether the Codex paragraph lists Codex hooks.
