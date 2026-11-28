# Discussion-Bouncer reports stalemate when the fixer's decision-record edits are uncommitted

`bouncer-stalemate-uncommitted` · Task · bug, shed, burler · done

Imported from [issue #287](https://github.com/Knatte18/loomyard/issues/287).

## Seen

A motif run's Discussion-Bouncer escalated round 2 with cause `stalemate`, though the round's fixer had changed the decision record.
The fixer's round-2 edits to `_lym/discussion/decision-record.md` and `support-log.md`, which answered the one open finding as the review proposed, were in the records worktree but uncommitted.
The round's `round-2-changes.md` was empty, and the run agent reported that both round snapshots hash the review target to the SHA-256 of empty input (`e3b0c442…b855`).
The round-2 review itself saw the round-1 fixes and marked ten of eleven keys resolved, so reviewers read the working tree while the stalemate check did not.

## Expected

The stalemate check hashes the same review target the reviewers read, so a round whose fixer changed the decision record is never a stalemate.
An empty hash input should fail loudly, not count as an unchanged target.

## Workaround

The parent reads the decision record, confirms the finding is answered, and runs `lym loom circling accept <slug>`.
A `continue` would stall at stalemate again.
