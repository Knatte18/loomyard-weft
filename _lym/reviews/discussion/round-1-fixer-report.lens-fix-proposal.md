```yaml
lens: fix-proposal
outcome: reported
```

Checked the proposals for F1, F2, F4, F5, F6, F7, F8, F9, F10 and F11. Proposals F1, F2, F5, F6, F7, F9 and F11 hold up. For F1, the default `design-` stays legal, and an empty `LYM_DESIGN_PREFIX` falls back to it through `:-`.

- **MEDIUM**: the F4 fix and its twin in D4.
  - Changed text: D10 "re-store the record with `lym loom decision store` from the entry's run", the Auto-mode line "until the new binary runs `lym loom decision store`", and D4 "which ... `lym loom decision store` re-stores".
  - Summary: the way forward cannot reach a renamed entry. The run's board slug is its worktree name, which a rename never changes. So `decision store` from that run targets the old slug, and the board no longer holds it.
  - Evidence: `internal/loomcli/wiring.go:222` passes `shedrun.ResolveRunID(location, runID)` as the slug, and `internal/shedrun/resolve.go:20-24` returns `l.WorktreeName`. D2 refuses "a slug the board holds no entry for", and its way forward is to restore the entry with `lym board upsert`, which would undo the rename. A flag without a file on the new slug has no working recovery in the artifact.
- **LOW**: the F8 fix in the D8 Bound.
  - Changed text: "so `decision add` copies its record, never approved, as the parent's goto decided".
  - Summary: this contradicts D8's own Rationale, which the fix left unchanged.
  - Evidence: the Rationale says "copying only past the approval keeps an unapproved record of a second run off an entry an earlier run already flagged". After a goto on a second run, the unapproved record now overwrites the earlier run's approved one, so the Rationale's claim no longer holds.
- **LOW**: the F10 fix in D8.
  - Changed text: "the failed-commit refusal names `lym loom decision store` as the step after `lym fabric commit` lands the record".
  - Summary: the refusal names that step unconditionally, but D9 refuses on a run not past its approval and on a done entry. On a run inside Discussion-Review, which D8 itself never copies, the named step leads to a refusal.
  - Evidence: D9 refuses "on a run that has not passed its approval as D8 judges it" and "On a done entry". The step should be named only when D8 would have copied.

Tool calls:
1. Read `template.yaml`'s `design_prefix` default and `SetRunStatus` callers: needed to test F1 against the shipped default and to find where the run's board slug comes from.
2. Read `ResolveRunID` and grepped `runlock.go` for rename: needed to confirm the run's board slug does not follow a rename.
