```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenging each proposal the fixer carried out as proposed (F2–F8, plus F4 in card 15).

**MEDIUM: F4's fix rests on `BeamHasCommits`, and its handling of records-only commits is never defined.**
- Changed text: card 15, refusal 5 ("`BeamHasCommits(slug)` (card 13) gives `hasCommits` … a no passes, a beam with no commits beyond its parent included"), and the new passing `TestMarkReady` row. Card 14's `prClose` `hasCommits` is the same issue.
- Case: refusal 4 runs before refusal 5. It requires `DescribedAt`, and card 12's `describe` commits `summary.md` and the landing record onto the beam pair's branch ("commit the landing directory, the description and the record together", over `loomengine.LandingDirRel()`). Card 12 also counts the run's status commits on that first-parent line, its `RecordsOnly` commits.
- So by the time refusal 5 runs, a beam whose members were all cancelled still has commits beyond its parent, unless `BeamHasCommits` skips records-only commits.
- Card 13 defines `BeamHasCommits` only as `func(slug string) (bool, error)`, filled "over the beam's pair", and never says whether records-only commits count.
  - If they count, the new passing row cannot happen on a real hub. It passes only over `newLandWorld`'s fakes, and the premise F4 fixes, a `required` parent with an empty beam, never reaches the `hasCommits == false` branch.
  - If they don't count, refusal 5 and `prClose` judge "empty" differently from Finalize, whose resolver gets its own `taskHasCommits`.
- Evidence: card 12, lines 42–43 and 45 (`RecordsOnly`, the describe commit); card 13, line 42 (the `BeamHasCommits` declaration, with no definition).
- The fix should state that `BeamHasCommits` counts only commits that are not `RecordsOnly`, through `fabricengine.BeamLineCommits`. It should also state how Finalize's `taskHasCommits` lines up with that.

**LOW: F8's fix does not say to read every page of `GetCombinedStatus`.**
- Changed text: card 3's `RequiredChecks` bullet ("It also reads `Repositories.GetCombinedStatus` at head …").
- The same bullet says "every page" for `GetRulesForBranch` and `ListCheckRunsForRef`.
- The combined status is paginated too, with up to 100 statuses per page. If the read stops at page one, a required context reported past the first page reads `missing`, the same defect F8 fixes.
- Evidence: the bullet's own wording, "reads every page of …" for the other two reads and none for this one.

**No findings for the other proposals:**
- **F2:** the split holds. `retiredRows` (`internal/loomrecipe/recipekeys.go:22-31`) has entries only for tapestry and darn, and none for pawl. So no arm-time rename lands a pawl run on Parent-Align, which card 5 drops.
- **F3:** each doc rewrite lands in a card that already edits that file or now lists the member.
- **F5:** passing `failed` matches card 15's refusal 1 ("running or halted").
- **F6:**
  - A crash after the await-marker removal, or after the dobby-stop removal, leaves the land request standing. `interruptedLanding` then reads false, `enter` writes the marker again, and the request is accepted again, with `landChecks` writing the dobby-stop marker again.
  - A crash after all three removals leaves the second detection to settle it.
  - No ordering of the removals leaves the marker standing without the request.
- **F7:** the old accept wrote only Parent-Align to the board (`beamawait.go:295`). No path writes PR-Review as the hold, so Parent-Align is at least as reachable as the PR-Review row it replaces.

Tool calls:
- `sed` of `internal/loomrecipe/recipekeys.go:20-32` and a grep of `pawlwiring.go`: my context had not shown the retired-rows table's entries, and I needed them to test whether F2's split holds for pawl.
