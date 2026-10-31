```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's proposals the fixer carried out as proposed (F1–F8).

**Findings**

1. **MEDIUM** — F7, `internal/boardengine/board_test.go:184,248-249`
   - Summary: the legacy fixture's `"type":"loom"` was rewritten to `"type":"tapestry"`, so no test covers what a real pre-upgrade file does now.
   - Evidence:
     - `internal/boardengine/legacy.go:118` sets `Recipe: r.Type` with no mapping. A real pre-upgrade `tasks.json` carrying `"type":"loom"` therefore loads as recipe `loom`. That is the retired name, which `shedrun.ValidateRecipe` now refuses (F2's new rows pin this).
     - The test now passes only because its input was changed to the new name. It no longer exercises the old-data path it exists for.
   - Suggested handling (the fixer's call): either map `loom` to `tapestry` in `legacy.go`, or keep a legacy-input row with `"type":"loom"` that asserts the chosen behaviour. The reviewer's "change the recipe value" does not fit pre-upgrade data, which genuinely carries the old value.

2. **LOW** — F7, `internal/boardengine/task_test.go:187`
   - Summary: the row is now `project tapestry`. It still passes, but it no longer shows that a project refuses the retired name `loom`.
   - Evidence: the review said the old tests "pass only because the board and run profile layers do not validate the vocabulary", and the row was renamed in place.
   - Impact: only the chosen value changed, so the refusal text is still pinned for `tapestry`. This is minor.

F1 and F8 hold up against the cases I built:
- F1: the `Loom-Preflight` successor is a row that exists in the tapestry recipe, and `peers.go:88` checks the tapestry map first. A darn status never carries `Loom-Preflight`, so there is no cross-recipe collision.
- F8: the restored `loom` in the three package docs agrees with line 4 of `internal/lenses/doc.go`.

**Calls made**
1. `grep` for `Type` handling in `internal/boardengine/*.go` (non-test), plus `legacyTasksJSON` uses. I needed to know whether legacy `type` values are mapped on load. My context showed only the test fixture.
