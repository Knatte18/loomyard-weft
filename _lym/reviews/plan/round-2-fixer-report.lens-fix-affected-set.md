```yaml
lens: fix-affected-set
outcome: reported
```

Lens task: check that the fix covered every member of each finding's set (F1 doc-comment wording, F2 row name, F3 subtest and its `gitkit.Git` use).

Unchecked list: none. I re-checked all of F1, F2 and F3.

no findings

- F1: no plan card still holds "unlike `Name`" or "ignores the machine file".
- F2: no plan card still names `NewNameMovesToTheEnd`.
- F3: no plan card still names `FailedHubIsSweptByTheReRun` or the unreachable-URL setup. The only remaining `gitkit` mentions are `HermeticGitEnv` in card 3 (lines 32 and 60). Card 3's reason for it, "card 4's tests drive git", still holds, because the remaining tier 1 subtests drive git through `hubforge` and the real Bolts.

Tool calls:
- One grep over the `_lym/plan/0*.md` cards for the three findings' names and wording. My context had only cards 02 and 04 and a partial grep, not a search of every card for all the terms.
