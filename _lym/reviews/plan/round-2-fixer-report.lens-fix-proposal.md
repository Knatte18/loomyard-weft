```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I checked each proposal the fixer carried out as written (F2, F3, F4, F5, F6, and F1's card-14 half) by trying to build a case where the proposal itself is wrong. Two findings, both LOW.

1. **LOW**, about the card 11 step 6 rewrite (F4) together with card 13's verdict `closed and not merged: Done when the record carries ClosedForContinue`.
   - **Summary:** F4's proposal makes this verdict reachable for the first time, and it ends Done without checking the tree, unlike its sibling verdicts.
   - **Case:** lym crashes after recording `ClosedForContinue`, but the interactive session stays alive and leaves uncommitted files. The head still equals `GatedHead`, so card 11 keeps the record and card 13's step 2 skips the gate. The poll then reads the pull request as closed, and the verdict ends Done on a dirty tree.
   - **Contrast:** the merged-at-`GatedHead` verdict halts on a dirty tree, names the files and gives a way forward. Here the dirty tree is caught only later, by Finalize's generic clean check (`internal/landingshed/finalize.go:839`, `fz.gate.clean(finalizeName, "before the merge-in")`). That check halts, so no wrong landing ships, but the halt does not use the row's PR-specific way forward.
   - **Possible fix:** require a clean tree on this verdict too, as the `Continue` sub-clause already does.

2. **LOW**, about card 13's new `checkHead` paragraph (F5): "on a failed result `the pr-review gate failed at tier <n>; …`, with `<n>` from `gateslot.FailedTier` over the log, else the highest tier `g.tiers(g.key)` lists".
   - **Summary:** F5's "else the highest tier" fallback also applies to a failed result whose shell never started, so the agent is told a tier failed when no tier ran.
   - **Evidence:** `verdict` handles `result.ExitCode < 0` as a separate case ("could not start", `internal/landingshed/verifygate.go`). `recordFailure`'s doc excludes the same case "since none of them names a failing test".
   - **Possible fix:** exclude `ExitCode < 0` from the tier fallback and send a could-not-start reason instead.

The other proposals held up:
- **F2:** no planned card other than landingshed, pairteardown, loomcli and envkit has non-test code import `internal/pullrequest`. I checked cards 9, 15 and 17 for this; card 17 touches only battencli and loomshed.
- **F3:** "its Stuck is always a Halt" holds through cards 11, 13 and 14; each of their Stucks is a Halt.
- **F6:** the exact-one assertion and the once-a-minute bound on later wakes are consistent with each other.
- **F1, card 14 half:** `ensureSession` brings in no other non-allowlisted import, since `runproc` and creel sit behind seams.

Tool calls:
- A grep over cards 9, 15 and 17 for packages that use `internal/pullrequest`. My context held only cards 8, 11, 13, 14, 16 and the overview, not these.
- A grep of `finalize.go` for its clean-tree checks, and of card 15 for its `ClosedForContinue` rule. My context did not show Finalize's code or card 15's text.
