```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Both finding sets are covered, and I found no member the fix missed.

- **F1 (sites that form or look up the chair's role):**
  - Outside tests, the only place that resolves a keep-warm cap from a chair role is now `segmentAssistBounds` at `internal/loomcli/wiring.go:1131-1132`. It runs once per entry of `loomSegmentPerches`, so all three segments are covered.
  - The only other user of `ReviewRolePrefix` is `reviewTable`, which now goes through `ReviewChairRole`.
  - `burlerReviewEngineLabel` in `internal/shedadapters/burlerreview.go:26` is a label, not a strand role, so it is out of scope.
  - The earlier grep for `CapAssistTimeout` found one production caller, which is the one that was changed.
- **F2 (sites that state the zero meaning for `assist_reviewer_timeout_min`):** a grep over `internal`, `docs` and `contracts` finds no other non-test place that says 0 defers to shuttle for the assist key. The loop in `internal/loomengine/config.go` was the only site.

Nothing left unchecked.

Tool calls:
1. A grep for `ReviewRolePrefix`, `ReviewChairRole`, `"burler-review"` and `shuttleengines.KeepWarm(` in non-test Go code. My context had no search of `KeepWarm(` callers or other strand-role forms.
2. A grep for zero-meaning wording on `assist_reviewer_timeout_min` in docs and contracts. My context had only the `config.go` hit, not the docs or templates.
