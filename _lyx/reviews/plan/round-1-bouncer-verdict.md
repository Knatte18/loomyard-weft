---
verdict: APPROVED
rationale: "All eight round-1 findings are fixed in the cards (card 1 attach seeding, Final bounded to PassOnCap, memo cleared after pending; card 2 refuses wait_min below 1; card 5 restart tests; card 7 overview edits and approve --review cwd; card 12 bound), every D1-D12 and Constraint maps to a card, and verify runs go test plus -tags integration over every targeted package"
---

The answer key is the decision record alone.
There is no `_lyx/loom/rework/` directory, so this is generation 0.

The round-1 report carries no `## Focus departures` section, so there is nothing to ratify or reject.
It settles all seven focus entries: five through findings F1, F2, F3, F5, F6 and F7, and the parentless-round and test-coverage questions through its own settlements, which I accept.
A parentless fresh start writing an empty `round-<N>/` is licensed by D3, since every fresh start opens the next round, and git commits no empty directory.

I checked each fix against the current cards:

- F1: card 1's Intent says an `AttachGated` run starts mid-turn and its replayed Done is the first gated arrival.
  It names the mid-turn-at-restart residual and bounds it by the closure's on-disk schedule, and `gate_pending_test.go` gains an `AttachGated` case.
- F2: a non-nil `Final` on a non-`PassOnCap` entry is a gate error, stated in the card 1 Bound, in the overview's Shared Decisions and in a test.
- F3: card 2 refuses `parent_review_wait_min` below 1 in a check of its own, and the refusal names `attempts: 0` as the off switch.
- F4: card 1 says the memo is cleared after a pending result and tests a verdict recorded between poll ticks.
- F5: card 5 tests the cap and throttle across a second `NewGate`, and a notify recorded between two closures.
- F6: card 7's `docs/overview.md` edit adds the tree line, the key and the status sentence.
- F7: card 12 has its own Bound for the gate entry and the widened pathspec.
- F8: card 7 resolves both review-file inputs against the invoking cwd.

My own pass found nothing blocking.
Every new verb, flag, gate name, hook and write carve-out states its bound (cards 1, 6, 7, 11 and 12).
No card uses `Custom`, and the split between the store (card 4) and the closure (card 5) gives each its own testable card.
Every `Edit` card's ImpactSummary states a conclusion rather than restating its Intent.
One observation does not rise to a finding.
Card 1 also edits two files outside its `Edit` list: the `wait.go` file comment, and conditionally `completionsignal_enforcement_test.go`'s pinned sites.
Both are in the package card 1 already targets through `Run.Wait`, and `go test ./...` covers them.
