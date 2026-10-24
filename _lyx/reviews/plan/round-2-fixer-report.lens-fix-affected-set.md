```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set lens: I reran each finding's search to check that the fix covers every member of the set.

Unchecked findings: none.

- **F1 (root-pair refusals with no way forward):** I searched the plan cards for root-pair and empty-`ParentBranch` text. Only cards 35 and 8 refuse a root pair, and the fix gave both refusals the way-forward clause and the test assertion. Cards 5 and 6 create or describe root pairs and refuse nothing. Card 15 reads an empty parent as the prime and card 18 ends a walk there, so neither refuses. The set is covered.
- **F2 (missing-records-worktree branch has no test):** I searched for refusals that name `lyx fabric reconcile` or a missing origin record. Only cards 2 and 8 word the two-cause refusal, and both now carry a removed-records-worktree row naming `lyx fabric reconcile`. Card 36's left-pair refusal names only the missing-worktree cause, and `TestBeamLandingReachesWorktreeCreateSkip` and `TestStartGuard_Refuse` already test that cause. It is outside F2's two-cause refusal and the fix did not touch it. Card 18 refuses nothing there. The set is covered.

No findings.

Tool calls:
1. Searched the cards other than 8 and 35 for root-pair or empty-parent text, to find F1's other members. My context showed only cards 2, 8 and 35, plus the parent's truncated search.
2. Searched every card for missing-origin-record and `fabric reconcile` refusals, to find F2's other members. My context held no search across all cards.
3. Searched card 36 for its origin-record refusal and the tests for it, to check whether it belongs to F2's set. My context lacked card 36.
