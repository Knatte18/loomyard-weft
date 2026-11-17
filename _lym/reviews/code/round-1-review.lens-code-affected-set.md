```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each thing the diff changes, I found what reads, calls, asserts or describes it and checked whether the diff left any of it wrong.

**Per change, the set found and whether it is covered**

1. **`SessionSignal.ContentHash` and the `content_hash` record key**, `internal/shuttleengine/sessionsignal.go:82`.
   - Writer `RecordEdit` and reader `fileEditSignal`: covered.
   - Fake parser in `fakes_test.go`: covered.
   - `claudeengine/doc.go`, `shuttleengine/doc.go`, `docs/overview.md:708`: covered.
   - The engine seam that describes what a recorder must write, the `EditRecorder` interface doc in `internal/shuttleengine/engine.go:314-323`: **not covered**. See finding 2.
2. **Edit-loop counting (`fold`) and the notice text**, `editloop.go`.
   - Callers `watchEditLoops` and `run.go:412`: covered.
   - `wait.go:305` and `rundir.go:144` describe only the offset and cursor, so they stay correct: covered.
   - `docs/overview.md:665`, `shuttleengine/doc.go`, `Config.EditLoopEdits` doc, `template.yaml`: covered.
   - Test `editloop_test.go`: covered.
   - The `hook-edit` verb's help text in `internal/shuttlecli/hookedit.go:26-31` still describes the old semantics: **not covered**. See finding 1.
3. **Template default changed from 4 to 3.** `config_test.go` `TemplateValues` is updated. Nothing else asserts 4: covered.
4. **New reinterpretation `shuttle-edit-loop-repeats`**, registered in `configreg.go:211`.
   - `configreg_test.go` row: covered.
   - The hubreconcile integration tests in `ensure_integration_test.go:246,1211,1586` walk every module's `Reinterpretations` generically, so the new marker is handled with no edit: covered.
   - The configreg and hubreconcile doc sections on reinterpretations are generic and enumerate no modules: covered.
   - Board marker and mail go through the existing `applyReinterpretations`: covered.
5. **`RecordEdit` hashes the file at hook time.**
   - Caller `hookedit.go` `hookEdit`: unchanged and correct.
   - `hookedit_test.go` asserts only that some text is held in the record, so the new key does not break it: covered.
6. **`TestModules_CreelRegistersTheTrustedHubsReinterpretation` renamed** to `TestModules_RegisterTheirReinterpretation`. The name appears elsewhere only in `docs/research` measurement files, which are point-in-time reports: no finding.
7. **Commit messages and departures.** Neither commit carries a `Pattern-Departure:` trailer, and this round declares no PATTERN changes, so there is nothing to check under either rule.
8. **Repository rules: `PATTERN-config-layers` "Config reinterpretation".** The change carries a `configreg.Reinterpretation`, `hubreconcile` applies it once behind its board marker, and the orch is mailed: covered. Whether dropping a value counts as "rewrites the value to its old meaning, or flags it" is outside this lens and left to the handler.

**Findings**

1. **LOW, consistency**, `internal/shuttlecli/hookedit.go:26-31`. The `hook-edit` help text still says the verb "appends one record of the edited file" and that the wait counts the records "to tell its parent about an agent that edits one file again and again". The diff changed both: the record now carries a content hash, and the watch counts only edits that return a file to earlier content. Evidence: `hookedit.go:26-31`, which the diff does not touch, against the new semantics in `editloop.go` `fold` and in `docs/overview.md:665`.

2. **LOW, consistency**, `internal/shuttleengine/engine.go:314-323`. The `EditRecorder` interface doc and the `RecordEdit` method doc still describe the record as only "each file edit". The watch now skips any edit signal with an empty `ContentHash`, at `editloop.go` `fold`, where the check is `signal.ContentHash == ""`. So an engine that implements `EditRecorder` from this contract and writes no hash gets an edit-loop watch that never fires. The seam contract should say the record carries the hash of the file's content after the edit. Evidence: `engine.go:314-323` against `sessionsignal.go:82-85` and `editloop.go:458`.

Tool calls:
- grep across `internal`, `cmd`, stencils and specs for edit-loop and reinterpretation references and for `EditRecorder`: the manifest lacked the full set of readers, tests and specs outside the diff.
- Read of `hookedit_test.go` assertions, the `EditRecorder` interface in `engine.go`, and a grep of the docs and specs: the manifest lacked the hook-edit test, the seam contract text and the bolt specs.
