# Round 1 fixer report — discussion

All seven findings were validated against the code and fixed.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1 (BLOCKING)** — confirmed: `setModule` (`internal/configcli/configcli.go`) routes any `HubWide` write to `setHubWide`, which commits the board file from any worktree.
  D10 now has `lyx config` refuse a write to any `HubWide` module when the resolved worktree is a task pair, with the prime as the way forward.
  Its Bound names that mechanism and what still passes: running from the prime's directory, or a direct board-file write, both of which land in `_board`'s history.
  The Goal, Scope, D5 (the refusal covers `gate`), the `PATTERN-refusal-way-forward` constraint and a new acceptance criterion follow.
  The command syntax was corrected to `lyx config <module> --set`, matching `configcli.Command`.
  Support log: Q7 interview and ledger lines, plus a rejected alternative (a standing deny in place of the refusal).
- **F2 (MEDIUM)** — D7 now decides hub membership by `cliwire`'s resolved target (the `-C` dir, else the cwd), never the bare cwd.
  Its Bound states that the unslotted path lets through only a target outside every hub.
  New acceptance criterion.
- **F3 (MEDIUM)** — D1 renders each gate step on its own line, and D8 has the fork run each step as its own Bash call, so one call holds at most one wait.
  D5 states the default's relation: half the Bash tool's 600-second maximum.
  The card-gate criterion now pins the unchained shape.
  Support log: Q4 interview and ledger lines, plus a rejected alternative (one invocation across modules).
- **F4 (LOW)** — the constraint now cites `verifyrun`'s package doc instead of `PATTERN-leaf-packages`.
  D4 now says `verifyrun`'s callers acquire the slot and compose the environment, and `verifyrun.Run` gains an environment parameter.
  The plan-writer note that restated this was removed.
- **F5 (LOW)** — D2 now states the slot-busy exit costs no self-fix attempt and does not make the report `FAILED`, for batch and card steps; D8 points at D2 instead of restating it.
- **F6 (LOW)** — D11's record now stores the failing verify as a kind, never a command string.
  The round gate builds its command from kind and quoted identities.
  D11's Bound states that deleting or editing the record skips or narrows only the round pass and D12's widening, and Publish's full rerun stays the guard.
  D12's wording follows.
- **F7 (LOW)** — new criteria cover D7's outside-a-hub path, D8's stencil statements and D11's no-identity fallback.
  The `CLAUDE.md` push rule moved into D8 beside the orch pre-push step, so its existing criterion now matches a Decision.

## Disputed

None.

## Deferred

None.
