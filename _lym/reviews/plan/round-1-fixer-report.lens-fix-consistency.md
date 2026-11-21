```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the change set agrees with its sibling cards, decisions and cross-references.

**Findings**

1. **LOW**: Card 7 (`07-pair-lifecycle-namespace.md:31`) still overclaims after the overview was narrowed.
   - Changed text: the overview's `coverage-of-namespace-sites` decision, which now says that `probeLiveLocalWeft` and `readSideOwnerArchives` keep only the two-repository suites' coverage.
   - Problem: card 7's Intent was left as it was and still says "These tests reach the namespace paths cards 2 and 3 changed." Those two sites are among the paths card 3 changed.
   - Evidence: the overview decision text, against `07-pair-lifecycle-namespace.md:31`.

2. **LOW**: The Card Index line for card 8 does not mention its new `PATTERN-code-conventions` work.
   - Changed text: card 8's Intent, Edit list (`pattern/PATTERN-code-conventions.md`, `PATTERN.md`) and ImpactSummary, which now declare that change.
   - Problem: the index line still reads "8 — docs-and-pattern — describe the layout and its hazards, and restore PATTERN-single-repo-hub". Every other changed card's index line was updated to match its new scope, for example cards 2 and 4.
   - Evidence: `00-overview.md`, Card Index.

**Outside this lens:** card 2 now says a joined board passes the new unseeded-board refusal because it "adopts `origin/main`". That only holds if `ensureBoardWorktree` sets `branch.main.merge` when it adopts, and the plan does not say so. If it doesn't, a join's `PushCloneBoard` and the `CloneSecond` rows in card 5 would hit the refusal. This is for the fix-claims lens to settle.

Tool calls:
- One read of the renumbered `00-overview.md` and a grep of card cross-references across `01`–`08`. My context held these files only as they were before `lym loom remove-card`, which rewrote the numbers.
