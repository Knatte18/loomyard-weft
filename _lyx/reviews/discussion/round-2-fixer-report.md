# Fixer report: discussion round 2

## Changes

- F1 (MEDIUM, design), fixed differently in one detail.
  Premise confirmed: `lyx shed seed` (`internal/shedcli/seed.go`) only calls `shedrun.WriteSeed` and commits nothing, and `seedAndCommitBootstrap` (`internal/loomcli/sharedbootstrap.go`) refuses at its seed stage before its step-3 commit.
  `shedrun.WriteSeed`'s `ErrDisagreeingSeed` message says "keep the existing seed and drive it", which loops on this path.
  Took the review's first option: D4 now replaces that way forward, for the shared bootstrap of `step` and `start`, with a plain `rm <seed path>` and a re-seed carrying the recorded parent.
  The detail that differs: the Fix says to delete `seed.json` "at the path the refusal names", but today's refusal names only the run-id (`internal/shedrun/seed.go:212`), so D4 says the path is rendered into the message, along with the existing seed's recipe and driver and the recorded parent the bootstrap has already read.
  D4 gains a Bound for the `rm` remedy: it names one file, `rm <path>` is not denied (`pattern/PATTERN-no-denied-recovery.md`), and the re-seed must still agree with the recorded parent.
  D3's Bound drops "no verb removes a seed" and states that a wrong value reaches no history.
  Scope, the `PATTERN-no-denied-recovery` constraint, Auto-mode assumptions, the acceptance criteria (the wrong-parent criterion and the refusal-spec rows) and the plan-writer notes follow.
  The stranded-pair entry is removed from Open risks.
  Support log: question ledger item 12 updated, and the open-risk alternative added under Rejected alternatives.
- `lyx loom validate-discussion` exits 0 after the edits.

## Disputed

None.

## Deferred

None.
