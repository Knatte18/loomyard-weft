```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. For each finding fixed as proposed, I built the case where the proposal is wrong or breaks something elsewhere in the artifact.

**Findings**

1. **MEDIUM.** D8 `front`/`on_land` bullet (F7): "`boardengine` refuses them on every payload write path … as `checkBeamPayload` refuses `beam`".
   - Summary: refusing the keys outright rejects a payload that carries the entry's own stored values unchanged. The model it cites, `checkBeamPayload`, refuses only a change.
   - Case: an operator or orch reads the fix task with `lym board get`, edits its body and sends the full entry back through `lym board upsert`, still carrying `front: true` and `on_land: adopt-check`. As written, the write is refused.
   - Evidence: `internal/boardengine/project.go:54-55`: "A value equal to the stored one, false for a new entry, is a no-op and is dropped from fields; any other value is refused."
   - The proposal cited the `beam` model but not this equal-value rule. The record should refuse only a value that differs from the stored one.

2. **MEDIUM.** D8 bullet "`fronttask` files no front task in a worktree where one stands open", together with D6's "While any front task stands open in the subject's worktree … the check files none, and the failed notice names that task instead" (F2 and F5).
   - Summary: the one-front-task-per-worktree bound lets an unrelated front task block the adopt fix loop, with nothing to resume it.
   - Case: a `parent-merge-task` front task (which D8 is built to serve) stands open in a beam's pair when its adopt check goes red. The check files no fix task. When the merge task lands, it runs its own `on_land` row, not `adopt-check`, so no check runs again. The held runs stay held.
   - The failed notice names only the open task. It does not name the operator's re-run of `lym adopt <subject>`, which is the only thing that starts a new check here.
   - Evidence: D6's notice wording, and D8's action table rows, one per key.
   - Fix direction: the notice names the re-run, or the bound counts only front tasks carrying the same `on_land`.

No case found against F1, F3, F4 (its hand-removal case is already stated as a bound), F5's hash rule, F6, F9 or F10.

Tool calls:
- Read `internal/boardengine/project.go:50-80`, to check whether `checkBeamPayload` refuses an equal stored value. My context showed only its doc line, not its rule.
