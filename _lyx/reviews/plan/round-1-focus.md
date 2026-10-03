---
round: 1
focus:
  - "Card 9 quiet rule: it treats a pair whose task worktree is gone as quiet, so `fabric remove` with `RefuseWhenBusy` skips the wait and ends any live strand by name. Settle whether D2's bound (the wait never kills a non-quiet driver on the `fabric remove` path) licenses that, or whether the card must probe the driver strand through the session by name, or state this as a bounded exception in its Intent."
  - "Card 11 bound versus `Remote: true`: `internal/battencli/wire.go` already calls `Remove(location, slug, false, true)`, so after card 3 batten's teardown also deletes the landed task branch on origin. Settle whether card 11's 'gains no new deletion' bound is accurate, and whether its Intent states what the inherited D8 deletion can let through and what bounds it."
  - "Card 6 `EndSessionByName`: a new engine-less path that kills a session and possibly the hub's tmux server without the op lock. Settle whether its Intent states a bound (what it may skip, which sessions it can reach, who may call it) per the attack-surface rule, beyond a doc-comment warning."
  - "Acceptance-criteria fidelity across cards 8, 9 and 11: check that each criterion in `_lyx/discussion/decision-record.md` has a card whose tests or Intent carry it, especially 'the archive tag contains the driver's committed stop report' after a driver retires during the wait, 'the old strand's detached remover leaves the new strand alive', and batten ending a driver still live after its grace and recording the abandoned session through the composite."
  - "Card 12 against card 11 and card 13: settle whether every remaining reference to the deleted helpers is assigned to a card, including the comment at `internal/battencli/wire.go:103` that names `PairSiblingRemnant`, given that card 12 checks only build and vet and card 13's tripwire scans only for `RemovePairBranch`."
  - "Card 13 as a `Prosa` card that also creates a Go tripwire test: settle whether the enforcement test is an independently testable unit that belongs with the composite (card 9), or is rightly grouped with the invariant it enforces, per the granularity rule."
---

The plan is generation 0: there is no `_lyx/loom/rework/` directory, so the decision record alone is the answer key.

The quiet-rule entry is the one with the most at stake.
D4's rationale names a live strand writing into a removed pair as the bug being fixed, and that strand is reachable only through a gone task worktree.
Card 9 declares exactly that state quiet, which on the `fabric remove` path would end a working driver with no wait.

The `Remote: true` entry rests on a fact verified in the tree: batten passes `remote=true` to `Remove` today, so it inherits D8's origin deletion.
The open question is only whether card 11's stated bound accounts for it.

Already checked and not promoted:
- Verify coverage: every targeted package is run under `go test ./...` and under the integration line, and `smoke` is only vetted.
- Tmux isolation: `internal/fabriccli` already runs under `tmuxkit.Main`, and card 9 wires both `TestMain` files for the new package.
- No `Custom` cards.
- No missing `DependsOn` lists, and no `ImpactSummary` gaps on non-`Edit`/`Delete` cards.
