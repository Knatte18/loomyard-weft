```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote about bounds, behaviour or code holds against the code at this worktree's HEAD.

No findings.

What I checked against HEAD:
- **D7, cancel:** "the operator's `lym batten cancel`, which today only a hub entry refuses" holds. Its only pause refusal is `hubPauseLive`, which reads `ScopeHub` (`internal/battencli/cancelcmd.go:469-472`). `OwnerHeld` only words a follow-up on another refusal (`internal/battencli/cancel.go:609-617`).
- **D7, hibernate wake:** "skips its reconcile under a hub entry, never reads a `hold` slug entry as one" holds. The loop skips every entry that is not `ScopeHub` (`internal/hibernateengine/wake.go:58-60`).
- **D7, one entry per scope:** holds (`internal/pauserecord/doc.go:4`).
- **D7 Bound, parent lock:** "a run that holds its parent lock is exempt from slug entries" holds (`internal/pauserecord/doc.go:52`).
- **D7 and D8, the orch:** the claim that slug entries leave the orch uninterrupted holds; "a slug entry never covers the orch" (`internal/orchengine/pause.go:4`).
- **D2, teardown lock:** the respawn of a pair's push that exited on a held teardown lock matches `respawnPairPush` (`internal/fabricengine/teardownlock.go:155-169`).
- **D2, anchor change:** the anchor change's push carries the pair's anchor move commit (`internal/fabricengine/anchorchange.go:543-557`).
- **D10 Facts:** "`Knatte18/loomyard-weft` is public" holds; `gh repo view` answered `PUBLIC` in this context.
- **D11, kept pairs:** "a pair kept by a cancel before the upgrade has no batten record left" holds. Cancel "removes the run's records" (`internal/battencli/cancelcmd.go:56-57`).

Outside my lens: D8 step 1's new way forward, waiting for the other machine's adopt to clear its hold, never makes progress when that adopter has stopped. That case belongs to the fix-way-forward lens.

Tool calls:
- Read `internal/battencli/cancelcmd.go:55-72` and searched for `OwnerPauseHeld`. My context lacked cancel's full refusal list and what an owner pause does to a cancel.
- Read `internal/battencli/cancel.go:600-625`. My context lacked whether `OwnerHeld` refuses a cancel or only words its follow-up.
