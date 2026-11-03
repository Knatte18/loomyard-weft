```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the round-1 fix introduced can be carried out from the state it names and leads to progress.

Ways forward the fix introduced:
1. D1: a refused `design_prefix` → set `design_prefix` in `board.yaml` to another value.
2. D4: a crash between a rename's save and its move → `lym loom decision store` re-stores.
3. D10: `--record` on a flagged entry whose record file is missing → re-store with `lym loom decision store` from the entry's run.
4. Auto-mode assumptions: an older binary's rename leaves a flag without a file → the new binary runs `lym loom decision store`.
5. D8: the failed-commit refusal → `lym fabric commit`, then `lym loom decision store`.
6. D8 and D9: a done entry → no way forward, by design.

**[MEDIUM] After a rename, `lym loom decision store` cannot reach the renamed entry (items 2, 3 and 4)**
- Changed text: D4's crash sentence ("`lym loom decision store` re-stores"), D10's new refusal ("re-store the record with `lym loom decision store` from the entry's run"), and the new Auto-mode line ("until the new binary runs `lym loom decision store`").
- Problem: all three point at a step that stores onto the run's board slug, which is the old slug after a rename.
- Evidence:
  - D6 says the store targets "the run's board slug, the one `Board.SetRunStatus` is called with". That is `shedrun.ResolveRunID(location, runID)` (`internal/loomcli/wiring.go:222`), which returns `l.WorktreeName` (`internal/shedrun/resolve.go:20-25`), the run's original slug.
  - A rename cannot happen while a run holds the entry (`internal/boardengine/rename.go:44-45`). So any rename comes before or after the run, and the run's worktree still carries the old slug.
  - `decision store` from that run therefore reaches D2's absent-slug refusal, whose own way forward ("restore the entry with `lym board upsert`") is wrong for a renamed entry.
  - When the renamed entry is done, D9's new done-entry refusal blocks it too. When the run's worktree is gone, there is no run to call it from.
- As written, the flag-without-file state has no reachable way forward.

**[MEDIUM] D8's failed-commit refusal names `lym loom decision store` even for a run that has not passed its approval (item 5)**
- Changed text: D8, "the failed-commit refusal names `lym loom decision store` as the step after `lym fabric commit` lands the record".
- Problem: the sentence is unconditional, so the refusal names a step that cannot run on a run still inside Discussion-Review.
- Evidence:
  - `decision add` runs on a run inside Discussion-Review: D8's acceptance criterion covers that case and leaves the board untouched.
  - D9 is refused on "a run that has not passed its approval as D8 judges it".
  - Wording that works: name `lym loom decision store` only when the run has passed its approval.

Items 1 and 6 hold: the default template's `design_prefix` is `design-`, which passes the new D1 refusal.

Tool calls:
- `internal/boardengine/template.yaml` head, `SetRunStatus` callers and `rename.go` callers: my context lacked the default prefix and how a run's board slug is bound.
- The run-hold refusal in `rename.go` and `ResolveRunID` in `internal/shedrun/resolve.go`: my context lacked whether a rename can happen while a run holds the entry, and what slug a run stores onto.
