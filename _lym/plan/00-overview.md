---
format: 5
approved: false
language: go
---

# Plan: Decision record on the board entry

When a tapestry run's Discussion is approved, lym copies the run's decision record onto the task's board entry as `decision-<slug>.md` at the Bolt root and flags the entry `discussed`.
The record file follows its entry through rename, removal and replay.
`lym loom decision add` refreshes the stored copy, the new `lym loom decision store` re-copies it, and `lym board get` shows the flag and prints the record.
The board engine comes first (cards 1–5), then the recipe-row rule (card 6), the approve seam (card 7) and the two loom verbs (cards 8–9).

## Card Index

1 — board-decision-record — `Task.Discussed`, the record file constants, `StoreDecisionRecord`, `DecisionRecord` and the `discussed` payload rule
2 — record-follows-entry — the write scaffold moves the record file on rename and removes it with its entry
3 — design-prefix-refusal — board config refuses a `design_prefix` that renders onto a record file name
4 — replay-carries-record — the board replay claims, names and applies record files and merges `discussed`
5 — board-get-record — `lym board get` shows `discussed` and prints the record with `--record`
6 — discussion-approval-row — loomrecipe finds a recipe's discussion approval row and judges a run past it
7 — discussion-approve-seam — Discussion-Bouncer's `approve_seam: discussion` stores the approved record, guarded by a recipe test
8 — decision-add-copies — `decision add` copies the committed record onto the entry past the approval
9 — decision-store-verb — `lym loom decision store` re-copies the committed record by hand

## Shared Decisions

- One loomcli helper, `storeDecisionRecord`, is the only place loom opens the hub board to store a record; the approve seam, `decision add` and `decision store` all call it.
  The approve seam passes `shedrun.ResolveRunID(location, c.runID)`, the slug `SetRunStatus` is written for, as D6 names; the two verbs pass `seedSlug(target.WorktreeName)`, the slug their own pre-run resolves, which equals it for the run of the worktree, the only run either verb addresses.
- "Past the approval" is judged only by `loomrecipe.PastDiscussionApproval` over the recipe's own rows, never its name, so darn, sampler and pawl runs answer no approval row.
- A done entry's refusal is the sentinel `boardengine.ErrRecordEntryDone`, so loomcli tells it apart from other store failures with `errors.Is`.
