```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F3, F10 and F11 change set for stale siblings and twins left unchanged.

**Findings**

1. **LOW**: `internal/driftengine/records.go` `CaptureRecord.Skip`, and `internal/driftengine/doc.go:95` ("It also holds the pid, HEAD, skipped tests, …").
   - The fix deleted `ScopeRecord.Skip` and the partial-scope skip list, but left its twin on the capture record.
   - Its comment says "a scope replacement reads it to tell whether the capture profiled each test". No remaining reader or writer of a capture-side skip list turned up.
   - The grep of `capture.go` and `records.go` for `Skip:`, `.Skip =` and `skip` found no assignment to it (only `{skip_file}` text).
   - It is the same old-Drift skip-list datum F3 removed, and `doc.go` still lists "skipped tests" as capture-record content.
   - Fix is to delete the field and the `doc.go` phrase, or name it under Kept on purpose in `_lym/landing/summary.md`.

2. **LOW**: `_lym/landing/summary.md`.
   - It was not changed and is outside the fixer's write surface.
   - It still lists the Drift removals only as config-side items.
   - It does not mention that the stored old-Drift scope readers (the `partial` scope, the compare, waiver-render and acceptance fields) are now deleted.
   - It does not record that a stored `partial` scope now reads as `full`, or that the waiver readers in `skipReason` and `verify.go:143` were kept.
   - A reader of the summary would not learn that these readers are gone or that the waiver ones remain.

No other finding. The F10 wording agrees across `config.go`, `doc.go` and the `config_test.go` row. The remaining `fabric reconcile` hits are research files and are out of scope.

**Further tool calls**
1. `grep` for `fabric reconcile` and stale drift phrases across `internal` and `docs`, to find twins and stale text the context lacked.
2. `grep` for `Skip:` and `skip` in `capture.go`, `records.go` and `capture_test.go`, to see whether `CaptureRecord.Skip` has any reader or writer left.
