```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes to cards 2 and 4 agree with the sibling cards, rows and cross-references around them.

no findings

- Card 2's new bound ("serves the machine name verb's own name set alone") matches card 3, which fills `hostName` with `machinename.FromThisHost` for the show and set halves of the same verb. It also matches the overview's card 2 line and its constraint note, so none of them is stale.
- In card 4, removing `internal/gitkit#Git` from Uses agrees with the cut subtest, its only user. Card 3's `gitkit.HermeticGitEnv` reason ("card 4's tests drive git") still holds, because the remaining tier 1 subtests use real hubs. `hubforge#CloneSecond` is still used by the seed.
- The input of the replacement row `SetNameOutOfOrderMovesToTheEnd` agrees with step 5's `nextFile` rule and with the shared successful-row assertion, whose `Replace` would be `["neve"]`. It does not duplicate `AlreadySetWritesNothing` or `OwnFormerNamePassesTheCollisionCheck`.
- The tier 1 lead-in ("Each subtest asserts only what the tier-0 rows cannot") and the closing "these subtests pin the verb's sweep end to end" still fit the two subtests that remain. The re-run criterion now sits on the tier 0 row `FailedHubIsReportedAndOthersSweep`, and no sibling card points the re-run at tier 1.

Tool calls:
- A grep of cards 0, 1 and 3 for re-run, tier 1 and move-to-end wording, because my context held only cards 2 and 4 whole and grep hits from the others.
