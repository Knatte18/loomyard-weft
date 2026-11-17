```yaml
lens: fix-affected-set
outcome: reported
```

Task: check whether the fix covers every site in the sets behind F1, F3 and F4 (reinterpretation descriptions, hook-edit and record descriptions).

- `config_test.go:379` and `config.go:227` ("report the dropped value") belong to `MigrateConfig`'s retired-key drop, not `DropEditLoopEdits`.
- No `doc.go` reference to the reinterpretation or the hash is left stale.

**LOW — `EditRecordsFileName` doc is stale**
- Changed text: `internal/shuttleengine/run.go:496`, "the file in a run directory that an engine's edit hook appends its file-edit records to".
- Premise: F4 says the seam contract omits the content hash. This is the same contract surface in a twin location.
- Evidence: the seam doc `engine.go:314-325` was updated, but this adjacent comment only says "file-edit records".
- Impact is low: the sentence is not false.

**LOW — `shuttleengine/doc.go:214` was left as it was**
- Changed text: `internal/shuttleengine/doc.go:214-215`.
- Evidence: it already says each record carries a content hash, so it is consistent.
- Not a finding.

**LOW — `claudeengine/settings.go:425` and `claudeengine/editrecord.go:2` call the record "slim" without the hash**
- Evidence: `editrecord.go:54-56` and `claudeengine/doc.go:308-310` do state the hash, so the siblings agree.
- Not a finding.

**Only finding: the `EditRecordsFileName` doc**
- Fix as F4 intended: add that each record carries a hash of the edited file's content.

**Not checked:**
- The `lym config` template comment: its only change is a `--set` hint in the `reinterpret.go` flagged line.
- `docs/overview.md`: it has no reinterpretation text beyond line 665/708. The grep over `docs/` for "drops a hub" and "DropEditLoopEdits" returned none, so no doc claims the old drop-everything behaviour.

Tool calls (3 greps, 1 multi-sed read):
1. Grep for drop/reinterpretation wording across `.go`, `.md` and `.yaml`: to find sites that still describe "drops any value".
2. Grep for edit-record, `RecordEdit` and hook wording: to find sites F3 and F4 did not cover.
3. Read `template.yaml` and `config.go`: to check their `edit_loop_edits` text against the change.
4. Read `editrecord.go`, `settings.go`, `claudeengine/doc.go`, `run.go` and the `configreg.go` `Reinterpretation` doc: my context lacked these sibling comments.
