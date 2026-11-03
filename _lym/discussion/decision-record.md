## Goal

```markdown
Imported from [issue #37](https://github.com/Knatte18/loomyard/issues/37).

What happened: the PR-Review row halted the loom run as blocked with 'pushing the task branch failed: fabricengine: push lock busy: <weft>/.weft/fabric.push.lock still held after 30s; neither side was pushed'. The way forward named was to run lym fabric push in the task worktree, then lym loom resume. The run had been through 17 history entries and had just finished Describe.

Why it is abnormal: the push lock sits in a weft directory shared across worktrees, so another session's push or a stale lock holder can stall a push; a 30s wait then ends the whole run as an escalation to a human, though contention is transient and a push is idempotent. Nothing in the note says which process held the lock, or whether it was stale.

What to do: retry the push with backoff for longer, or re-step the row automatically before blocking; when the lock is still busy, report the holder (pid, age) and clear a lock whose holder is dead.

Provenance: task pinned-solvers-everywhere; notes loom-halt-2.md
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-03): No clearing of a dead holder's push lock

Decision: Keep the omission: the run clears no push lock whose holder is dead. A stale holder record is reported as stale, as the diff does.

Rationale: The push lock is an OS lock that its holder's exit frees, so a dead process cannot hold it; only the holder record can go stale, and the next holder overwrites it. The Goal's clearing step has nothing to clear.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - dead-holder-lock-clear-unrecorded (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
