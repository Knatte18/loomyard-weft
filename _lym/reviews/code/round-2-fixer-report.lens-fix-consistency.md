```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens, meaning whether the change set still agrees with its surroundings and whether any twin was left behind.

No findings.

- **Zero meaning (F2):** `internal/loomengine/template.yaml:29` already says "0 takes that whole capped room" for `assist_reviewer_timeout_min`. The new per-knob `zeroMeans` text in `internal/loomengine/config.go` agrees with it, and the config test fragment "must not be negative" is kept.
- **Await cap docs (F1):** `CapAssistTimeout` and its Warn text in `internal/loomengine/review.go` speak of "one await cap" generically, so a per-segment cap does not contradict them.
- **Role formation:** the only callers of the old two-value `loomSegmentSettings` signature were in `wiring.go`, and both are updated. The only other consumer of `ReviewRolePrefix` for role formation is `reviewTable`, now `ReviewChairRole`. The remaining uses are tests that build `ReviewRolePrefix`-based roles directly, which still match an empty prefix.
- **Overview doc:** `docs/overview.md` lists `assist_reviewer_timeout_min` as "capped to end before the chair's own timeout". That is still true, so no edit is needed.

Tool calls (1 of 6 used):
1. A `grep` for `assist_reviewer_timeout_min`, "await cap" and `ReviewRolePrefix` across the Go, YAML and Markdown sources, to find twins and stale cross-references. My context held only the files I had read, not the template, the docs or other callers.
