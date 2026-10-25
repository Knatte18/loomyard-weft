```yaml
lens: fix-claims
outcome: reported
```

Checked the fix's claims in cards 4, 6, 7, 12 and 16 against the code at HEAD. I found one issue.

**MEDIUM: card 12's "What bounds it" paragraph leaves out one re-escalation path**
- **Changed text:** `_lyx/plan/12-bouncer-decision-escalation.md`, the new paragraph that opens "What bounds it:". The fix copied this clause from the reviewer's proposed fix: "one escalation per judged round, plus one per decision retired at the budget below the hard cap".
- **Problem:** the bound misses a case the same card describes. A pending `continue` at or past the hard cap is also retired and escalated again.
- **Evidence:** the card's own "A pending `continue`" bullet says: "at or past the hard cap, `retireContinue` then `escalateWith(…, EscalationDecision, …)`". Test row 12 covers it: "a `continue` recorded below the cap, settled after `HardCap` is lowered to the round, re-escalates keeping cause `decision`". So a retired decision can re-escalate at or past the hard cap, not only below it.
- **Suggested fix:** say "plus one per decision retired: an accept held at the budget below the hard cap, or a continue that meets the hard cap".

**Claims that hold:**
- **Card 16, `TestLoadConfig`:** the two scope-refusal rows key `merge-in`, and their want text holds `gates.merge-in` (`config_test.go:125-126`).
- **Card 16, `roundGateConfig`:** its `gates` block keys `merge-in` (`loomshed/verifygate_integration_test.go:146`).
- **Card 16, websterengine's `verifyGateConfig`:** its `gates` block keys `merge-in` (`websterengine/verifygate_integration_test.go:47`), and it is a different function from webstercli's.
- **Card 16, `newMergeInFixture`:** its gate key is `merge-in` (`mergein_test.go:105`, `:131`).
- **Card 16, `TestMergeIn_PlanVerifyFailureRecord`:**
  - It looks up `fixedGateTierLists["merge-in"]` (`mergein_test.go:543`).
  - It wants `Kind` to be `merge-in` (`mergein_test.go:544`).
  - The kind is `verifytree.FailureKind(g.key)` (`landingshed/verifygate.go:62`), so it follows the fixture's gate key.
- **Card 16, `TestMergeIn_VerifyGate`:** it wants the site `Label` `Merge-In` (`mergein_test.go:664`). The label is the producer name (`landingshed/verifygate.go:233`), and the producer name is `mergeInName`, which the card renames.
- **Card 16, the claim that the list comes from a literal search:** this matches the fixer's grep. The hits left out (fabric verb, git-merge order labels, `FixerAttemptsPath` argument, `markMergeRecord`'s verb) are fabric or free-form uses.
- **Card 6, `docs/overview.md`:** the config paragraph lists `parent_review_wait_min` and none of the alignment keys. The reconcile paragraph is where the dropped keys are named, and card 6 has `MigrateConfig` drop all four keys.
- **Card 4:** with the parent-review gate removed, the sentence names a verify wait and a shuttle wait, as before.
- **Card 12, "What it can skip or let through":**
  - An accept is refused only by an open BLOCKING entry that has no class, so other BLOCKING findings pass once a judgment is admitted.
  - The carry-over adds the `decision` findings and the BLOCKING findings of the review.
  - A continue applies only the dropped-key check.
- **Card 12, "the hard cap, at which a held refusal halts the run":** matches the accept bullet.
- **Card 7:** removing the bullet leaves no dangling reference. No current test checks a sentence of the `goal-scope` or `board-fit` lens text.

Tool calls:
1. Read `websterengine/verifygate_integration_test.go` lines 34-53 and 196-202, and the `Label` line in `landingshed/verifygate.go`. My context did not show the body of that `gates` block, the `strings.Replace` caller, or where the site label comes from.
