```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for siblings or twins left out of step.

**LOW — `internal/fabricengine/doc.go:1295` is stale after F2.**
The doc says "`MoveInProgress` reads it for every weft writer, so a held lock names a change in progress and a free lock names the interrupted change". `MoveInProgress` now names a held lock as "in progress in process N" only when the record's pid and start time are a live process. A held lock beside a stale record is named as "starting in another process". The same doc.go section at line 1297 describes the Begin refusals in terms of "a held move lock", which F2 does not contradict. The stale text is the line 1295 sentence.

Checked, with no inconsistency found:
- **F2 tests:** the two tests that asserted the old wording are updated — the `LockHeld` row in `moverecord_test.go` and `MoveRecordOnForkPoint_LockHeld` in `addparent_integration_test.go`. The `anchorchange_integration_test.go` rows at lines 305–330 already expect the live and starting wording, so they agree. A search of `internal`, `docs`, `PATTERN.md` and `pattern` found no other assertion or doc on "in progress in process".
- **F1:** the other caller, `addanchor.go:88`, discards the first return value. It compiles with the new bool and already treated an after-commit error as an error. The only `moveWeftContent` doc mention (`doc.go:1307`) does not name its return type.
- **F3:** the `readRecordedAnchor` header was updated to drop "unreadable file" from the fallback list. No doc outside `anchor.go` mentions an unreadable marker.

Tool calls (3 further):
1. Grep of `internal`, `docs`, `PATTERN.md` and `pattern` for the old wording and held-lock phrasing. My context lacked a repo-wide view of twin assertions.
2. Grep of `doc.go`, the `lyxcwd` doc, `PATTERN.md` and `pattern` for lock and marker descriptions. My context lacked the doc text.
3. Read of `anchorchange_integration_test.go:305-340`. My context lacked the sibling held-lock rows.
