```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the F3 and F10 searches and the covered sets hold. The F11 search was not rerun.

No findings.

- **F3 (stored old-Drift readers):** A repo-wide grep of `driftengine` finds no remaining reader of the old scope fields. The grep covered `ScopePartial`, `leaveOut`, `Accepted`, `ComparedAt`, `HaltNamed`, "old Drift"/"earlier Drift" and "skip list". The only `partial` hits are the `config.go:183`, `config.go:273` and `doc.go:25` refusal texts, which are correct. The `CaptureRecord.Skip` field and its `doc.go:95` mention ("skipped tests") remain. It is a different record and outside the review's named set, so it is not a fix gap. Nothing in the repo still writes or reads it for a partial scope, so it is dead and a possible follow-up.
- **F10 (default-layer way forward):** Non-test Go sources that offer `lym fabric reconcile` as a way to prune a key are none. The remaining hits in `fabriccli`/`beamcli` are about wiring, records or pushes, not retired keys. The research-doc hits are archived benchmark data, so I treated them as out of scope.
- **F11 (configsync seed row):** Not rerun. The fix adds one row to the one named test, and the search premise is a single mutation site.

Further tool calls, 2 of 6:
1. A `grep` for the leftover old-Drift symbols and the `fabric reconcile` retired-key text. My context held only the spans the fixer edited, not the whole package.
2. A `grep` of non-test Go for `lym fabric reconcile` offers, and for `CaptureRecord.Skip` users. My context did not show the other callers.
