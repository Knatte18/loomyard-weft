# Discussion round 2 — fixer report

## Fixed

- **F1** (MEDIUM, consistency): Decision 4 now states the prune exception at its rule: removing from a locked entry's `depends_on` a slug the same write removes as `done` passes, and every other `depends_on` change on a locked entry is still refused.
  Decision 2's prune bound points to it instead of restating the reason.
  Confirmed against `Store.Prune` in `internal/boardengine/store.go`, which rewrites every survivor's `DependsOn`.
- **F2** (MEDIUM, design): Decision 5 now says the check runs from the hub's prime worktree, names the outcomes that mean no run holds the entry (no batten seed, state `done`, or a halted state the operator decides to abandon; `running` means held), and says an agent in a task pair, where batten verbs refuse, takes the note and leaves the unlock to the prime or the operator.
  The Acceptance criterion on the refusal's message matches that wording, and Open risks records that a dead batten process still reads `running`.
  Confirmed against `internal/battencli/cli.go` (prime-only verbs, status reporting the persisted state and refusing an unseeded slug, no lock-holder report in `internal/shedverbs/status.go`).
- Support log's question ledger: Q14 revised, Q15 added for the prune exception.

## Disputed

None.

## Deferred

None.
