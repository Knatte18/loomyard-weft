```yaml
lens: fix-proposal
outcome: reported
```

Findings: one MEDIUM, on F1. The F2 and F3 proposals hold.

**MEDIUM — F1's proposal contradicts card 3, the sibling card that uses `FromThisHost`.**
- Changed text: `_lym/plan/02-hostname-name.md`, the Intent's doc comment ("…it is for callers that need that name beside the file's names, and never as this machine's name") and its Bound ("`FromThisHost` serves the machine name verb's own name set alone").
- The problem: card 3's show uses `FromThisHost`'s result as this machine's name when there is no file or no directory.
- Evidence:
  - `_lym/plan/03-machine-name-show.md`: `productionDeps` fills `hostName` with `machinename.FromThisHost`.
  - In the same card, `current`'s "valid current name" is "the file's `name` when `machinename.CheckName` accepts it, else the hostname rule's name when no file or no directory". The show prints that value as `name` with source `hostname`, pinned by rows `NoFileReadsTheHostnameRule` and `NoDirectoryReadsTheHostnameRule`.
  - Card 4's `nextFile` likewise treats "the valid current name, which is the hostname rule's name with no file" as the old name.
  - This use is correct: `internal/machinename/machinename.go:126-142` (`resolveNames`) returns that same name from `Name` when no file exists.
- The case that breaks: there is no machine file, and `lym machine name` prints `FromThisHost()`'s result as this machine's name. That is exactly the use the new doc comment says never happens. The show also does not fit "the verb's own name set".
- The wording should not deny the use as this machine's name. It should limit callers to the machine name verb (show and set) and say that other code takes this machine's name from `Name`/`Names`.

**F2 (`SetNameOutOfOrderMovesToTheEnd`): no case breaks it.**
- Card 4's `nextFile` steps give this result:
  - The start is `["osl46","neve"]`.
  - The old name `osl46` and the hostname name `osl46` are both already there.
  - The move to the end gives `["neve","osl46"]`, which differs, so the file is written.
  - The replace set is `["neve"]`.
- This agrees with decision-record line 37.

**F3 (fold into `FailedHubIsReportedAndOthersSweep`): no case breaks it.**
- The second run finds the name already set and no change to `names`, so `nextFile` writes nothing and the replace set is the same.
- The fold carries decision-record line 147's criterion ("a re-run with the same name sweeps it").
- The removed real-origin case adds no unpushed-commit path. Per the review, `Refusal_UnreachableOriginAtPull` leaves the Bolt as it was, so a failed pull leaves no commit for the re-run to push.
- Card 3's `TestMain` line, "since card 4's tests drive git", stays true because the tier 1 subtests that remain use real hubs.

Tool calls:
- I grepped `00-overview.md` and card 3 for re-run and tier 1 references and listed `_lym`. The context did not show those texts or the layout of `_lym`.
- I grepped `_lym/discussion` for the decision record's re-run criteria. The context did not hold the decision record.
- I read lines 36-60 of `_lym/plan/03-machine-name-show.md`. The context held only the grep lines for card 3, not how the show uses `hostName`.
