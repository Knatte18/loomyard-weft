```yaml
lens: coverage
outcome: reported
```

Coverage lens: does every decision, acceptance criterion and new branch in the plan have a planned row that fails when it breaks?

1. **MEDIUM — D1 and criterion 6 (empty title from a missing or unreadable seed): `lym loom start`'s use of the seed has no row.**
   - Card 2 makes `internal/loomcli#loomCLI.ensureStatusStrand` pass `loomengine.ChecklistRecipe(c.location)` "never `c.recipe`". No planned row reaches `ensureStatusStrand`.
   - `internal/loomcli#TestStatusStrandAddSpec` passes the literal `pawl` on every row, and `TestSmokeStatusStrandAcrossDriverSeeds` passes `""` with nil strands. Both bypass the seed read.
   - So an implementation that passes `c.recipe` (tapestry by default), or a hard-coded recipe, passes every planned test. The criterion's loomcli half ("both loomcli's and orchcli's beside spec have an empty `Title`") is argued only through `TestChecklistRecipe`.

2. **MEDIUM — criterion 6, orchcli half: no row covers an unreadable seed.**
   - Card 3's row "with no seed the moved checklist has an empty title" covers only a missing seed.
   - The unreadable-seed case (`{not json`) is pinned only in card 1's `TestChecklistRecipe`, not on the orchcli path the criterion names.

3. **MEDIUM — card 3's new fall-back for a failed re-read has no row.**
   - Card 3 adds "A failed read there falls back to the add" after a failed move.
   - `internal/orchcli#fakeStrands.Strands` (cli_test.go:62-68) cannot return an error, and card 3 adds no such field. No row reaches the branch.

4. **MEDIUM — D2 case 2: the checklist sidecar written after the relaunch is not asserted.**
   - D2 says "After each replace of `loom-status` it writes the loom status sidecar".
   - Card 3's row "a dead loom status strand beside a current runs strand is relaunched beside it" names no wanted checklist sidecar. Only the case-1 rows name `replaced-loom-status`.
   - A `moveChecklist` that skipped the sidecar write in case 2 would pass.

5. **MEDIUM — D2 case 2 with a failed move has no row.**
   - Card 3 does not say what happens when the move fails while a runs strand is present (warn, then keep or replace the runs strand).
   - The record's failure bullet ("a failed replace of `loom-status` warns, naming … `lym loom start`") applies here too.
   - Both failed-move rows have no runs strand, so the case-2 failure branch goes unreached.

6. **LOW — D2 with a stale runs strand and a dead `loom-status` strand has no row.**
   - In this combination card 3 moves the checklist and then replaces the runs strand.
   - The "precedes/follows" rows use a live `s1`, so only the replace of the runs strand is reached, never the move followed by that replace.

7. **MEDIUM — D4: the blank line after the own-run line is never reached.**
   - Card 5's `frame` adds "a blank line when the dobby or orch section follows".
   - `TestOverviewSource_DrawsTheOwnRunLine` builds a beam "with no orch state" and no dobby, so the frame never has a section after the line.
   - The existing frame tests hold no seed, so neither the blank line nor the line-above-the-dobby order is pinned.

8. **LOW — D4: a seed that fails to read has no row, and card 5 misdescribes an unreadable status.**
   - Card 5's seed-read-error branch (logged at Debug) is not reached: the rows cover a missing seed only, never an unreadable one.
   - Card 5 says an unreadable status "answers `Found` false". `internal/statuspane/cache.go:84-89` (`StatusCache.fail`) returns `Found: c.haveStatus` on the first failure after a good read. So the line stays drawn for one poll, then disappears once `Unreadable` is set.
   - No row pins either behaviour.

9. **LOW — D3: a break-out of a strand with a reserved title has no row.**
   - Card 4 makes `moveShellWindowStrandsLocked` use `ownWindowName`.
   - The `TestListReconcilePanesLocked_MovesShellWindowStrands` rows cover a `pawl` title and an empty title (role) only. The `shell` and `overview` fall-back is pinned only on the `AddStrand` path in the tier-2 subtest.

10. **LOW — card 3: the early returns of the re-signed `ensureRunsStrand` have no row.**
    - These are the build identity, executable and strand-read failures; card 3 changes their return to a one-element `[]string`.
    - `beamUp` now appends all of them, and no row covers any of these early returns.

Extra tool calls:
- Read `internal/statuspane/cache.go:10-90` to confirm what `StatusCache.Poll` answers for an unreadable file; my context only had the `StatusPoll` field summary.
