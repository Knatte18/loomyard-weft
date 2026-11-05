```yaml
lens: coverage
outcome: reported
```

Coverage lens: checking each decision, acceptance criterion and new branch in the plan for a card that carries it and a test row that would fail if it broke.

**BLOCKING — D10/D11, card 10: a goto back to Beam-Await while the ready record stands probably never reaches `recheckStanding`.**
- The rule fires only when the board's run status still names Finalize or a retired row.
- Loom mirrors every shed transition onto the board: `SetBoardStatus` writes `boardStatus(producer, state, …)` (`internal/loomcli/wiring.go:197-203`, `boardStatus` just below). If the goto's transition also passes through that mirror, which I did not confirm, the board names Beam-Await before Beam-Await's `Call` runs.
- Beam-Await would then wait for a land request that never comes, because the dobby's land episode stays recorded (Constraints). The run is stuck.
- No planned row reaches the goto path. Card 10's rows only fake a board status that names Finalize or PR-Review.
- The D10 criterion ("a goto back … re-checks and is Done") has no test that would fail. Fix: key the entry on the ready record standing with no land request, whatever the board says, and add a goto row.

**BLOCKING — D7, card 16: the dobby's notices can loop.** `announceLandings` counts every first-parent commit past the announce head as a member landing. The beam verbs commit straight onto the beam branch:
- `lym beam describe` commits "beam: change description for <slug>" (card 12).
- `lym beam pr open` and `update` commit the landing record (cards 13 and 14).
- The run's own status commits land there too.

Each describe or update the notice asks for makes a new commit, and the next tick announces it, so the notice repeats forever. D7 says "each landing into the beam branch". No row covers a describe or record commit past the head.

**MEDIUM — D8, card 11: a pull request merged at another head is landed by a local squash merge, whatever the parent requires.** D8 says to land it "as with none recorded". Under a `required` parent, `merge` parent or rules-required parent that means finding or opening a pull request. A local push there is refused as protected. No row covers a merged-at-another-head pull request under a required parent.

**MEDIUM — D3, card 15: three branches have no row or no answer.**
- Refusal 5's resolver can return a `Halt`, for example a required parent with no GitHub origin. The card names no answer for it and no row.
- No row covers the rules-required parent with none recorded.
- Card 11 marks every beam yes as `MergeAtOnce`, so the decision cannot tell `merge` from `required`. The card has to read the mode itself, and no row pins that.

**MEDIUM — D3 refusals 3 and 6, card 15: the real logic lives in `hubLandDeps` closures that no row runs.**
- `ParentUnmerged` and `MergedLacks` carry the behavior: the upstream fetch only when an upstream exists and push skip is unset, local tip only otherwise, and a parent merge counted by its second parent being reachable from the parent branch.
- `TestMarkReady` drives fakes only, so the rows for a parent with no upstream judged by its local tip, and for a merge whose only extra commit merges the parent in, would pass with a wrong closure.

**MEDIUM — D5/D4, card 12: the description stamp and the joint commit are untested.** The stamp of `DescribedAt` and the commit of description plus record together sit inside the `Describe` closure. `TestDescribe` fakes that closure. The criterion "stamps the record … commits both" has no failing row, and card 13's no-description refusal depends on that stamp.

**MEDIUM — D6, cards 13 and 14: four branches have no row.**
- `pr open` on a recorded pull request GitHub reports merged.
- `pr open` when the record write fails after the pull request was created.
- `pr close` when the mode read fails (it still closes and drops).
- `pr close` reporting on a rules-required parent; only a `required` beam.yaml parent has a row.

**MEDIUM — D13, card 8: two parts of the move have no row.**
- `TestMoveOffRetired` asserts nothing about `EndSession`, so "ends a live PR-Review session" has no row.
- The hub closure that reads GitHub before carrying the scratch pull request has no row, including the "read failure counts as found" branch.
- The criterion "a run at Finalize is untouched" has no row.

**MEDIUM — D2, card 7: the hand-run path is tested only through the pure precondition.** The `beamForkParent` closure that resolves the beam's pair and calls `ReadyFor` is not driven by any row. The criterion "a hand `lym batten run` of a member under a ready beam is refused" rests on that untested wiring.

**LOW — D8, card 11: two branches are unspecified or unreached.**
- A failed `BeamRecord` read leads to an error stop, with no row.
- For a beam, `EntryField` must not be read, but the card never says `Finalize.pullRequestLanding` skips the `EntryField` call. The resolver row covers only the argument.

**LOW — D7, card 16: failure branches without rows.**
- A failed `BeamLandingsSince` or `LandingPullRequest` read has no row.
- The `hubDobbyDeps` first-parent log closure has no row.

**LOW — D5/D7: the description will grow its own commits.** `beamCommits` lists every first-parent commit, so each re-describe adds its own describe and record commits to "## Changes". This is the same root cause as the notice-loop finding.

Tool calls:
1. Grepped for the GitHub-auth guard test and the `SetRunStatus` writers: the context held no record of who writes a pawl run's board status at Finalize or after a goto.
2. Read `internal/loomcli/wiring.go:185-215` and grepped landingshed's seam guard: needed to confirm the board status mirrors each shed transition.
